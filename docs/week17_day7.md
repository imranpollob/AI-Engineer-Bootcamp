# Day 7: Final Capstone — The AI Research Assistant

Welcome to Day 7 — and the final day of the AI Engineer Bootcamp. Today is the culmination of 17 weeks of work. You will build a complete, production-quality **AI Research Assistant**: a CLI agent that accepts research questions, searches the web autonomously, synthesizes findings, and delivers cited answers — with proper cost controls, logging, and safety guardrails.

This is everything you have learned, assembled into one real-world application.

## What You Are Building

```
$ python research_assistant.py

╔══════════════════════════════════════════════════════╗
║         AI Research Assistant v1.0                    ║
║   Type your question, /help for commands, /quit      ║
╚══════════════════════════════════════════════════════╝

You: What are the latest approaches to reducing LLM hallucination?

[Researching... turn 1/10]
  → web_search(["query"])
[Researching... turn 2/10]
  → fetch_page(["url"])
  → fetch_page(["url"])
[Researching... turn 3/10]
  → summarize_text(["text", "focus"])
[Done in 3 turns | 2,847 tokens | $0.0284]

Answer:
Recent approaches include: (1) RAG — grounding answers in retrieved documents...
Sources: [arxiv.org/...], [huggingface.co/...]
```

## The Complete Application

```python
# research_assistant.py
"""
AI Research Assistant — Final Capstone
Combines: Tool use, web search, cost control, observability, safety guardrails.
"""
import json
import time
import uuid
import logging
from datetime import datetime, timezone
from dotenv import load_dotenv
import anthropic

load_dotenv()

# ─── Logging ─────────────────────────────────────────────────────
logging.basicConfig(
    filename="research_assistant.log",
    level=logging.INFO,
    format="%(message)s"
)
logger = logging.getLogger("assistant")


# ─── Tool Implementations ────────────────────────────────────────
def web_search(query: str, max_results: int = 5) -> list[dict]:
    try:
        from duckduckgo_search import DDGS
        with DDGS() as ddgs:
            results = list(ddgs.text(query, max_results=max_results))
        return [{"title": r.get("title",""), "url": r.get("href",""), "snippet": r.get("body","")} for r in results]
    except Exception as e:
        return [{"error": str(e)}]

def fetch_page(url: str, max_chars: int = 3000) -> str:
    try:
        import httpx
        from bs4 import BeautifulSoup
        resp = httpx.get(url, headers={"User-Agent": "ResearchBot/1.0"}, timeout=10, follow_redirects=True)
        soup = BeautifulSoup(resp.text, "html.parser")
        for tag in soup(["script","style","nav","footer","header"]):
            tag.decompose()
        lines = [l for l in soup.get_text(separator="\n", strip=True).splitlines() if len(l.strip()) > 30]
        return "\n".join(lines)[:max_chars]
    except Exception as e:
        return f"Error: {e}"


TOOLS = [
    {
        "name": "web_search",
        "description": "Search the web for current information. Use this first to find relevant sources.",
        "input_schema": {
            "type": "object",
            "properties": {
                "query": {"type": "string", "description": "Search query"},
                "max_results": {"type": "integer", "default": 5}
            },
            "required": ["query"]
        }
    },
    {
        "name": "fetch_page",
        "description": "Read the full content of a web page. Use after web_search to get detailed information.",
        "input_schema": {
            "type": "object",
            "properties": {
                "url": {"type": "string", "description": "The URL to fetch"}
            },
            "required": ["url"]
        }
    }
]

TOOL_REGISTRY = {"web_search": web_search, "fetch_page": fetch_page}

SYSTEM = """You are a rigorous research assistant with web search capabilities.

Process:
1. Search for 3-5 relevant sources on the topic
2. Read the most promising 2-3 pages in full
3. Synthesize findings into a comprehensive, well-structured answer
4. Always cite your sources with URLs

Standards:
- Distinguish between established facts and emerging research
- Note when sources disagree
- Acknowledge uncertainty rather than speculating
- Be concise but complete"""

INJECTION_PATTERNS = ["ignore previous instructions", "ignore all instructions", "disregard", "you are now", "forget your"]


# ─── Safety ──────────────────────────────────────────────────────
def validate_input(text: str) -> None:
    lower = text.lower()
    for pattern in INJECTION_PATTERNS:
        if pattern in lower:
            raise ValueError(f"Unsafe input detected: '{pattern}'")
    if len(text) > 2000:
        raise ValueError("Question too long (max 2000 characters).")


# ─── Core Agent ──────────────────────────────────────────────────
class ResearchAssistant:
    MAX_TOKENS_PER_SESSION = 80_000
    MAX_TURNS = 10
    PRICE_IN  = 15.00   # per million tokens
    PRICE_OUT = 75.00

    def __init__(self):
        self.client          = anthropic.Anthropic()
        self.session_id      = str(uuid.uuid4())[:8]
        self.session_tokens  = {"input": 0, "output": 0}
        self.session_cost    = 0.0
        self.conversation    = []

    def _cost(self, inp: int, out: int) -> float:
        return (inp * self.PRICE_IN + out * self.PRICE_OUT) / 1_000_000

    def _log(self, event: str, **data):
        logger.info(json.dumps({
            "session": self.session_id,
            "ts": datetime.now(timezone.utc).isoformat(),
            "event": event,
            **data
        }))

    def research(self, question: str) -> str:
        validate_input(question)
        messages = [{"role": "user", "content": question}]
        turn_count = 0

        while turn_count < self.MAX_TURNS:
            turn_count += 1
            print(f"  [Researching... turn {turn_count}/{self.MAX_TURNS}]", end="", flush=True)

            response = self.client.messages.create(
                model="claude-opus-4-7",
                max_tokens=1500,
                tools=TOOLS,
                system=SYSTEM,
                messages=messages
            )

            in_tok  = response.usage.input_tokens
            out_tok = response.usage.output_tokens

            # Budget check
            total_now = self.session_tokens["input"] + self.session_tokens["output"] + in_tok + out_tok
            if total_now > self.MAX_TOKENS_PER_SESSION:
                print()
                self._log("budget_exceeded", tokens=total_now)
                return "Session token budget exceeded. Please start a new session."

            self.session_tokens["input"]  += in_tok
            self.session_tokens["output"] += out_tok
            cost = self._cost(in_tok, out_tok)
            self.session_cost += cost

            self._log("llm_call", turn=turn_count, stop_reason=response.stop_reason,
                      input_tokens=in_tok, output_tokens=out_tok)

            if response.stop_reason == "end_turn":
                print()
                return next((b.text for b in response.content if b.type == "text"), "No answer.")

            if response.stop_reason == "tool_use":
                messages.append({"role": "assistant", "content": response.content})
                tool_results = []

                for block in response.content:
                    if block.type == "tool_use":
                        print(f"\n    → {block.name}({list(block.input.keys())})", end="", flush=True)
                        t0 = time.time()
                        result = TOOL_REGISTRY[block.name](**block.input)
                        elapsed = round((time.time() - t0) * 1000, 1)
                        result_str = json.dumps(result) if isinstance(result, (list,dict)) else str(result)
                        self._log("tool_call", tool=block.name, duration_ms=elapsed)
                        tool_results.append({
                            "type": "tool_result",
                            "tool_use_id": block.id,
                            "content": result_str
                        })

                print()
                messages.append({"role": "user", "content": tool_results})

        return "Research reached maximum turns. Please try a more specific question."


# ─── CLI ─────────────────────────────────────────────────────────
def main():
    assistant = ResearchAssistant()
    print("╔══════════════════════════════════════════════════════╗")
    print("║         AI Research Assistant v1.0                    ║")
    print("║   /cost  /clear  /help  /quit                         ║")
    print("╚══════════════════════════════════════════════════════╝\n")

    while True:
        try:
            question = input("You: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\nGoodbye!")
            break

        if not question:
            continue

        if question == "/quit":
            print(f"Session cost: ${assistant.session_cost:.4f}")
            break
        elif question == "/cost":
            t = assistant.session_tokens
            print(f"  Tokens: {t['input']}↑ {t['output']}↓ | Cost: ${assistant.session_cost:.4f}")
            continue
        elif question == "/help":
            print("  Ask any research question. Commands: /cost  /quit")
            continue

        try:
            answer = assistant.research(question)
            in_t = assistant.session_tokens["input"]
            out_t = assistant.session_tokens["output"]
            cost  = assistant.session_cost
            print(f"\nAnswer:\n{answer}")
            print(f"\n[{in_t}↑ {out_t}↓ tokens | ${cost:.4f} total]\n")
        except ValueError as e:
            print(f"  Error: {e}\n")
        except anthropic.APIError as e:
            print(f"  API Error: {e}\n")

if __name__ == "__main__":
    main()
```

