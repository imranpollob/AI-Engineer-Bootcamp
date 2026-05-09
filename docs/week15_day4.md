# Day 4: Tokens, Context Windows, and Costs

Welcome to Day 4. You have been making API calls and getting results. But every token costs money and every model has a finite attention span. Today we demystify tokens and context windows — the two constraints that govern every LLM application you build.

## What Is a Token?

LLMs do not process words — they process **tokens**, which are chunks of text produced by a tokenizer. The tokenizer splits text into pieces that the model finds natural based on frequency in training data.

Rough rules of thumb for English:
- 1 word ≈ 1.3 tokens
- 1000 words ≈ 1300 tokens
- 1 page of text ≈ 700 tokens
- 100 tokens ≈ 75 words

```python
# token_counting.py
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

texts = [
    "Hello",                  # Simple word
    "Hippopotamus",           # Longer word
    "OpenAI",                 # Proper noun
    "def calculate_sum(a, b): return a + b",  # Code
    "The quick brown fox jumps over the lazy dog. " * 10,  # Repeated text
]

for text in texts:
    # Count tokens without sending to the model
    response = client.messages.count_tokens(
        model="claude-opus-4-7",
        messages=[{"role": "user", "content": text}]
    )
    print(f"{len(text):5d} chars → {response.input_tokens:4d} tokens | '{text[:40]}...'")
```

```result
    5 chars →    1 tokens | 'Hello...'
   14 chars →    3 tokens | 'Hippopotamus...'
    6 chars →    2 tokens | 'OpenAI...'
   36 chars →   11 tokens | 'def calculate_sum(a, b): return a + b...'
  470 chars →   90 tokens | 'The quick brown fox jumps over the lazy dog.  The ...'
```

Code and uncommon words often tokenize into more pieces than simple English words.

## The Context Window

The **context window** is the maximum number of tokens a model can process in a single API call — both input and output combined.

| Model | Context Window |
|-------|----------------|
| GPT-3.5-turbo | 16K tokens |
| GPT-4o | 128K tokens |
| Claude 3.5 Sonnet | 200K tokens |
| Claude 3 Opus | 200K tokens |
| Gemini 1.5 Pro | 1M tokens |

200K tokens ≈ 150,000 words ≈ 500 pages of text. Sounds huge — but large codebases, full books, or long document collections can exceed even these limits.

```python
# context_window_math.py
# How much can fit in Claude's 200K context window?

context_window = 200_000  # tokens

examples = {
    "Short email":           50,
    "News article":          700,
    "Short story":         5_000,
    "Full novel (Gatsby)": 47_000,
    "Python codebase (medium)": 30_000,
}

print(f"{'Content':<35} {'Tokens':>8} {'Fits?':>8} {'How Many?':>10}")
print("─" * 65)
for name, tokens in examples.items():
    fits = "Yes" if tokens <= context_window else "No"
    count = context_window // tokens
    print(f"{name:<35} {tokens:>8,} {fits:>8} {count:>10}x")
```

```result
Content                              Tokens    Fits?  How Many?
─────────────────────────────────────────────────────────────────
Short email                              50      Yes      4000x
News article                            700      Yes       285x
Short story                           5,000      Yes        40x
Full novel (Gatsby)                  47,000      Yes         4x
Python codebase (medium)             30,000      Yes         6x
```

## Understanding Costs

LLM APIs charge separately for input tokens (your prompt) and output tokens (the model's reply). Output tokens typically cost 3–5x more than input tokens because generation requires more computation.

```python
# cost_calculator.py

# Approximate prices per million tokens (check provider sites for current rates)
pricing = {
    "claude-opus-4-7":    {"input": 15.00, "output": 75.00},
    "claude-sonnet-4-6":  {"input":  3.00, "output": 15.00},
    "gpt-4o":             {"input":  5.00, "output": 15.00},
    "gpt-4o-mini":        {"input":  0.15, "output":  0.60},
}

def estimate_cost(model: str, input_tokens: int, output_tokens: int) -> float:
    p = pricing[model]
    return (input_tokens * p["input"] + output_tokens * p["output"]) / 1_000_000

# Scenario: Customer support bot processing 10,000 queries/day
# Each query: ~500 input tokens + ~200 output tokens
daily_queries = 10_000
input_tokens_per_query = 500
output_tokens_per_query = 200

print(f"{'Model':<25} {'Cost/Query':>12} {'Daily Cost':>12} {'Monthly Cost':>14}")
print("─" * 65)
for model in pricing:
    cost_per = estimate_cost(model, input_tokens_per_query, output_tokens_per_query)
    daily    = cost_per * daily_queries
    monthly  = daily * 30
    print(f"{model:<25} ${cost_per:>10.4f} ${daily:>11.2f} ${monthly:>13.2f}")
```

```result
Model                     Cost/Query    Daily Cost   Monthly Cost
─────────────────────────────────────────────────────────────────
claude-opus-4-7           $    0.0225 $     225.00 $      6750.00
claude-sonnet-4-6         $    0.0045 $      45.00 $      1350.00
gpt-4o                    $    0.0055 $      55.00 $      1650.00
gpt-4o-mini               $    0.0002 $       1.95 $        58.50
```

This is why model selection is a business decision, not just a technical one. Use the most powerful model for tasks that require it and the smallest model that works for the rest.

## Practical Cost Control

```python
# cost_control.py
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

def chat_with_budget(messages: list, max_input_tokens: int = 1000) -> str:
    # Check token count before sending
    count = client.messages.count_tokens(
        model="claude-opus-4-7",
        messages=messages
    )

    if count.input_tokens > max_input_tokens:
        raise ValueError(
            f"Prompt too large: {count.input_tokens} tokens > {max_input_tokens} limit. "
            "Truncate your input or increase the budget."
        )

    response = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=500,
        messages=messages
    )
    return response.content[0].text

try:
    result = chat_with_budget(
        messages=[{"role": "user", "content": "What is 2 + 2?"}],
        max_input_tokens=1000
    )
    print(result)
except ValueError as e:
    print(f"Budget exceeded: {e}")
```

```result
2 + 2 equals 4.
```

## Wrapping Up Day 4

You now understand the economics of LLM APIs: tokens are the unit of measurement, context windows are the limit, and cost is a function of both. Tomorrow on **Day 5: System Prompts and Personas**, we go deeper into the single most powerful tool for controlling model behavior consistently across an application.
