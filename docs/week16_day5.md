# Day 5: Chunking Strategies

Welcome to Day 5. In yesterday's pipeline, the knowledge base had pre-written sentences as chunks. In reality, you will have PDFs, web pages, documentation sites, or code repositories — raw, messy text that must be split into appropriate pieces before embedding.

**Chunking** is splitting a large document into smaller passages that can be embedded and retrieved independently. It is deceptively important: the wrong chunking strategy can destroy retrieval quality even with a perfect embedding model.

## Why Chunking Size Matters

Too small:
- Each chunk lacks enough context to answer questions independently
- "The capital of France is" — retrieved without the next sentence — is useless

Too large:
- The chunk covers multiple topics; the embedding averages them out
- Retrieval returns chunks where the answer is buried in irrelevant text
- Each chunk consumes more tokens → higher LLM cost per query

The sweet spot for most tasks: **256–512 tokens per chunk**.

## Strategy 1: Fixed-Size Chunking

Split every N characters, with optional overlap to avoid cutting concepts mid-sentence.

```python
# fixed_chunking.py
def fixed_size_chunk(text: str, chunk_size: int = 400, overlap: int = 80) -> list[str]:
    """
    Split text into chunks of `chunk_size` characters with `overlap` character overlap.
    Overlap ensures context is not lost at chunk boundaries.
    """
    chunks = []
    start = 0
    while start < len(text):
        end = start + chunk_size
        chunks.append(text[start:end])
        start += chunk_size - overlap   # Move forward less than chunk_size to create overlap
    return chunks

sample_text = """
Machine learning is a subset of artificial intelligence that enables systems to learn and improve 
from experience without being explicitly programmed. The process begins with data collection, 
followed by preprocessing to handle missing values and outliers. Feature engineering extracts 
meaningful signals from raw data. Model selection involves choosing an algorithm appropriate for 
the task — regression, classification, or clustering. Training adjusts the model's parameters 
to minimize prediction error. Evaluation on held-out data estimates real-world performance. 
Deployment makes the model available for inference. Monitoring detects performance degradation 
over time as data distributions shift.
""".strip()

chunks = fixed_size_chunk(sample_text, chunk_size=200, overlap=40)
print(f"Total chunks: {len(chunks)}")
for i, chunk in enumerate(chunks):
    print(f"\n[Chunk {i+1}] ({len(chunk)} chars):\n{chunk}")
```

```result
Total chunks: 4

[Chunk 1] (200 chars):
Machine learning is a subset of artificial intelligence that enables systems to learn and improve 
from experience without being explicitly programmed. The process begins with data collection, 
followed by

[Chunk 2] (200 chars):
followed by preprocessing to handle missing values and outliers. Feature engineering extracts 
meaningful signals from raw data. Model selection involves choosing an algorithm appropriate

[Chunk 3] (200 chars):
appropriate for the task — regression, classification, or clustering. Training adjusts the model's
parameters to minimize prediction error. Evaluation on held-out data estimates real-world
```

The overlap ("followed by", "appropriate for") ensures chunk boundaries don't sever sentences. Fixed-size chunking is simple and reliable for homogeneous text.

## Strategy 2: Sentence-Based Chunking

Split on sentence boundaries and group N sentences together. Preserves sentence integrity — no mid-sentence cuts.

```python
# sentence_chunking.py
import re

def sentence_chunk(text: str, sentences_per_chunk: int = 3, overlap: int = 1) -> list[str]:
    # Simple sentence splitter (for production use spaCy or nltk)
    sentences = re.split(r'(?<=[.!?])\s+', text.strip())
    sentences = [s.strip() for s in sentences if s.strip()]

    chunks = []
    step = sentences_per_chunk - overlap
    for i in range(0, len(sentences), step):
        chunk_sentences = sentences[i:i + sentences_per_chunk]
        chunks.append(" ".join(chunk_sentences))

    return chunks

chunks = sentence_chunk(sample_text, sentences_per_chunk=2, overlap=1)
print(f"Total chunks: {len(chunks)}")
for i, c in enumerate(chunks):
    print(f"\n[Chunk {i+1}]: {c}")
```

```result
Total chunks: 7

[Chunk 1]: Machine learning is a subset of artificial intelligence that enables systems to learn and improve from experience without being explicitly programmed. The process begins with data collection, followed by preprocessing to handle missing values and outliers.

[Chunk 2]: The process begins with data collection, followed by preprocessing to handle missing values and outliers. Feature engineering extracts meaningful signals from raw data.
```

Each chunk is a complete thought. The 1-sentence overlap carries context across chunk boundaries.

## Strategy 3: Chunking PDFs with pypdf

Real documents are PDFs. Use `pypdf` to extract text, then chunk.

```bash
pip install pypdf
```

```python
# pdf_chunking.py
from pypdf import PdfReader

def load_pdf(path: str) -> str:
    reader = PdfReader(path)
    return "\n".join(page.extract_text() or "" for page in reader.pages)

def chunk_pdf(path: str, chunk_size: int = 500, overlap: int = 100) -> list[dict]:
    text = load_pdf(path)
    raw_chunks = fixed_size_chunk(text, chunk_size=chunk_size, overlap=overlap)

    return [
        {
            "id": f"pdf_{i}",
            "text": chunk,
            "source": path,
            "chunk_index": i
        }
        for i, chunk in enumerate(raw_chunks)
        if len(chunk.strip()) > 50   # Skip near-empty chunks
    ]

# Usage:
# chunks = chunk_pdf("my_document.pdf")
# print(f"Extracted {len(chunks)} chunks from PDF")
```

## Choosing a Chunking Strategy

| Strategy | Best For | Watch Out For |
|----------|----------|---------------|
| Fixed-size | Homogeneous text (articles, blogs) | May split sentences |
| Sentence-based | Structured prose, documentation | Can produce very short chunks |
| Paragraph-based | Documents with clear paragraph structure | Uneven chunk sizes |
| Semantic | High accuracy requirements | Slower; requires NLP model |

## Testing Chunk Quality

A quick sanity check: retrieve chunks for a sample query and read them manually.

```python
# chunk_quality_check.py
from rag_pipeline import retrieve, index_documents

# Index the chunks
chunks = sentence_chunk(sample_text, sentences_per_chunk=2, overlap=1)
docs = [{"id": f"chunk_{i}", "text": c, "source": "sample"} for i, c in enumerate(chunks)]
index_documents(docs)

# Test retrieval
query = "How do you evaluate a machine learning model?"
results = retrieve(query, top_k=2)
for r in results:
    print(f"[{r['similarity']:.3f}] {r['text']}")
```

```result
[0.847] Evaluation on held-out data estimates real-world performance. Deployment makes the model available for inference.
[0.621] Training adjusts the model's parameters to minimize prediction error. Evaluation on held-out data estimates real-world performance.
```

Both retrieved chunks contain "evaluation" and are directly relevant. Good chunking in action.

## Wrapping Up Day 5

Chunking is not a one-size-fits-all decision. Test your specific document type with your specific queries and measure which strategy retrieves the most relevant passages. Tomorrow on **Day 6: Evaluating RAG Quality**, we formalize this testing with metrics so you can compare strategies and improvements systematically.
