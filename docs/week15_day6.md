# Day 6: Advanced Prompting — Templates and ReAct

Welcome to Day 6. You have the fundamentals: zero-shot, few-shot, chain-of-thought, system prompts. Today we go a level deeper with two tools that bridge prompting and software engineering: **prompt templates** (for reusability and maintainability) and **ReAct prompting** (for tasks that require reasoning and action together).

## The Problem with Hardcoded Prompts

If you write prompts directly in code as f-strings, you end up with:

```python
# Hard to maintain, test, or reuse
prompt = f"Summarize this {document_type} for a {audience} in {length} words: {text}"
```

When prompts become complex — dozens of lines with conditional logic — this pattern breaks down fast. Prompt templates solve this.

## Prompt Templates with Jinja2

**Jinja2** is Python's most popular templating engine, used by Flask and Django. It lets you write prompts as text files with variables, conditionals, and loops — cleanly separated from your Python logic.

```bash
pip install jinja2
```

```
prompts/
├── summarizer.j2
├── classifier.j2
└── extractor.j2
```

```jinja2
{# prompts/summarizer.j2 #}
You are a professional summarizer.

{% if audience == "technical" %}
Use technical terminology. Assume the reader has domain expertise.
{% elif audience == "executive" %}
Focus on business impact and key decisions. No jargon. Keep it concise.
{% else %}
Use plain language suitable for a general audience.
{% endif %}

Summarize the following {{ document_type }} in {{ max_words }} words or fewer.
Respond only with the summary — no preamble, no "Here is a summary:".

Document:
{{ document }}
```

```python
# template_demo.py
from jinja2 import Environment, FileSystemLoader
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

env = Environment(loader=FileSystemLoader("prompts"))

def summarize(document: str, audience: str, document_type: str, max_words: int = 100) -> str:
    template = env.get_template("summarizer.j2")
    prompt = template.render(
        document=document,
        audience=audience,
        document_type=document_type,
        max_words=max_words
    )
    r = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=300,
        messages=[{"role": "user", "content": prompt}]
    )
    return r.content[0].text

article = """
Researchers at MIT have demonstrated a new battery chemistry using sodium-ion cells
that achieves 400 Wh/kg energy density at a cost of $40/kWh — significantly below
the current lithium-ion baseline of $100/kWh. The cells use an earth-abundant cathode
material and show 1,000 charge cycle stability at 80% capacity retention.
"""

print("=== Technical Audience ===")
print(summarize(article, "technical", "research paper", max_words=60))
print("\n=== Executive Audience ===")
print(summarize(article, "executive", "research paper", max_words=60))
```

```result
=== Technical Audience ===
MIT demonstrates sodium-ion cells at 400 Wh/kg and $40/kWh, achieving 4× cost reduction vs. lithium-ion. Cathode uses earth-abundant materials; 1,000-cycle stability at 80% retention validated.

=== Executive Audience ===
MIT breakthrough cuts battery costs by 75% vs. current technology using sodium instead of lithium. Batteries last 1,000+ charge cycles — potentially accelerating EV and grid storage adoption.
```

## ReAct Prompting: Reasoning + Acting

**ReAct** (Reasoning + Acting) is a prompting pattern for tasks where the model needs to interleave thinking with tool use. It was introduced in a 2022 research paper and is the foundation of how LLM agents work (Week 17).

The pattern has three repeating steps:

```
Thought: [What am I trying to figure out?]
Action:  [What tool/operation should I use?]
Observation: [What did the tool return?]
... repeat until ...
Answer:  [Final response]
```

```python
# react_demo.py
# We simulate tool outputs here; in Week 17 we wire up real tools.
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

REACT_SYSTEM = """You are a research assistant with access to these tools:
- search(query): Returns web search results
- calculator(expression): Evaluates a math expression
- get_stock_price(ticker): Returns current stock price

To use a tool, write:
Action: tool_name(argument)

I will respond with:
Observation: [result]

Repeat Thought/Action/Observation until you have enough information.
When done, write:
Answer: [your final answer]

Always start with "Thought:" to reason about what to do first."""

query = "What is 15% of Apple's current stock price?"

messages = [{"role": "user", "content": query}]

# Simulate one turn of reasoning
r = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=300,
    system=REACT_SYSTEM,
    messages=messages
)
print(r.content[0].text)

# In a real agent, we'd parse "Action: get_stock_price(AAPL)",
# call the actual function, and feed the result back as:
# {"role": "user", "content": "Observation: $213.49"}
```

```result
Thought: I need Apple's current stock price first, then calculate 15% of it.
Action: get_stock_price(AAPL)
```

The model correctly identifies it needs data before it can compute. In a real agent loop, we intercept `Action:`, call the actual function, and feed back `Observation:` — the model then reasons further. This loop is exactly what Week 17 implements with real tools.

## Building a Reusable Prompt Library

```python
# prompt_library.py
from dataclasses import dataclass
from typing import Optional

@dataclass
class Prompt:
    name: str
    system: str
    user_template: str

    def render(self, **kwargs) -> tuple[str, str]:
        return self.system, self.user_template.format(**kwargs)

# Define reusable prompts
SUMMARIZER = Prompt(
    name="summarizer",
    system="You are a concise summarizer. Return only the summary, no preamble.",
    user_template="Summarize in {words} words:\n\n{text}"
)

CLASSIFIER = Prompt(
    name="sentiment_classifier",
    system="Classify sentiment. Return exactly one word: positive, negative, or neutral.",
    user_template="{text}"
)

def run_prompt(prompt: Prompt, model: str = "claude-opus-4-7", **kwargs) -> str:
    system, user = prompt.render(**kwargs)
    r = client.messages.create(
        model=model, max_tokens=300, system=system,
        messages=[{"role": "user", "content": user}]
    )
    return r.content[0].text

# Use the library
text = "The new update broke everything I loved about this app."
print(run_prompt(SUMMARIZER, text=text, words=10))
print(run_prompt(CLASSIFIER, text=text))
```

```result
Update ruined previously loved app features.
negative
```

## Wrapping Up Day 6

Prompt templates and ReAct are the building blocks of production-grade AI applications. Templates keep your prompts maintainable as they grow; ReAct patterns enable the model to reason and act across multiple steps. Tomorrow on **Day 7: Capstone**, you combine everything from this week into a real multi-turn CLI chatbot with conversation memory, context limits, and cost tracking.
