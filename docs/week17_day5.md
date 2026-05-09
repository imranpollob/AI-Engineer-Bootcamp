# Day 5: LangChain and LlamaIndex — Frameworks vs. Raw API

Welcome to Day 5. You have built agents and RAG pipelines from scratch this week. That is intentional — understanding the primitives makes you a better engineer regardless of what tools you use. Today we look at the two most popular frameworks in the LLM ecosystem: **LangChain** and **LlamaIndex**. You will learn what they actually do, where they help, and when to avoid them.

## Why Frameworks Exist

The patterns you built this week — the agent loop, tool execution, message history management, chunk-embed-retrieve — appear in almost every LLM application. Frameworks package these patterns into reusable components so you don't reinvent them each time.

The trade-off: convenience vs. control. Frameworks make simple things faster and complex things harder to debug.

## LangChain

**LangChain** is a general-purpose LLM application framework. It provides abstractions for:
- **Chains**: sequences of operations (prompt → LLM → output parser)
- **Agents**: LLMs with tool use, including built-in agent types
- **Memory**: conversation history management
- **Tools**: 100+ pre-built integrations (search, databases, APIs)
- **LangSmith**: observability and tracing (highly valuable in production)

```bash
pip install langchain langchain-anthropic langchain-community
```

```python
# langchain_demo.py
from langchain_anthropic import ChatAnthropic
from langchain.agents import AgentExecutor, create_tool_calling_agent
from langchain.tools import tool
from langchain_core.prompts import ChatPromptTemplate
from dotenv import load_dotenv

load_dotenv()

llm = ChatAnthropic(model="claude-opus-4-7")

# LangChain tools are decorated Python functions
@tool
def calculator(expression: str) -> str:
    """Evaluate a mathematical expression. Input should be a valid Python math expression."""
    import math
    allowed = {k: v for k, v in math.__dict__.items() if not k.startswith("_")}
    result = eval(expression, {"__builtins__": {}}, allowed)
    return str(round(float(result), 6))

@tool
def get_weather(city: str) -> str:
    """Get the current weather for a city."""
    mock = {"Paris": "18°C, partly cloudy", "Tokyo": "24°C, sunny"}
    return mock.get(city, "20°C, unknown conditions")

tools = [calculator, get_weather]

# Prompt template
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant with tools."),
    ("placeholder", "{chat_history}"),
    ("human", "{input}"),
    ("placeholder", "{agent_scratchpad}"),
])

# Create and run the agent
agent = create_tool_calling_agent(llm, tools, prompt)
executor = AgentExecutor(agent=agent, tools=tools, verbose=True)

result = executor.invoke({
    "input": "What's the weather in Tokyo, and what's 15% of 875?",
    "chat_history": []
})
print(result["output"])
```

```result
> Entering new AgentExecutor chain...
Invoking: `get_weather` with `{'city': 'Tokyo'}`
24°C, sunny
Invoking: `calculator` with `{'expression': '875 * 0.15'}`
131.25

Tokyo is currently 24°C and sunny. And 15% of 875 is 131.25.
> Finished chain.
```

**LangChain Strengths:**
- Rapid prototyping with pre-built agent types
- LangSmith provides excellent tracing and debugging
- Large ecosystem of pre-built tools and integrations

**LangChain Weaknesses:**
- Abstractions can obscure what's actually happening
- Debugging is harder than raw API calls
- Heavy dependencies; updates can break things

## LlamaIndex

**LlamaIndex** (formerly GPT Index) is purpose-built for **data indexing and RAG**. It excels at:
- Loading documents from 100+ sources (PDFs, Notion, Slack, databases)
- Multiple index types (vector, keyword, knowledge graph)
- Advanced retrieval techniques (hybrid search, recursive retrieval)
- Built-in evaluation tools

```bash
pip install llama-index llama-index-llms-anthropic llama-index-embeddings-openai
```

```python
# llamaindex_demo.py
from llama_index.core import VectorStoreIndex, SimpleDirectoryReader, Settings
from llama_index.llms.anthropic import Anthropic
from llama_index.embeddings.openai import OpenAIEmbedding
from dotenv import load_dotenv

load_dotenv()

# Configure the models globally
Settings.llm       = Anthropic(model="claude-opus-4-7")
Settings.embed_model = OpenAIEmbedding(model="text-embedding-3-small")

# Load all documents from a directory
# documents = SimpleDirectoryReader("./documents").load_data()

# For demo, create a simple document
from llama_index.core import Document
documents = [
    Document(text="The Transformer architecture was introduced in 2017 in the paper 'Attention is All You Need' by Vaswani et al."),
    Document(text="BERT is a bidirectional transformer pre-trained on masked language modeling. It is used for classification and NER tasks."),
    Document(text="GPT models are autoregressive, decoder-only transformers designed for text generation tasks."),
]

# Build a vector index — handles chunking, embedding, and indexing automatically
index = VectorStoreIndex.from_documents(documents)
query_engine = index.as_query_engine(similarity_top_k=2)

# Query it
response = query_engine.query("How is BERT different from GPT?")
print(response)

# LlamaIndex also returns source nodes for citation
for node in response.source_nodes:
    print(f"  [Score: {node.score:.3f}] {node.text[:80]}...")
```

```result
BERT is a bidirectional transformer pre-trained on masked language modeling for classification tasks, while GPT models are autoregressive decoder-only transformers designed for text generation.

  [Score: 0.891] BERT is a bidirectional transformer pre-trained on masked language modeling...
  [Score: 0.823] GPT models are autoregressive, decoder-only transformers designed for text...
```

**LlamaIndex Strengths:**
- Superior RAG tooling out of the box
- 100+ data connectors (Notion, Slack, Confluence, databases)
- Advanced retrieval methods (MMR, hybrid, recursive)
- Built-in evaluation framework

**LlamaIndex Weaknesses:**
- Less general-purpose than LangChain for non-RAG use cases
- More opinionated about RAG architecture

## Decision Framework

```
Need RAG / document Q&A?
    Yes → Start with LlamaIndex
    No  → Continue below

Need rapid prototyping with existing tool integrations?
    Yes → LangChain is faster
    No  → Consider raw API

Need maximum control and debuggability?
    Yes → Raw Anthropic/OpenAI SDK
    No  → Either framework works

Building for production at scale?
    Yes → Raw SDK + LangSmith for observability
    No  → Framework is fine
```

## The Honest Assessment

Both frameworks improve frequently — LangChain had a reputation for instability but has stabilized significantly. LlamaIndex has become the go-to for production RAG.

The most important skill is understanding the primitives (which you now do), not the framework syntax. If you understand what `embed → store → retrieve → generate` means, you can pick up any framework in a day.

## Wrapping Up Day 5

You have seen how frameworks abstract the patterns you built from scratch. Tomorrow on **Day 6: Production Concerns**, we address what keeps AI applications reliable in production: rate limiting, error handling, cost controls, and safety guardrails — the unglamorous work that separates a demo from a deployed product.
