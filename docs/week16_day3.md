# Day 3: Vector Databases — ChromaDB and FAISS

Welcome to Day 3. Yesterday's semantic search worked beautifully — on 6 documents. In production you might have 100,000 documents. Computing cosine similarity against every one on every query would be too slow. You need a database purpose-built for vector search.

## The Library Card Catalog Analogy

Finding a specific book by reading every book in a library is impractical. A card catalog (or modern search index) lets you find books by subject in seconds — it is a specialized index built precisely for the retrieval task. A vector database is the card catalog for embeddings.

## Two Options

**FAISS** (Facebook AI Similarity Search): An in-memory library from Meta. Blazingly fast. Best when your data fits in RAM and you want zero infrastructure.

**ChromaDB**: A full vector database with persistence, metadata filtering, and a simple API. Best for development and medium-scale production.

In industry you will also encounter Pinecone, Weaviate, Qdrant, and pgvector — they all solve the same problem with different trade-offs. The concepts below transfer directly.

## ChromaDB: Getting Started

```bash
pip install chromadb openai python-dotenv
```

```python
# chroma_intro.py
import chromadb
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()
openai_client = OpenAI()

# ChromaDB persists data to disk in ./chroma_db/
chroma_client = chromadb.PersistentClient(path="./chroma_db")

# A collection is like a table — groups related embeddings
collection = chroma_client.get_or_create_collection(
    name="knowledge_base",
    metadata={"hnsw:space": "cosine"}   # Use cosine distance
)

def embed(text: str) -> list[float]:
    r = openai_client.embeddings.create(model="text-embedding-3-small", input=text)
    return r.data[0].embedding

# Add documents with unique IDs and optional metadata
documents = [
    {"id": "doc1", "text": "Python was created by Guido van Rossum in 1991.", "source": "wikipedia"},
    {"id": "doc2", "text": "Gradient descent minimizes loss by following the negative gradient.", "source": "textbook"},
    {"id": "doc3", "text": "Docker containers ensure consistent environments across machines.", "source": "docs"},
    {"id": "doc4", "text": "The Transformer architecture uses self-attention mechanisms.", "source": "paper"},
    {"id": "doc5", "text": "RAG combines retrieval with language model generation.", "source": "blog"},
]

print("Indexing documents...")
collection.add(
    ids=[d["id"] for d in documents],
    embeddings=[embed(d["text"]) for d in documents],
    documents=[d["text"] for d in documents],
    metadatas=[{"source": d["source"]} for d in documents]
)

print(f"Collection size: {collection.count()} documents")
```

```result
Indexing documents...
Collection size: 5 documents
```

## Querying ChromaDB

```python
# chroma_query.py
def search(query: str, top_k: int = 2, source_filter: str = None):
    query_embedding = embed(query)

    # Optional: filter by metadata
    where = {"source": source_filter} if source_filter else None

    results = collection.query(
        query_embeddings=[query_embedding],
        n_results=top_k,
        where=where,
        include=["documents", "distances", "metadatas"]
    )

    for doc, dist, meta in zip(
        results["documents"][0],
        results["distances"][0],
        results["metadatas"][0]
    ):
        similarity = 1 - dist   # ChromaDB returns distance, not similarity
        print(f"  [{similarity:.3f}] ({meta['source']}) {doc}")

print("Query: What is gradient descent?")
search("What is gradient descent?")

print("\nQuery: History of Python — papers only:")
search("History of Python", source_filter="wikipedia")
```

```result
Query: What is gradient descent?
  [0.821] (textbook) Gradient descent minimizes loss by following the negative gradient.
  [0.543] (paper) The Transformer architecture uses self-attention mechanisms.

Query: History of Python — papers only:
  [0.831] (wikipedia) Python was created by Guido van Rossum in 1991.
```

Metadata filtering is powerful — it lets you scope retrieval to specific document types, date ranges, or authors without embedding those constraints.

## FAISS: When You Need Raw Speed

FAISS operates entirely in memory and is significantly faster than ChromaDB for large-scale search.

```bash
pip install faiss-cpu numpy
```

```python
# faiss_intro.py
import faiss
import numpy as np
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()
client = OpenAI()

def embed_batch(texts: list[str]) -> np.ndarray:
    r = client.embeddings.create(model="text-embedding-3-small", input=texts)
    vectors = [item.embedding for item in r.data]
    return np.array(vectors, dtype="float32")

documents = [
    "Python was created by Guido van Rossum in 1991.",
    "Gradient descent minimizes loss by following the negative gradient.",
    "Docker containers ensure consistent environments across machines.",
    "The Transformer architecture uses self-attention mechanisms.",
    "RAG combines retrieval with language model generation.",
]

# Embed all documents in one batch (cheaper and faster than one-by-one)
print("Embedding documents...")
doc_vectors = embed_batch(documents)   # shape: (5, 1536)

# Normalize for cosine similarity search
faiss.normalize_L2(doc_vectors)

# Create an index — IndexFlatIP = Inner Product (cosine sim after normalization)
dimension = doc_vectors.shape[1]   # 1536
index = faiss.IndexFlatIP(dimension)
index.add(doc_vectors)
print(f"FAISS index contains {index.ntotal} vectors")

def search_faiss(query: str, top_k: int = 2):
    q_vec = embed_batch([query])
    faiss.normalize_L2(q_vec)
    scores, indices = index.search(q_vec, top_k)
    for score, idx in zip(scores[0], indices[0]):
        print(f"  [{score:.3f}] {documents[idx]}")

print("\nQuery: How do machines learn?")
search_faiss("How do machines learn?")
```

```result
FAISS index contains 5 vectors

Query: How do machines learn?
  [0.793] Gradient descent minimizes loss by following the negative gradient.
  [0.612] The Transformer architecture uses self-attention mechanisms.
```

## Choosing Between Them

| | ChromaDB | FAISS |
|--|---------|-------|
| Persistence | Built-in | Manual (save/load index) |
| Metadata filtering | Yes | No (need separate logic) |
| Speed at 1M+ vectors | Moderate | Excellent |
| Setup complexity | Low | Low |
| Best for | Development, medium scale | High-throughput production |

For the rest of this week we use ChromaDB because its metadata support and persistence make the examples cleaner.

## Wrapping Up Day 3

You can now index thousands of documents into a vector database and retrieve the most semantically relevant ones in milliseconds. Tomorrow on **Day 4: Building a RAG Pipeline**, we connect all the pieces — embedding, retrieval, and generation — into a working question-answering system.
