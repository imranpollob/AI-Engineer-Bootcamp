# Day 7: Capstone — Chat with a PDF

Welcome to Day 7. This is the Week 16 capstone: a complete **"Chat with a PDF"** application — the most requested RAG project type. You will upload any PDF, chunk and index it, then ask questions about it in natural language. The LLM answers only from the PDF's content.

## What We Are Building

```
$ python chat_pdf.py research_paper.pdf

Loading and indexing 'research_paper.pdf'...
Extracted 42 chunks. Indexed successfully.

Type your questions (or /quit to exit, /sources to see references):

You: What is the main contribution of this paper?
Bot: The paper introduces a new attention mechanism called...
     [Sources: chunk_3, chunk_7]

You: /quit
```

## The Full Application

```python
# chat_pdf.py
"""
Chat with any PDF using RAG.
Usage: python chat_pdf.py <path_to_pdf>
"""
import sys
import re
import json
import chromadb
from pypdf import PdfReader
from openai import OpenAI
import anthropic
from dotenv import load_dotenv

load_dotenv()

openai_client    = OpenAI()
anthropic_client = anthropic.Anthropic()


# ─── PDF Loading and Chunking ────────────────────────────────────
def load_pdf(path: str) -> str:
    reader = PdfReader(path)
    pages = [page.extract_text() or "" for page in reader.pages]
    return "\n".join(pages)

def chunk_text(text: str, chunk_size: int = 450, overlap: int = 80) -> list[str]:
    chunks = []
    start = 0
    while start < len(text):
        end = min(start + chunk_size, len(text))
        chunk = text[start:end].strip()
        if len(chunk) > 30:   # Skip near-empty chunks
            chunks.append(chunk)
        start += chunk_size - overlap
    return chunks


# ─── Embedding ───────────────────────────────────────────────────
def embed(text: str) -> list[float]:
    r = openai_client.embeddings.create(
        model="text-embedding-3-small", input=text
    )
    return r.data[0].embedding


# ─── Vector Store ────────────────────────────────────────────────
def build_index(pdf_path: str) -> chromadb.Collection:
    print(f"Loading '{pdf_path}'...")
    text   = load_pdf(pdf_path)
    chunks = chunk_text(text)
    print(f"Extracted {len(chunks)} chunks. Embedding...")

    chroma     = chromadb.Client()   # In-memory for this session
    collection = chroma.create_collection(
        name="pdf_chat",
        metadata={"hnsw:space": "cosine"}
    )

    # Embed in small batches to avoid API rate limits
    BATCH_SIZE = 10
    for i in range(0, len(chunks), BATCH_SIZE):
        batch   = chunks[i:i + BATCH_SIZE]
        batch_i = list(range(i, i + len(batch)))

        embeddings = [embed(c) for c in batch]
        collection.add(
            ids       =[f"chunk_{j}" for j in batch_i],
            embeddings=embeddings,
            documents =batch,
            metadatas =[{"chunk_index": j} for j in batch_i]
        )

    print(f"Indexed {collection.count()} chunks.\n")
    return collection


# ─── Retrieval ───────────────────────────────────────────────────
def retrieve(collection: chromadb.Collection, query: str, top_k: int = 4) -> list[dict]:
    results = collection.query(
        query_embeddings=[embed(query)],
        n_results=top_k,
        include=["documents", "distances", "metadatas", "ids"]
    )
    return [
        {
            "id":         results["ids"][0][i],
            "text":       results["documents"][0][i],
            "similarity": round(1 - results["distances"][0][i], 4),
        }
        for i in range(len(results["ids"][0]))
    ]


# ─── Generation ──────────────────────────────────────────────────
SYSTEM = """You are a document assistant. Answer questions based ONLY on the provided context.
If the context doesn't contain enough information, say "The document doesn't cover this."
Cite the chunk IDs (e.g., [chunk_3]) when you use specific information."""

def generate(query: str, chunks: list[dict], history: list[dict]) -> str:
    context = "\n\n".join(
        f"[{c['id']} | relevance: {c['similarity']}]\n{c['text']}"
        for c in chunks
    )
    user_message = f"Context from document:\n{context}\n\nQuestion: {query}"

    messages = history + [{"role": "user", "content": user_message}]

    r = anthropic_client.messages.create(
        model="claude-opus-4-7",
        max_tokens=600,
        system=SYSTEM,
        messages=messages
    )
    return r.content[0].text


# ─── CLI Chat Loop ────────────────────────────────────────────────
def main():
    if len(sys.argv) < 2:
        print("Usage: python chat_pdf.py <path_to_pdf>")
        sys.exit(1)

    pdf_path   = sys.argv[1]
    collection = build_index(pdf_path)
    history    = []
    last_chunks = []

    print("Ask questions about your PDF. Commands: /sources  /clear  /quit\n")

    while True:
        try:
            user_input = input("You: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\nGoodbye!")
            break

        if not user_input:
            continue

        if user_input == "/quit":
            print("Goodbye!")
            break
        elif user_input == "/clear":
            history.clear()
            print("  Conversation history cleared.\n")
            continue
        elif user_input == "/sources":
            if last_chunks:
                print("  Last retrieved sources:")
                for c in last_chunks:
                    print(f"    [{c['id']} | {c['similarity']}] {c['text'][:100]}...")
            else:
                print("  No sources yet.")
            continue

        chunks = retrieve(collection, user_input, top_k=4)
        last_chunks = chunks

        answer = generate(user_input, chunks, history)

        # Update conversation history (keep last 6 turns to manage context)
        history.append({"role": "user",      "content": user_input})
        history.append({"role": "assistant", "content": answer})
        if len(history) > 12:
            history = history[-12:]

        print(f"\nBot: {answer}\n")


if __name__ == "__main__":
    main()
```

