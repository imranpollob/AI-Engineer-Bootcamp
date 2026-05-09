# Day 1: What Are AI Agents?

Welcome to Week 17, the final week of the AI Engineer Bootcamp. You have covered an extraordinary amount of ground: Python fundamentals, data science, classical ML, deep learning, transformers, deployment, and LLM applications. This week we reach the frontier: **AI agents** — systems that can perceive their environment, reason about what to do, and take actions autonomously.

## The Difference Between a Chatbot and an Agent

A chatbot is reactive and stateless: you ask a question, it answers. That is it.

An agent is proactive and cyclical: it receives a goal, plans how to achieve it, uses tools to gather information or take actions, observes the results, and loops until the goal is met. The agent *does things*, not just *says things*.

```
Chatbot:    User: "What's the weather?" → LLM: "I don't know the current weather."

Agent:      User: "What's the weather?"
            → LLM thinks: "I need to call a weather API."
            → Agent calls weather API → gets result
            → LLM synthesizes: "It's 72°F and sunny in your location."
```

## The Observe → Think → Act Loop

Every agent — from a simple script to AutoGPT — follows the same basic loop:

```
┌─────────────────────────────────┐
│            GOAL                 │
└─────────────────┬───────────────┘
                  │
                  ▼
          ┌───────────────┐
          │    OBSERVE    │  ← What do I know so far?
          └───────┬───────┘
                  │
                  ▼
          ┌───────────────┐
          │     THINK     │  ← What should I do next?
          └───────┬───────┘
                  │
                  ▼
          ┌───────────────┐
          │     ACT       │  ← Execute a tool or action
          └───────┬───────┘
                  │
            ┌─────┘
            ▼
       Goal achieved? → YES → Return answer
            │
            NO → back to OBSERVE
```

The LLM is the "THINK" step. It decides which action to take based on its observations. The surrounding code handles "ACT" (actually calling tools) and feeds results back as "OBSERVE."

## The ReAct Pattern in Practice

The **ReAct** (Reasoning + Acting) pattern from Day 6 of Week 15 is the prompt-level implementation of this loop. The model explicitly writes out its reasoning before acting:

```
Thought: I need to find the current price of NVIDIA stock.
Action: get_stock_price("NVDA")
Observation: $875.42

Thought: I have the price. Now I need to calculate 15% of it.
Action: calculator("875.42 * 0.15")
Observation: 131.313

Thought: I have the answer.
Answer: 15% of NVIDIA's current stock price ($875.42) is $131.31.
```

This explicitness makes agents more reliable and debuggable — you can see exactly what the model was thinking at each step.

## What Can Agents Do?

Agents can use any tool you give them access to:

| Tool Type | Examples |
|-----------|---------|
| **Information** | Web search, Wikipedia lookup, database query |
| **Computation** | Calculator, code execution, unit conversion |
| **Communication** | Send email, post Slack message, file GitHub issue |
| **Storage** | Read/write files, query a vector database |
| **External APIs** | Weather, stocks, maps, calendar |
| **Code** | Write and run Python/shell code |

An agent's power is directly proportional to the quality and breadth of its tools.

## When to Use Agents (and When Not To)

**Use agents when:**
- The task requires multiple steps that cannot be pre-planned
- The task requires gathering information from external sources
- The task requires decision-making based on intermediate results
- The exact steps depend on data you don't have upfront

**Don't use agents when:**
- A single prompt handles the entire task
- You need guaranteed deterministic behavior
- Latency is critical (agent loops add seconds per step)
- The task scope is bounded and well-defined

Agents trade reliability and speed for capability and flexibility. Use them only when the capability is worth the cost.

## This Week's Project

By the end of Week 17, you will have built a **Research Agent** that:
1. Accepts a research question from the user
2. Searches the web for relevant information (Day 3)
3. Reads and summarizes web pages (Day 4)
4. Uses multiple tools in sequence (Day 3)
5. Handles multi-agent coordination (Day 4)
6. Runs reliably in production with rate limiting and error handling (Day 6)

## Wrapping Up Day 1

Agents are the most powerful and most complex pattern in modern AI engineering. The core idea is simple — observe, think, act — but the implementation details matter enormously. Tomorrow on **Day 2: Tool Use with LLM APIs**, we write the code that turns "the model said to call this function" into an actual function call, which is the mechanical heart of every agent.
