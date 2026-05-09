# Day 4: Multi-Agent Patterns

Welcome to Day 4. Yesterday's research agent handles a single question well. But some tasks are inherently parallel or too complex for one agent to manage alone: research 5 topics simultaneously, have one agent critique another's output, or break a large task into specialized subtasks.

**Multi-agent systems** solve this. Multiple agents collaborate — each specialized, each operating in parallel or in sequence — to accomplish what no single agent could do efficiently alone.

## The Orchestrator-Worker Pattern

The most common multi-agent architecture uses an **orchestrator** (coordinator) and multiple **workers** (specialists):

```
User Request
     │
     ▼
Orchestrator Agent
     ├── assigns Task A → Worker Agent 1 (Research)
     ├── assigns Task B → Worker Agent 2 (Data Analysis)
     └── assigns Task C → Worker Agent 3 (Writing)
          │
          ▼ (all run in parallel)
     Orchestrator collects results and synthesizes final answer
```

The orchestrator decides task decomposition; workers focus on their domain.

## Implementation: Parallel Research

```python
# multi_agent.py
"""
Multi-agent system: an orchestrator decomposes a research question
into sub-questions, workers research them in parallel, and the
orchestrator synthesizes the final answer.
"""
import json
import asyncio
import anthropic
from dotenv import load_dotenv
from research_tools import web_search, summarize_text

load_dotenv()
client = anthropic.Anthropic()


# ─── Worker Agent ─────────────────────────────────────────────────
def research_worker(sub_question: str, worker_id: int) -> dict:
    """A specialized worker that researches a single sub-question."""
    print(f"  [Worker {worker_id}] Starting: {sub_question[:60]}...")

    results = web_search(sub_question, max_results=3)
    snippets = "\n\n".join(
        f"Source: {r['url']}\n{r['snippet']}"
        for r in results if r.get("snippet")
    )

    if not snippets:
        return {"sub_question": sub_question, "findings": "No results found.", "sources": []}

    summary = summarize_text(snippets, focus=sub_question)
    sources = [r["url"] for r in results]

    print(f"  [Worker {worker_id}] Done.")
    return {
        "sub_question": sub_question,
        "findings":     summary,
        "sources":      sources
    }


# ─── Orchestrator ─────────────────────────────────────────────────
def decompose_question(main_question: str) -> list[str]:
    """Ask Claude to break a complex question into sub-questions."""
    r = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=300,
        messages=[{
            "role": "user",
            "content": f"""Break this research question into 3-4 specific sub-questions that can be researched independently.

Main question: {main_question}

Return ONLY a JSON array of strings, e.g.:
["sub-question 1", "sub-question 2", "sub-question 3"]"""
        }]
    )
    return json.loads(r.content[0].text)


def synthesize_findings(main_question: str, worker_results: list[dict]) -> str:
    """Ask Claude to synthesize all worker findings into a final answer."""
    findings_text = "\n\n".join(
        f"Sub-question: {r['sub_question']}\nFindings: {r['findings']}\nSources: {', '.join(r['sources'][:2])}"
        for r in worker_results
    )

    r = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=600,
        system="You are a research synthesizer. Combine findings from multiple research workers into a coherent, well-structured answer.",
        messages=[{
            "role": "user",
            "content": f"""Main question: {main_question}

Research findings from workers:
{findings_text}

Synthesize these into a comprehensive answer. Mention sources where relevant."""
        }]
    )
    return r.content[0].text


# ─── Multi-Agent Coordinator ──────────────────────────────────────
def multi_agent_research(main_question: str) -> str:
    print(f"[Orchestrator] Decomposing: {main_question}\n")

    sub_questions = decompose_question(main_question)
    print(f"[Orchestrator] Sub-questions:")
    for i, q in enumerate(sub_questions, 1):
        print(f"  {i}. {q}")
    print()

    # Workers run sequentially here; use threading/asyncio in production for parallel execution
    worker_results = []
    for i, q in enumerate(sub_questions, 1):
        result = research_worker(q, worker_id=i)
        worker_results.append(result)

    print("\n[Orchestrator] Synthesizing findings...")
    return synthesize_findings(main_question, worker_results)


if __name__ == "__main__":
    question = "How does fine-tuning a large language model differ from RAG, and when should you use each approach?"

    print(f"Question: {question}")
    print("=" * 70)
    answer = multi_agent_research(question)
    print(f"\nFinal Answer:\n{answer}")
```

```result
Question: How does fine-tuning a large language model differ from RAG, and when should you use each approach?
══════════════════════════════════════════════════════════════════════
[Orchestrator] Decomposing: How does fine-tuning...

[Orchestrator] Sub-questions:
  1. What is fine-tuning and how does it work technically?
  2. What is RAG and how does it differ architecturally from fine-tuning?
  3. What are the costs and infrastructure requirements of each approach?
  4. What are the best use cases and limitations for fine-tuning vs RAG?

  [Worker 1] Starting: What is fine-tuning and how does it work technically?...
  [Worker 1] Done.
  [Worker 2] Starting: What is RAG and how does it differ architecturally...
  [Worker 2] Done.
  [Worker 3] Starting: What are the costs and infrastructure requirements...
  [Worker 3] Done.
  [Worker 4] Starting: What are the best use cases and limitations...
  [Worker 4] Done.

[Orchestrator] Synthesizing findings...

Final Answer:
Fine-tuning and RAG are complementary approaches that solve different problems...
[Full synthesized answer follows]
```

## The Critic-Reviser Pattern

Another powerful pattern: one agent writes, another critiques, and the first revises.

```python
# critic_reviser.py
def critic_reviser(task: str, initial_draft: str) -> str:
    """
    Two-agent loop: a critic identifies issues, a reviser fixes them.
    """
    # Step 1: Critic evaluates the draft
    critic_response = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=400,
        system="You are a strict editor. Identify specific issues with clarity, accuracy, and completeness.",
        messages=[{
            "role": "user",
            "content": f"Task: {task}\n\nDraft:\n{initial_draft}\n\nList 3 specific improvements needed."
        }]
    )
    critique = critic_response.content[0].text
    print(f"[Critic]: {critique[:200]}...")

    # Step 2: Reviser improves based on critique
    reviser_response = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=600,
        system="You are a skilled writer. Revise the draft based on the provided critique.",
        messages=[{
            "role": "user",
            "content": f"Task: {task}\n\nOriginal draft:\n{initial_draft}\n\nCritique:\n{critique}\n\nWrite an improved version."
        }]
    )
    return reviser_response.content[0].text
```

## When to Use Multi-Agent Systems

| Pattern | Use When |
|---------|---------|
| Orchestrator-Worker | Tasks can be parallelized into independent sub-tasks |
| Critic-Reviser | Output quality needs iterative improvement |
| Pipeline (A→B→C) | Tasks have strict sequential dependencies |
| Debate (A vs B) | Need adversarial perspectives before deciding |

## Wrapping Up Day 4

Multi-agent patterns let you scale beyond what a single prompt can handle — by decomposing, parallelizing, and iterating. Tomorrow on **Day 5: LangChain and LlamaIndex Overview**, we look at popular frameworks that abstract these patterns so you don't always have to build from scratch.
