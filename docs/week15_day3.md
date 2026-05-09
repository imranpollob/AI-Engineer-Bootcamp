# Day 3: Prompt Engineering Fundamentals

Welcome to Day 3. You know how to call an LLM. Now let us talk about how to *talk* to it well. Prompt engineering is the practice of crafting inputs that reliably produce high-quality outputs. It is part science, part communication skill.

## The Recipe Card Analogy

A vague recipe says "bake until done." A good recipe says "bake at 375°F for 25 minutes until a toothpick inserted in the center comes out clean." Both ask for the same thing, but only one reliably produces a good cake. Prompting is the same: specificity and structure produce consistent results.

## Prompting Technique 1: Zero-Shot

Ask the model directly with no examples. Works for straightforward tasks.

```python
# zero_shot.py
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

def ask(prompt: str) -> str:
    r = client.messages.create(
        model="claude-opus-4-7", max_tokens=200,
        messages=[{"role": "user", "content": prompt}]
    )
    return r.content[0].text

# Zero-shot classification
print(ask("Classify the sentiment of this review as positive, neutral, or negative:\n\n'The battery life is decent but the screen is too dim.'"))
```

```result
Neutral
```

## Prompting Technique 2: Few-Shot

Provide 2–5 examples to demonstrate the exact format and style you want. Dramatically improves consistency.

```python
# few_shot.py
prompt = """Classify each review as positive, neutral, or negative.

Review: "Amazing product, works perfectly!" → positive
Review: "Arrived on time, nothing special." → neutral
Review: "Broke after two days, terrible quality." → negative

Review: "Great camera but the software is buggy." → """

print(ask(prompt))
```

```result
neutral
```

Few-shot prompting is especially powerful when you need a specific output format and zero-shot keeps returning the wrong structure.

## Prompting Technique 3: Chain of Thought (CoT)

For reasoning tasks, ask the model to think step by step before giving the answer. This dramatically improves accuracy on math, logic, and multi-step problems.

```python
# chain_of_thought.py

# Without CoT — often wrong on tricky problems
bad_prompt = "If a store sells 3 widgets at $4.50 each and gives a 10% discount on orders over $10, how much does the customer pay?"

# With CoT — reliable
good_prompt = """If a store sells 3 widgets at $4.50 each and gives a 10% discount on orders over $10, how much does the customer pay?

Let's think through this step by step."""

print("Without CoT:")
print(ask(bad_prompt))
print("\nWith CoT:")
print(ask(good_prompt))
```

```result
Without CoT:
$12.15

With CoT:
Step 1: Calculate the total before discount.
3 widgets × $4.50 = $13.50

Step 2: Check if the order qualifies for a discount.
$13.50 > $10 → Yes, the 10% discount applies.

Step 3: Calculate the discount.
10% of $13.50 = $1.35

Step 4: Subtract the discount.
$13.50 - $1.35 = $12.15

The customer pays $12.15.
```

## Prompting Technique 4: Structured Output

For applications, you often need the model's response as structured data (JSON) that code can parse. Specify the exact schema in the prompt.

```python
# structured_output.py
import json

prompt = """Extract the following information from this job posting and return ONLY valid JSON with these exact keys: title, company, location, salary_range, required_skills (list).

Job posting:
"Senior Python Developer at TechCorp in Austin, TX. Salary: $130k-$160k. Must have: Python, FastAPI, Docker, PostgreSQL, 5+ years experience."
"""

response_text = ask(prompt)
data = json.loads(response_text)

print(f"Title:    {data['title']}")
print(f"Company:  {data['company']}")
print(f"Skills:   {', '.join(data['required_skills'])}")
print(f"Salary:   {data['salary_range']}")
```

```result
Title:    Senior Python Developer
Company:  TechCorp
Skills:   Python, FastAPI, Docker, PostgreSQL
Salary:   $130k-$160k
```

For production use, Anthropic's API also supports a `tool_use` feature that guarantees structured JSON output — we will cover that in Week 17 when we build agents.

## Prompting Technique 5: Role Prompting

Tell the model *who it is* to activate relevant knowledge and tone.

```python
# role_prompting.py
prompts = [
    "Explain recursion.",
    "You are a patient teacher explaining to a 10-year-old. Explain recursion.",
    "You are a senior software engineer doing a code review. Explain recursion.",
]

for p in prompts:
    print(f"Prompt: {p[:60]}...")
    print(f"Response: {ask(p)[:200]}\n{'─'*50}")
```

```result
Prompt: Explain recursion....
Response: Recursion is a programming technique where a function calls itself...

Prompt: You are a patient teacher explaining to a 10-year-old. Explain recursion....
Response: Imagine you're looking in a mirror and holding another mirror behind you...

Prompt: You are a senior software engineer doing a code review. Explain recursion....
Response: Recursion is a technique where a function invokes itself with a reduced subproblem...
```

## Prompting Best Practices

| Do | Don't |
|----|-------|
| Be specific about output format | Assume the model knows what you want |
| Use examples for new/unusual tasks | Use vague directives like "be good" |
| Tell the model its role | Leave the system prompt empty |
| Ask for step-by-step reasoning | Expect perfect math without CoT |
| Validate and parse structured output | Trust unvalidated JSON output |

## Wrapping Up Day 3

You now have a practical toolkit of prompting techniques: zero-shot for simple tasks, few-shot for format control, chain-of-thought for reasoning, structured output for data extraction, and role prompting for tone control. Tomorrow on **Day 4: Tokens, Context, and Costs**, we go under the hood to understand how LLMs actually measure and charge for text — knowledge you need to avoid expensive mistakes in production.
