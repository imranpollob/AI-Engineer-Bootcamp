# Day 1: What Are Large Language Models?

Welcome to Week 15. Everything you have learned so far — Python, Math, Classical ML, Deep Learning, Transformers, deployment — has been building to this moment. Large Language Models (LLMs) are the technology reshaping every industry right now, and understanding them is the defining skill of a modern AI engineer.

## From Transformer to GPT

In Week 12 you learned about the Transformer architecture. An LLM is a Transformer that has been trained at a scale that changes everything.

The original "Attention is All You Need" paper (2017) introduced Transformers for translation tasks using millions of examples. Modern LLMs — GPT-4, Claude, Gemini — use the same core architecture but trained on:

- **Trillions of tokens** (tokens ≈ word fragments; a trillion tokens ≈ 4 million books)
- **Hundreds of billions of parameters** (the "weights" adjusted during training)
- **Months of compute** on thousands of specialized chips (GPUs/TPUs)

The result is a model that learns not just translation, but *general reasoning about language* — including code, math, and creative writing.

## The Autocomplete That Learned to Reason

At its core, an LLM does one thing: **predict the next token**. Given the sequence "The cat sat on the", it predicts "mat" is more likely than "elephant." That is it.

But when you train this simple objective on enough data, something remarkable emerges: the model develops internal representations of facts, reasoning chains, coding patterns, and even common sense. It is not explicitly programmed to know Paris is the capital of France — it learns it because that pattern appeared in the training data thousands of times.

```
Input:  "The capital of France is"
Model:  [calculates probability distribution over all 50,000+ tokens]
Output: "Paris"  (highest probability token)
```

## The Major LLMs Today

| Model | Company | Key Strength |
|-------|---------|--------------|
| **GPT-4o** | OpenAI | General-purpose, multimodal (text + images) |
| **Claude 3.5 / 4** | Anthropic | Long context, safety, coding |
| **Gemini 1.5 / 2** | Google DeepMind | Massive context window, multimodal |
| **Llama 3** | Meta | Open-source, self-hostable |
| **Mistral** | Mistral AI | Open-source, efficient |

As an AI engineer, you will primarily interact with these models through their APIs — you do not train them (that costs hundreds of millions of dollars), but you use them as the "brain" in your applications.

## Training vs. Fine-Tuning vs. Using

```
Pre-Training       Fine-Tuning          Using via API
─────────────      ────────────         ─────────────────
Billions of $      Thousands of $       Cents per request
Months             Hours to days        Milliseconds
Google/OpenAI      ML teams             YOU (this week)
Learns language    Learns a task        Solves a problem
```

As a practitioner, you live in the right column. The left column already happened.

## How LLMs Are Deployed

After pre-training, LLMs go through two more steps before they become the assistants you interact with:

**1. Instruction Fine-Tuning (IFT / SFT):** The model is further trained on examples of instruction-response pairs — "Summarize this text" → [good summary]. This teaches it to follow directions instead of just completing text.

**2. Reinforcement Learning from Human Feedback (RLHF):** Human raters compare model outputs and prefer safer, more helpful responses. The model learns to produce those preferred outputs. This is why Claude refuses to help with harmful requests and why ChatGPT apologizes when it is wrong.

## What an LLM Cannot Do

Understanding the limits is as important as understanding the capabilities:

- **No real-time knowledge**: Training data has a cutoff date. The model does not know what happened after that date (without tools).
- **No persistent memory**: Each conversation starts fresh. The model does not remember previous chats (without external storage).
- **Can hallucinate**: LLMs generate plausible text — sometimes that text is confidently wrong. Never trust LLM output for high-stakes decisions without verification.
- **Context window limit**: Models can only process a finite amount of text at once (typically 8K–200K tokens). Documents longer than this must be chunked.

## Wrapping Up Day 1

You now have a solid mental model of what LLMs are, how they came to exist, and where they fit relative to the classical ML you learned earlier. Tomorrow on **Day 2: Using LLM APIs**, we stop theorizing and start coding — you will make your first API call to Claude and OpenAI and see how to structure conversations programmatically.
