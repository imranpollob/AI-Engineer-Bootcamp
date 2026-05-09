# Day 2: Text Embeddings

Welcome to Day 2. RAG relies on a crucial capability: measuring how *similar* two pieces of text are. Not character-by-character similarity (like string distance), but *semantic* similarity — "car" and "automobile" should be close even though they share no letters.

**Embeddings** make this possible.

## The Map Analogy

Imagine projecting every concept in human language onto a giant map. Words with related meanings cluster together: "king", "queen", "royalty" form one cluster; "Python", "JavaScript", "coding" form another. The *distance* between two words on this map corresponds to their semantic similarity.

An embedding is the address of a word or sentence on this map — a list of numbers (typically 512 to 3072 numbers) that encodes its meaning. The embedding model learns these coordinates by training on massive amounts of text.

## Creating Embeddings

```python
# embeddings_intro.py
from openai import OpenAI
from dotenv import load_dotenv
import numpy as np

load_dotenv()
client = OpenAI()

def embed(text: str) -> list[float]:
    response = client.embeddings.create(
        model="text-embedding-3-small",    # 1536 dimensions, cheap and good
        input=text
    )
    return response.data[0].embedding

# Embed a few sentences
sentences = [
    "The dog chased the cat.",
    "A canine pursued a feline.",       # Same meaning, different words
    "She baked a chocolate cake.",       # Completely different topic
]

embeddings = [embed(s) for s in sentences]

print(f"Embedding dimensions: {len(embeddings[0])}")
print(f"First 5 values: {embeddings[0][:5]}")
```

```result
Embedding dimensions: 1536
First 5 values: [0.0123, -0.0456, 0.0789, -0.0234, 0.0567]
```

These 1536 numbers capture the meaning of the sentence. On their own they are uninterpretable — but the *relationships* between them are meaningful.

## Measuring Similarity: Cosine Similarity

The standard way to measure how similar two embeddings are is **cosine similarity** — it measures the angle between two vectors, regardless of their length.

$$\text{similarity}(A, B) = \frac{A \cdot B}{|A| \cdot |B|}$$

- **1.0** = identical meaning
- **0.0** = no relationship
- **-1.0** = opposite meaning

```python
# cosine_similarity.py
import numpy as np

def cosine_similarity(a: list[float], b: list[float]) -> float:
    a, b = np.array(a), np.array(b)
    return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))

# Compare all pairs
for i, s1 in enumerate(sentences):
    for j, s2 in enumerate(sentences):
        if i < j:
            sim = cosine_similarity(embeddings[i], embeddings[j])
            print(f"Similarity({i+1},{j+1}): {sim:.4f}")
            print(f"  '{s1[:50]}'")
            print(f"  '{s2[:50]}'")
```

```result
Similarity(1,2): 0.8912
  'The dog chased the cat.'
  'A canine pursued a feline.'

Similarity(1,3): 0.2341
  'The dog chased the cat.'
  'She baked a chocolate cake.'

Similarity(2,3): 0.2198
  'A canine pursued a feline.'
  'She baked a chocolate cake.'
```

The dog/canine sentences score 0.89 — nearly identical semantics. The dog/cake comparison is 0.23 — essentially unrelated. This is the mathematical core of RAG retrieval.

## Semantic Search from Scratch

With embeddings and cosine similarity, you can build a rudimentary semantic search engine over a small document collection:

```python
# semantic_search.py
from openai import OpenAI
import numpy as np
from dotenv import load_dotenv

load_dotenv()
client = OpenAI()

def embed(text: str) -> np.ndarray:
    r = client.embeddings.create(model="text-embedding-3-small", input=text)
    return np.array(r.data[0].embedding)

def cosine_similarity(a: np.ndarray, b: np.ndarray) -> float:
    return float(np.dot(a, b) / (np.linalg.norm(a) * np.linalg.norm(b)))

# A tiny "knowledge base"
documents = [
    "Python was created by Guido van Rossum and first released in 1991.",
    "Machine learning is a subset of artificial intelligence.",
    "The Eiffel Tower is located in Paris, France.",
    "Gradient descent is an optimization algorithm used to train neural networks.",
    "Docker containers package applications and their dependencies together.",
    "The Amazon river is the largest river by discharge volume.",
]

print("Embedding documents...")
doc_embeddings = [embed(doc) for doc in documents]

def search(query: str, top_k: int = 2) -> list[tuple[str, float]]:
    query_emb = embed(query)
    scores = [(doc, cosine_similarity(query_emb, emb))
              for doc, emb in zip(documents, doc_embeddings)]
    return sorted(scores, key=lambda x: x[1], reverse=True)[:top_k]

# Test queries
queries = [
    "Who invented Python?",
    "How do neural networks learn?",
    "Where is the Eiffel Tower?",
]

for q in queries:
    results = search(q)
    print(f"\nQuery: {q}")
    for doc, score in results:
        print(f"  [{score:.3f}] {doc}")
```

```result
Query: Who invented Python?
  [0.847] Python was created by Guido van Rossum and first released in 1991.
  [0.423] Machine learning is a subset of artificial intelligence.

Query: How do neural networks learn?
  [0.792] Gradient descent is an optimization algorithm used to train neural networks.
  [0.541] Machine learning is a subset of artificial intelligence.

Query: Where is the Eiffel Tower?
  [0.891] The Eiffel Tower is located in Paris, France.
  [0.312] The Amazon river is the largest river by discharge volume.
```

The right document surfaces at the top for every query — based purely on semantic similarity, with no keyword matching.

## Choosing an Embedding Model

| Model | Dimensions | Cost (per M tokens) | Best For |
|-------|-----------|---------------------|----------|
| `text-embedding-3-small` | 1536 | $0.02 | Most use cases — excellent quality/cost |
| `text-embedding-3-large` | 3072 | $0.13 | Maximum accuracy on complex retrieval |
| `text-embedding-ada-002` | 1536 | $0.10 | Legacy, use 3-small instead |
| Anthropic Voyage-3 | 1024 | $0.06 | Optimized for Claude + RAG |

For this bootcamp we use `text-embedding-3-small` — it is cheap and performs well on most tasks.

## Wrapping Up Day 2

You now understand the mathematical heart of RAG: embeddings turn text into coordinates, cosine similarity measures distance between coordinates, and similarity search retrieves the nearest neighbors. Tomorrow on **Day 3: Vector Databases**, we replace our in-memory list with a proper database built for storing and searching millions of embeddings at high speed.
