# Day 3: Building a Real Agent — Web Search and Summarization

Welcome to Day 3. Yesterday's agent called mock functions. Today we wire up real APIs and build a genuinely useful tool: a web research agent that searches the internet, reads pages, and synthesizes information to answer complex questions.

## The Tools We Are Building

1. **`web_search(query)`** — Uses DuckDuckGo's free search API to find relevant URLs
2. **`fetch_page(url)`** — Downloads and extracts readable text from a URL
3. **`summarize_text(text, focus)`** — Summarizes a long text focused on a specific question

```bash
pip install duckduckgo-search httpx beautifulsoup4 anthropic
```

## Tool Implementations

```python
# research_tools.py
import httpx
from bs4 import BeautifulSoup
from duckduckgo_search import DDGS
import anthropic
import os

client = anthropic.Anthropic()


def web_search(query: str, max_results: int = 5) -> list[dict]:
    """
    Search the web using DuckDuckGo (no API key required).
    Returns a list of {title, url, snippet} dicts.
    """
    with DDGS() as ddgs:
        results = list(ddgs.text(query, max_results=max_results))
    return [
        {"title": r.get("title", ""), "url": r.get("href", ""), "snippet": r.get("body", "")}
        for r in results
    ]


def fetch_page(url: str, max_chars: int = 4000) -> str:
    """
    Fetch a web page and extract readable text (strips HTML, scripts, styles).
    Returns first max_chars characters to stay within token limits.
    """
    try:
        headers = {"User-Agent": "Mozilla/5.0 (compatible; ResearchBot/1.0)"}
        response = httpx.get(url, headers=headers, timeout=10, follow_redirects=True)
        response.raise_for_status()

        soup = BeautifulSoup(response.text, "html.parser")

        # Remove non-content elements
        for tag in soup(["script", "style", "nav", "footer", "header", "aside"]):
            tag.decompose()

        text = soup.get_text(separator="\n", strip=True)
        lines = [line for line in text.splitlines() if len(line.strip()) > 30]
        return "\n".join(lines)[:max_chars]

    except Exception as e:
        return f"Error fetching page: {str(e)}"


def summarize_text(text: str, focus: str) -> str:
    """
    Use Claude to summarize text with a specific focus question.
    """
    r = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=300,
        messages=[{
            "role": "user",
            "content": f"""Summarize the following text, focusing on: {focus}

Text:
{text[:3000]}

Provide a concise summary (3-5 sentences) focused on the question above."""
        }]
    )
    return r.content[0].text
```

## Tool Schemas for the Agent

```python
# research_agent_tools.py
RESEARCH_TOOLS = [
    {
        "name": "web_search",
        "description": "Search the web for information. Returns titles, URLs, and snippets. Use this first to find relevant sources.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "The search query"},
                "max_results": {"type": "integer", "description": "Number of results (default 5)", "default": 5}
            },
            "required": ["query"]
        }
    },
    {
        "name": "fetch_page",
        "description": "Fetch and read the full content of a web page by URL. Use after web_search to read the most relevant pages.",
        "input_schema": {
            "type": "object",
            "properties": {
                "url": {"type": "string", "description": "The full URL of the page to fetch"}
            },
            "required": ["url"]
        }
    },
    {
        "name": "summarize_text",
        "description": "Summarize a long piece of text with a specific focus. Use to condense page content before including it in your response.",
        "input_schema": {
            "type": "object",
            "properties": {
                "text":  {"type": "string", "description": "The text to summarize"},
                "focus": {"type": "string", "description": "The specific question or aspect to focus on"}
            },
            "required": ["text", "focus"]
        }
    }
]
```

## The Research Agent

```python
# research_agent.py
import json
import anthropic
from dotenv import load_dotenv
from research_tools import web_search, fetch_page, summarize_text
from research_agent_tools import RESEARCH_TOOLS

load_dotenv()
client = anthropic.Anthropic()

TOOL_REGISTRY = {
    "web_search":    lambda **kw: web_search(**kw),
    "fetch_page":    lambda **kw: fetch_page(**kw),
    "summarize_text": lambda **kw: summarize_text(**kw),
}

SYSTEM = """You are a research assistant with web search capabilities.

When given a research question:
1. Search for relevant sources using web_search
2. Read the most promising pages using fetch_page
3. Summarize key information using summarize_text if pages are long
4. Synthesize findings into a comprehensive, accurate answer

Always cite your sources by mentioning the URLs you read.
Be honest about uncertainty — if sources conflict, say so."""

def research(question: str, verbose: bool = True) -> str:
    messages = [{"role": "user", "content": question}]

    for turn in range(15):
        response = client.messages.create(
            model="claude-opus-4-7",
            max_tokens=1500,
            tools=RESEARCH_TOOLS,
            system=SYSTEM,
            messages=messages
        )

        if verbose:
            print(f"  [Turn {turn+1}] {response.stop_reason}", end="")

        if response.stop_reason == "end_turn":
            if verbose:
                print()
            return next(
                (b.text for b in response.content if b.type == "text"),
                "No answer generated."
            )

        if response.stop_reason == "tool_use":
            messages.append({"role": "assistant", "content": response.content})
            tool_results = []

            for block in response.content:
                if block.type == "tool_use":
                    if verbose:
                        print(f" → {block.name}({list(block.input.keys())})")
                    result = TOOL_REGISTRY[block.name](**block.input)
                    tool_results.append({
                        "type":        "tool_result",
                        "tool_use_id": block.id,
                        "content":     json.dumps(result) if isinstance(result, (list, dict)) else str(result)
                    })

            messages.append({"role": "user", "content": tool_results})

    return "Research did not complete within the turn limit."


if __name__ == "__main__":
    question = "What are the main differences between LangChain and LlamaIndex for building RAG applications?"
    print(f"Research Question: {question}\n{'─'*60}\n")
    answer = research(question, verbose=True)
    print(f"\nAnswer:\n{answer}")
```

```result
Research Question: What are the main differences between LangChain and LlamaIndex for building RAG applications?
────────────────────────────────────────────────────────────

  [Turn 1] tool_use → web_search(['query'])
  [Turn 2] tool_use → fetch_page(['url'])
  [Turn 3] tool_use → fetch_page(['url'])
  [Turn 4] tool_use → summarize_text(['text', 'focus'])
  [Turn 5] end_turn

Answer:
Based on my research, here are the key differences:

**LangChain** is a general-purpose LLM application framework covering chains, agents, memory, and integrations with 100+ tools and data sources. It is highly flexible but has a steeper learning curve due to its breadth.

**LlamaIndex** (formerly GPT Index) is specifically designed for RAG and data indexing. It provides superior built-in support for document loading, chunking strategies, and retrieval evaluation. Teams building document Q&A systems often find it more productive out of the box.

**When to use each:**
- LangChain: Multi-step agents, complex workflows, broad integrations
- LlamaIndex: Document ingestion, RAG pipelines, knowledge base Q&A

Sources: [docs.llamaindex.ai], [python.langchain.com/docs]
```

## Wrapping Up Day 3

Your research agent searches the web, reads real pages, and synthesizes findings — all autonomously. The LLM decides which pages to read, in what order, and when it has enough information to answer. Tomorrow on **Day 4: Multi-Agent Patterns**, we explore how to coordinate multiple specialized agents to handle tasks that are too complex for a single agent.
