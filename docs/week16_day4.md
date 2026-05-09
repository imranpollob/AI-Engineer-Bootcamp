# Day 4: Building a Basic RAG Pipeline

Welcome to Day 4. You have all the pieces: embeddings (Day 2), vector databases (Day 3), and LLM APIs (Week 15). Today we assemble them into a working Retrieval-Augmented Generation pipeline. This is the core pattern behind ChatPDF, GitHub Copilot's codebase understanding, and most enterprise AI assistants.

## The Pipeline in Code

The RAG pipeline has two phases:

**Offline (run once):**
```
Documents → Chunk → Embed → Store in vector DB
```

**Online (run per query):**
```
User query → Embed query → Retrieve top-K chunks → Build prompt → LLM → Answer
```

## Complete RAG Implementation

```python
# rag_pipeline.py
"""
A minimal but complete RAG pipeline.
"""
import chromadb
from openai import OpenAI
import anthropic
from dotenv import load_dotenv

load_dotenv()

openai_client  = anthropic_client = None
openai_client  = OpenAI()
anthropic_client = anthropic.Anthropic()

chroma = chromadb.PersistentClient(path="./rag_db")
collection = chroma.get_or_create_collection(
    name="documents",
    metadata={"hnsw:space": "cosine"}
)


# ─── Embedding ───────────────────────────────────────────────────
def embed(text: str) -> list[float]:
    r = openai_client.embeddings.create(
        model="text-embedding-3-small", input=text
    )
    return r.data[0].embedding


# ─── Indexing ────────────────────────────────────────────────────
def index_documents(docs: list[dict]):
    """
    docs: list of {"id": str, "text": str, "source": str}
    """
    existing_ids = set(collection.get()["ids"])

    new_docs = [d for d in docs if d["id"] not in existing_ids]
    if not new_docs:
        print("All documents already indexed.")
        return

    print(f"Indexing {len(new_docs)} new documents...")
    collection.add(
        ids=[d["id"] for d in new_docs],
        embeddings=[embed(d["text"]) for d in new_docs],
        documents=[d["text"] for d in new_docs],
        metadatas=[{"source": d["source"]} for d in new_docs]
    )
    print(f"Index now contains {collection.count()} documents.")


# ─── Retrieval ───────────────────────────────────────────────────
def retrieve(query: str, top_k: int = 3) -> list[dict]:
    results = collection.query(
        query_embeddings=[embed(query)],
        n_results=top_k,
        include=["documents", "distances", "metadatas"]
    )
    chunks = []
    for text, dist, meta in zip(
        results["documents"][0],
        results["distances"][0],
        results["metadatas"][0]
    ):
        chunks.append({
            "text": text,
            "similarity": round(1 - dist, 4),
            "source": meta.get("source", "unknown")
        })
    return chunks


# ─── Generation ──────────────────────────────────────────────────
def generate_answer(query: str, context_chunks: list[dict]) -> str:
    # Build context block from retrieved chunks
    context = "\n\n".join(
        f"[Source: {c['source']} | Relevance: {c['similarity']}]\n{c['text']}"
        for c in context_chunks
    )

    prompt = f"""Answer the question using ONLY the provided context.
If the context does not contain enough information to answer confidently, say "I don't have enough information to answer this."
Do not use prior knowledge beyond what is in the context.

Context:
{context}

Question: {query}

Answer:"""

    r = anthropic_client.messages.create(
        model="claude-opus-4-7",
        max_tokens=400,
        messages=[{"role": "user", "content": prompt}]
    )
    return r.content[0].text


# ─── End-to-End RAG ──────────────────────────────────────────────
def rag(query: str, top_k: int = 3, verbose: bool = False) -> str:
    chunks = retrieve(query, top_k=top_k)

    if verbose:
        print(f"\n[Retrieved {len(chunks)} chunks]")
        for c in chunks:
            print(f"  [{c['similarity']}] {c['text'][:80]}...")

    return generate_answer(query, chunks)
```

## Testing the Pipeline

```python
# test_rag.py
from rag_pipeline import index_documents, rag

# Index a small knowledge base about AI concepts
knowledge_base = [
    {"id": "1", "text": "Transformers use self-attention to process sequences in parallel, unlike RNNs which process sequentially.", "source": "textbook"},
    {"id": "2", "text": "BERT is a bidirectional transformer pre-trained using masked language modeling on Wikipedia and BooksCorpus.", "source": "paper"},
    {"id": "3", "text": "GPT models are decoder-only transformers trained to predict the next token autoregressively.", "source": "paper"},
    {"id": "4", "text": "Fine-tuning adapts a pre-trained model to a specific task using a smaller, task-specific dataset.", "source": "textbook"},
    {"id": "5", "text": "RAG reduces hallucination by grounding model responses in retrieved external documents.", "source": "blog"},
    {"id": "6", "text": "The context window is the maximum number of tokens a model can process in a single forward pass.", "source": "docs"},
    {"id": "7", "text": "Embeddings represent text as dense vectors where semantic similarity corresponds to vector proximity.", "source": "textbook"},
    {"id": "8", "text": "Claude was developed by Anthropic with a focus on safety, helpfulness, and honesty.", "source": "docs"},
]

index_documents(knowledge_base)

# Test queries
queries = [
    "How is BERT different from GPT?",
    "Why does RAG help with hallucination?",
    "What is a context window?",
    "Who created Claude?",
    "What is the weather like today?",  # Out-of-scope test
]

for q in queries:
    print(f"\nQ: {q}")
    print(f"A: {rag(q, verbose=True)}")
    print("─" * 60)
```

```result
[Retrieved 3 chunks]
  [0.821] BERT is a bidirectional transformer pre-trained using masked language...
  [0.798] GPT models are decoder-only transformers trained to predict the next...
  [0.612] Transformers use self-attention to process sequences in parallel...

Q: How is BERT different from GPT?
A: BERT is a bidirectional transformer pre-trained using masked language modeling, while GPT models are decoder-only transformers trained to predict the next token autoregressively. BERT processes context from both directions simultaneously, whereas GPT processes left-to-right only.
────────────────────────────────────────────────────────────

Q: What is the weather like today?
A: I don't have enough information to answer this.
────────────────────────────────────────────────────────────
```

The last query correctly returns "I don't have enough information" — the model cannot hallucinate an answer because the retrieved chunks contain nothing about weather.

## The Grounding Prompt Pattern

The key to preventing hallucination is this phrase in the prompt:

> "Answer using ONLY the provided context. If the context does not contain enough information, say so."

Without this instruction, the model falls back on training data and may invent plausible-sounding but wrong answers. With it, the model's knowledge is locked to what you retrieved.

## Wrapping Up Day 4

You have a working RAG pipeline: index → retrieve → generate, with hallucination guardrails. Tomorrow on **Day 5: Chunking Strategies**, we tackle the most nuanced part of RAG — how you split your documents into chunks has a massive impact on retrieval quality, and there are several approaches with real trade-offs.