## What This Capstone Demonstrates

| Feature | Week Learned | Implementation |
|---------|-------------|---------------|
| Python | Week 1 | Entire codebase |
| Data handling | Week 2 | JSON parsing, structured logs |
| APIs and HTTP | Week 5 | `httpx` for page fetching |
| LLM APIs | Week 15 | `anthropic.Anthropic()` |
| Tool use | Week 17 (Day 2) | Tool schemas + execution loop |
| Web agents | Week 17 (Day 3) | DuckDuckGo + BeautifulSoup |
| Cost control | Week 15 (Day 4) | Token budget enforcement |
| Observability | Week 14 (Day 5) | Structured JSON logging |
| Safety | Week 17 (Day 6) | Injection pattern detection |

## You Are Now an AI Engineer

Over 17 weeks you have covered:

**Foundations** (Weeks 1-4): Python, NumPy, Pandas, Matplotlib, Math for ML

**Classical ML** (Weeks 5-8): Regression, Classification, Ensembles, Hyperparameter Tuning

**Deep Learning** (Weeks 9-12): Neural Networks, CNNs, RNNs, Transformers

**Advanced AI** (Week 13): Transfer Learning and Fine-Tuning

**Deployment & MLOps** (Week 14): FastAPI, Docker, Cloud, MLflow, Monitoring, CI/CD

**Modern AI Engineering** (Weeks 15-17): LLMs, Prompt Engineering, RAG, Vector Databases, AI Agents

You understand the entire stack — from a matrix multiplication to a production AI application. Not just how to call an API, but *why* things work the way they do. That depth is what separates an AI engineer from someone who copies code from tutorials.

The field moves fast. Models improve monthly. New architectures emerge constantly. But the foundations you have built — understanding data, math, algorithms, systems, and how to ship software — those transfer to every new development.

**Build things. Break things. Read papers. Stay curious.**

Welcome to the profession.
