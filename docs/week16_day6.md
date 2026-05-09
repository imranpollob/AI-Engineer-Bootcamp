# Day 6: Evaluating RAG Quality

Welcome to Day 6. You have a RAG pipeline that produces answers — but how do you know if those answers are any good? "It seems to work" is not engineering. Systematic evaluation is what separates a prototype from a production system.

## The Three Failure Modes of RAG

**Retrieval fails:** The right chunks are not returned. The model sees irrelevant context and either hallucinates or says "I don't know." No amount of prompt engineering fixes bad retrieval.

**Generation fails:** The right chunks are retrieved but the model's answer does not reflect them — it ignores the context or misinterprets it.

**Both work but together they don't:** The retrieved chunks are individually correct but the model cannot synthesize them into a coherent answer (common when the answer spans multiple chunks).

Evaluating each failure mode requires different metrics.

## The Evaluation Dataset

Good evaluation starts with a **golden dataset**: question-answer pairs where you know the correct answer. Build this by hand for your specific domain.

```python
# eval_dataset.py
eval_set = [
    {
        "question": "What is BERT's pre-training objective?",
        "expected_answer": "Masked language modeling on Wikipedia and BooksCorpus.",
        "relevant_chunk_id": "2"
    },
    {
        "question": "How do GPT models generate text?",
        "expected_answer": "By predicting the next token autoregressively.",
        "relevant_chunk_id": "3"
    },
    {
        "question": "Why does RAG reduce hallucination?",
        "expected_answer": "By grounding model responses in retrieved external documents.",
        "relevant_chunk_id": "5"
    },
    {
        "question": "What is a context window?",
        "expected_answer": "The maximum number of tokens a model can process in a single forward pass.",
        "relevant_chunk_id": "6"
    },
]
```

## Metric 1: Retrieval Hit Rate

Did the correct chunk appear in the top-K results? This measures retrieval quality independently of generation.

```python
# eval_retrieval.py
from rag_pipeline import retrieve

def evaluate_retrieval(eval_set: list[dict], top_k: int = 3) -> dict:
    hits = 0
    for item in eval_set:
        results = retrieve(item["question"], top_k=top_k)
        retrieved_ids = {r.get("id", "") for r in results}

        # ChromaDB doesn't return IDs by default — we check text match as proxy
        retrieved_texts = " ".join(r["text"] for r in results).lower()
        expected = item["expected_answer"].lower()

        # Substring match as a proxy for chunk retrieval
        hit = any(word in retrieved_texts for word in expected.split()[:5])
        hits += int(hit)

    hit_rate = hits / len(eval_set)
    print(f"Retrieval Hit Rate @ {top_k}: {hit_rate:.2%} ({hits}/{len(eval_set)})")
    return {"hit_rate": hit_rate, "hits": hits, "total": len(eval_set)}

evaluate_retrieval(eval_set, top_k=3)
```

```result
Retrieval Hit Rate @ 3: 100.00% (4/4)
```

A hit rate below 70% usually means your chunking or embedding model needs adjustment.

## Metric 2: Answer Faithfulness

Did the generated answer come from the retrieved context — or did the model hallucinate? We use an LLM judge for this.

```python
# eval_faithfulness.py
import anthropic
from rag_pipeline import rag

client = anthropic.Anthropic()

def judge_faithfulness(question: str, context: str, answer: str) -> dict:
    prompt = f"""You are evaluating whether a generated answer is faithful to the provided context.

Context:
{context}

Question: {question}
Answer: {answer}

Is the answer faithful to the context? Answer with:
- score: 1 if fully faithful, 0.5 if partially, 0 if not faithful or hallucinated
- reason: one sentence explaining the score

Respond in JSON only: {{"score": float, "reason": "string"}}"""

    r = client.messages.create(
        model="claude-opus-4-7", max_tokens=150,
        messages=[{"role": "user", "content": prompt}]
    )
    import json
    return json.loads(r.content[0].text)

def evaluate_faithfulness(eval_set: list[dict]) -> dict:
    from rag_pipeline import retrieve, generate_answer
    scores = []

    for item in eval_set:
        chunks = retrieve(item["question"], top_k=3)
        context = "\n".join(c["text"] for c in chunks)
        answer  = generate_answer(item["question"], chunks)

        result = judge_faithfulness(item["question"], context, answer)
        scores.append(result["score"])
        print(f"Q: {item['question'][:60]}")
        print(f"   Score: {result['score']} | {result['reason']}")

    avg = sum(scores) / len(scores)
    print(f"\nMean Faithfulness: {avg:.2f}")
    return {"mean_faithfulness": avg}

evaluate_faithfulness(eval_set)
```

```result
Q: What is BERT's pre-training objective?
   Score: 1 | Answer directly restates the context without adding external information.
Q: How do GPT models generate text?
   Score: 1 | Answer faithfully reflects the context's description of autoregressive generation.
Q: Why does RAG reduce hallucination?
   Score: 1 | Answer accurately paraphrases the retrieved chunk.
Q: What is a context window?
   Score: 1 | Answer matches the context definition exactly.

Mean Faithfulness: 1.00
```

## Metric 3: Answer Correctness

Did the answer match the expected answer? Compare using an LLM.

```python
# eval_correctness.py
def judge_correctness(question: str, expected: str, actual: str) -> dict:
    prompt = f"""Compare two answers to the same question.

Question: {question}
Expected answer: {expected}
Actual answer: {actual}

Is the actual answer semantically correct compared to the expected answer?
Score: 1 = correct, 0.5 = partially correct, 0 = incorrect.

Respond in JSON only: {{"score": float, "reason": "string"}}"""

    r = client.messages.create(
        model="claude-opus-4-7", max_tokens=150,
        messages=[{"role": "user", "content": prompt}]
    )
    import json
    return json.loads(r.content[0].text)

def evaluate_correctness(eval_set: list[dict]) -> dict:
    from rag_pipeline import rag
    scores = []

    for item in eval_set:
        actual = rag(item["question"])
        result = judge_correctness(item["question"], item["expected_answer"], actual)
        scores.append(result["score"])

    avg = sum(scores) / len(scores)
    print(f"Mean Correctness: {avg:.2f}")
    return {"mean_correctness": avg}
```

## The RAG Evaluation Summary

| Metric | What It Measures | Target |
|--------|-----------------|--------|
| Hit Rate @ K | Does retrieval find the right chunk? | > 80% |
| Faithfulness | Does the answer stay grounded in context? | > 0.90 |
| Correctness | Is the answer right? | > 0.80 |

Run these metrics whenever you change chunking strategy, embedding model, or prompt. Treat them like unit tests for your RAG pipeline.

## Wrapping Up Day 6

You now have a systematic way to measure RAG quality — not just "does it feel right" but quantitative metrics on retrieval and generation. Tomorrow on **Day 7: Capstone**, you wire everything into a complete "Chat with a PDF" application — the most common RAG project type in industry.