## Running It

```bash
# Download a sample PDF (any PDF works)
curl -o sample.pdf https://arxiv.org/pdf/1706.03762  # "Attention is All You Need"

python chat_pdf.py sample.pdf
```

```result
Loading 'sample.pdf'...
Extracted 58 chunks. Embedding...
Indexed 58 chunks.

Ask questions about your PDF. Commands: /sources  /clear  /quit

You: What problem does this paper solve?
Bot: The paper addresses the limitations of recurrent and convolutional neural networks in sequence transduction tasks. Traditional models process sequences sequentially, making parallelization difficult and creating long-range dependency problems. The Transformer architecture solves this by relying entirely on attention mechanisms [chunk_2, chunk_5].

You: /sources
  Last retrieved sources:
    [chunk_2 | 0.851] The dominant sequence transduction models are based on complex recurrent...
    [chunk_5 | 0.823] We propose the Transformer, a model architecture eschewing recurrence...

You: /quit
Goodbye!
```

## What This Capstone Demonstrates

| Feature | Implementation |
|---------|---------------|
| PDF ingestion | `pypdf.PdfReader` |
| Fixed-size chunking with overlap | `chunk_text()` |
| Embedding via API | `text-embedding-3-small` |
| Vector storage | ChromaDB in-memory |
| Semantic retrieval | cosine similarity, top-K |
| Grounded generation | Claude with context injection |
| Multi-turn memory | rolling conversation history |
| Source attribution | chunk IDs in response |

## Wrapping Up Week 16

You can now build a RAG system from scratch: load documents, chunk them, embed them, store them in a vector database, retrieve the most relevant passages, and generate grounded answers. This pattern powers ChatPDF, Notion AI, GitHub Copilot's codebase Q&A, and countless enterprise knowledge bases.

Next week in **Week 17: AI Agents & Production Applications**, we move beyond answering questions about static documents to building systems that can *take actions* — search the web, call APIs, write and run code, all orchestrated by an LLM reasoning loop.
