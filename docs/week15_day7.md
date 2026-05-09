# Day 7: Capstone — Multi-Turn CLI Chatbot with Memory

Welcome to Day 7. This week you learned: what LLMs are, how to call their APIs, prompt engineering techniques, tokens and costs, system prompts, and prompt templates. Today you combine all of it into a production-quality command-line chatbot.

## What We Are Building

A CLI chatbot called **DevBot** — a coding assistant that:
- Maintains multi-turn conversation history
- Enforces a context window budget (auto-truncates old messages)
- Tracks and displays cost per message and session total
- Supports slash commands (`/clear`, `/cost`, `/export`, `/quit`)
- Uses a well-crafted system prompt persona

## The Full Implementation

```python
# devbot.py
"""
DevBot — A production-quality CLI coding assistant.
Usage: python devbot.py
"""
import os
import json
from datetime import datetime
from dotenv import load_dotenv
import anthropic

load_dotenv()

# ─── Configuration ──────────────────────────────────────────────
MODEL          = "claude-opus-4-7"
MAX_TOKENS_OUT = 1024
CONTEXT_BUDGET = 4000   # Truncate history if input exceeds this many tokens

SYSTEM_PROMPT = """You are DevBot, an expert coding assistant for software engineers.

Expertise: Python, JavaScript/TypeScript, SQL, system design, debugging, code review.

Style:
- Be concise and direct. Experienced engineers prefer brevity.
- Always provide runnable code examples in fenced code blocks.
- When debugging, identify the root cause before suggesting fixes.
- If you are unsure, say so — never fabricate function names or behavior.

Format:
- Use markdown headers to organize long responses.
- For code, always include the language identifier (```python, ```bash, etc.)
"""

# Anthropic pricing per million tokens (update as needed)
PRICE_INPUT_PER_M  = 15.00
PRICE_OUTPUT_PER_M = 75.00

# ─── State ──────────────────────────────────────────────────────
client          = anthropic.Anthropic()
messages        = []
session_cost    = 0.0
session_tokens  = {"input": 0, "output": 0}


# ─── Helpers ────────────────────────────────────────────────────
def estimate_cost(input_tokens: int, output_tokens: int) -> float:
    return (input_tokens * PRICE_INPUT_PER_M + output_tokens * PRICE_OUTPUT_PER_M) / 1_000_000

def count_current_tokens() -> int:
    if not messages:
        return 0
    result = client.messages.count_tokens(model=MODEL, messages=messages)
    return result.input_tokens

def truncate_history():
    """Remove oldest user+assistant pairs until we are under budget."""
    while count_current_tokens() > CONTEXT_BUDGET and len(messages) > 2:
        # Remove the oldest user+assistant pair (indices 0 and 1)
        messages.pop(0)
        if messages:
            messages.pop(0)

def export_conversation():
    filename = f"chat_{datetime.now().strftime('%Y%m%d_%H%M%S')}.json"
    with open(filename, "w") as f:
        json.dump({
            "model": MODEL,
            "system": SYSTEM_PROMPT,
            "messages": messages,
            "session_cost_usd": round(session_cost, 6),
            "session_tokens": session_tokens
        }, f, indent=2)
    print(f"  Conversation saved to {filename}")


# ─── Command Handlers ────────────────────────────────────────────
def handle_command(cmd: str) -> bool:
    """Returns True if the input was a command (skip LLM call)."""
    cmd = cmd.strip().lower()
    if cmd == "/quit" or cmd == "/exit":
        print(f"\n  Session cost: ${session_cost:.4f} ({session_tokens['input']} in + {session_tokens['output']} out tokens)")
        print("  Goodbye!")
        raise SystemExit(0)
    elif cmd == "/clear":
        messages.clear()
        print("  Conversation history cleared.")
        return True
    elif cmd == "/cost":
        print(f"  Session cost: ${session_cost:.4f} ({session_tokens['input']} in + {session_tokens['output']} out tokens)")
        return True
    elif cmd == "/export":
        export_conversation()
        return True
    elif cmd == "/help":
        print("  Commands: /clear  /cost  /export  /quit  /help")
        return True
    return False


# ─── Main Chat Loop ──────────────────────────────────────────────
def chat(user_input: str) -> str:
    global session_cost

    messages.append({"role": "user", "content": user_input})
    truncate_history()

    response = client.messages.create(
        model=MODEL,
        max_tokens=MAX_TOKENS_OUT,
        system=SYSTEM_PROMPT,
        messages=messages
    )

    reply = response.content[0].text
    messages.append({"role": "assistant", "content": reply})

    # Track usage
    in_tok  = response.usage.input_tokens
    out_tok = response.usage.output_tokens
    cost    = estimate_cost(in_tok, out_tok)

    session_tokens["input"]  += in_tok
    session_tokens["output"] += out_tok
    session_cost             += cost

    return reply, in_tok, out_tok, cost


def main():
    print("╔══════════════════════════════════════╗")
    print("║  DevBot — AI Coding Assistant         ║")
    print("║  Type /help for commands, /quit to exit ║")
    print("╚══════════════════════════════════════╝\n")

    while True:
        try:
            user_input = input("You: ").strip()
        except (KeyboardInterrupt, EOFError):
            print("\n  Use /quit to exit.")
            continue

        if not user_input:
            continue

        if handle_command(user_input):
            continue

        try:
            reply, in_tok, out_tok, cost = chat(user_input)
            print(f"\nDevBot: {reply}")
            print(f"  [tokens: {in_tok}↑ {out_tok}↓ | cost: ${cost:.4f}]\n")
        except anthropic.APIError as e:
            print(f"  API Error: {e}")


if __name__ == "__main__":
    main()
```

## Running It

```bash
python devbot.py
```

```result
╔══════════════════════════════════════╗
║  DevBot — AI Coding Assistant         ║
║  Type /help for commands, /quit to exit ║
╚══════════════════════════════════════╝

You: How do I reverse a string in Python?

DevBot: Three common approaches:

```python
s = "hello"

# 1. Slice (most Pythonic)
reversed_s = s[::-1]

# 2. reversed() + join
reversed_s = "".join(reversed(s))

# 3. Loop (verbose, avoid)
reversed_s = ""
for char in s:
    reversed_s = char + reversed_s
```

Prefer slice for simple cases; `reversed()` if you need an iterator.
  [tokens: 187↑ 89↓ | cost: $0.0094]

You: /cost
  Session cost: $0.0094 (187 in + 89 out tokens)

You: /quit
  Session cost: $0.0094 (187 in + 89 out tokens)
  Goodbye!
```

## What This Capstone Demonstrates

| Feature | Implementation |
|---------|----------------|
| Conversation memory | `messages` list, appended per turn |
| Context budget | `truncate_history()` removes old pairs |
| Cost tracking | `estimate_cost()` per call + session total |
| Slash commands | `handle_command()` before LLM call |
| Error handling | `anthropic.APIError` catch |
| Export | JSON dump with full metadata |

## Wrapping Up Week 15

You started this week knowing what an LLM is in theory. You leave it having built a real, cost-aware, multi-turn CLI assistant. But there is a fundamental limitation: the chatbot only knows what you tell it in the conversation. It cannot access your company's documentation, a PDF you just uploaded, or any information beyond its training cutoff.

Next week in **Week 16: RAG Systems & Vector Databases**, we solve that problem with Retrieval-Augmented Generation — the technique that lets LLMs answer questions about *your* data.
