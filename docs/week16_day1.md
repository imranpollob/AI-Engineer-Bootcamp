# Day 1: Why RAG? The Problem with LLMs and Knowledge

Welcome to Week 16. Last week you built a capable chatbot. But test it with a question like "What does our company's refund policy say?" or "Summarize this PDF I uploaded" — it fails completely. The model was trained months ago on public internet data; it knows nothing about your internal documents, recent events, or private databases.

**Retrieval-Augmented Generation (RAG)** is the solution. It is the most important architecture pattern in modern AI engineering.

## The Open-Book vs. Closed-Book Exam Analogy

Imagine two students taking the same exam. The first must answer from memory alone — they are brilliant but can only recall what they studied before. The second is allowed to bring any books and notes they want.

On questions about recent events or obscure specifics, the open-book student wins every time — not because they are smarter, but because they can *look things up*.

RAG makes your LLM an open-book student. Instead of relying on frozen training data, it retrieves relevant passages from your documents *at query time* and includes them in the prompt. The LLM then reasons over that freshly retrieved context.

## The Three Problems RAG Solves

**Problem 1: Knowledge cutoff.** LLMs have a training cutoff (e.g., April 2024). They know nothing after that date. RAG lets you inject current data at query time.

**Problem 2: Private/domain data.** Your company's documents, codebase, product manuals, and internal wikis were never in the training data. RAG lets the model reason over them.

**Problem 3: Hallucination.** When a model doesn't know something, it often invents a plausible-sounding answer. If you force it to answer only from retrieved context, hallucinations drop dramatically.

## The RAG Architecture

```
                    User Query
                        │
                        ▼
              ┌─────────────────┐
              │  Embed Query    │  (convert query to a vector)
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  Vector Search  │  (find K most similar document chunks)
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │ Build Prompt    │  context = retrieved chunks + original query
              └────────┬────────┘
                       │
                       ▼
              ┌─────────────────┐
              │  LLM Generate   │  answer grounded in retrieved context
              └─────────────────┘
```

And the offline preparation step:

```
Your Documents
      │
      ▼
  Chunking       (split into small passages)
      │
      ▼
  Embedding      (convert each chunk to a vector)
      │
      ▼
  Vector Store   (index all vectors for fast similarity search)
```

## Why Not Just Stuff Everything in the Prompt?

You might ask: if context windows are 200K tokens, why not just put all your documents in the prompt every time?

1. **Cost**: 200K tokens × $15/million = $3 per query. With 1,000 queries/day, that is $90,000/month.
2. **Latency**: Processing 200K tokens takes several seconds per query.
3. **Attention degradation**: LLMs perform worse when drowning in irrelevant context. Studies show accuracy drops when the relevant passage is buried in a long prompt.
4. **Scale limit**: Even 200K tokens is only ~150,000 words. Large codebases or document collections are far larger.

RAG retrieves the *most relevant 2-5%* of your data and shows only that to the model — cheap, fast, and more accurate.

## What You Will Build This Week

By the end of Week 16 you will have built a **"Chat with a PDF"** application:

1. Upload a PDF (Day 4-5)
2. Chunk it into passages (Day 5)
3. Embed each passage and store in a vector database (Day 2-3)
4. Query: retrieve the most relevant passages and answer using Claude (Day 4)
5. Evaluate the system's accuracy (Day 6)
6. Build the full application (Day 7)

## Wrapping Up Day 1

RAG is not complicated — it is a clever combination of techniques you already know: text search (similarity), databases (vector stores), and prompting. Tomorrow on **Day 2: Text Embeddings**, we start with the mathematical foundation — what embeddings are, why cosine similarity works, and how to create them with an API.
