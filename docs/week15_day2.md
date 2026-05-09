# Day 2: Using LLM APIs

Welcome to Day 2. Yesterday you built a mental model of what LLMs are. Today you use them — hands on, in code. The Anthropic and OpenAI APIs are the two most common in industry. We will start with Anthropic's SDK (which powers Claude) since it has excellent design, then show the OpenAI equivalent.

## The API Mental Model

An LLM API is a web service that accepts a **conversation** (a list of messages) and returns the model's **response**. You pay per token processed — both the tokens you send and the tokens the model generates.

The conversation structure is always the same:

```
system   → Sets the model's persona and rules (optional but important)
user     → Your message to the model
assistant→ The model's reply
user     → Your follow-up
assistant→ The model's follow-up reply
...
```

## Install the SDKs

```bash
pip install anthropic openai python-dotenv
```

Store your keys in a `.env` file (never commit this to git):

```bash
# .env
ANTHROPIC_API_KEY=sk-ant-...
OPENAI_API_KEY=sk-...
```

Add `.env` to your `.gitignore`:

```bash
echo ".env" >> .gitignore
```

## Your First Anthropic API Call

```python
# anthropic_hello.py
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=256,
    messages=[
        {"role": "user", "content": "What is the difference between supervised and unsupervised learning? One sentence each."}
    ]
)

print(response.content[0].text)
```

```result
Supervised learning trains models on labeled data where the correct output is known, learning to map inputs to outputs. Unsupervised learning finds patterns and structure in unlabeled data without any correct answers provided.
```

## Understanding the Response Object

```python
# response_anatomy.py
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=100,
    messages=[{"role": "user", "content": "Say hello!"}]
)

print(f"Model:        {response.model}")
print(f"Stop reason:  {response.stop_reason}")     # end_turn, max_tokens, stop_sequence
print(f"Input tokens: {response.usage.input_tokens}")
print(f"Output tokens:{response.usage.output_tokens}")
print(f"Text content: {response.content[0].text}")
```

```result
Model:         claude-opus-4-7
Stop reason:   end_turn
Input tokens:  11
Output tokens: 10
Text content:  Hello! How can I help you today?
```

`stop_reason = "max_tokens"` means the model was cut off before finishing. Increase `max_tokens` if this happens.

## Adding a System Prompt

The system prompt sets the model's personality and operating rules. It is the most powerful tool for shaping behavior consistently.

```python
# system_prompt_demo.py
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

response = client.messages.create(
    model="claude-opus-4-7",
    max_tokens=200,
    system="You are a senior data scientist. Respond only in bullet points. Be concise.",
    messages=[
        {"role": "user", "content": "What should I do before training a machine learning model?"}
    ]
)

print(response.content[0].text)
```

```result
• Explore and understand your data (EDA)
• Handle missing values and outliers
• Encode categorical variables
• Split into train/validation/test sets
• Scale/normalize numerical features
• Check for class imbalance
• Establish a baseline model
```

## Multi-Turn Conversations

To have a back-and-forth conversation, you must manually maintain the message history. The API is stateless — it does not remember previous calls.

```python
# multi_turn.py
import anthropic
from dotenv import load_dotenv

load_dotenv()
client = anthropic.Anthropic()

# Maintain the conversation history yourself
messages = []

def chat(user_message: str) -> str:
    messages.append({"role": "user", "content": user_message})

    response = client.messages.create(
        model="claude-opus-4-7",
        max_tokens=300,
        system="You are a helpful coding tutor.",
        messages=messages
    )

    assistant_reply = response.content[0].text
    messages.append({"role": "assistant", "content": assistant_reply})
    return assistant_reply

# Simulate a conversation
print(chat("What is a list comprehension in Python?"))
print("---")
print(chat("Can you show me an example using that concept?"))
print("---")
print(chat("How would I filter even numbers from 1 to 20 with it?"))
```

```result
A list comprehension is a concise way to create lists in Python using a single line of code...
---
Sure! Here is a basic example: squares = [x**2 for x in range(5)] # [0, 1, 4, 9, 16]
---
[x for x in range(1, 21) if x % 2 == 0]  # [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
```

## The OpenAI Equivalent

The OpenAI API follows the same pattern:

```python
# openai_hello.py
from openai import OpenAI
from dotenv import load_dotenv

load_dotenv()
client = OpenAI()

response = client.chat.completions.create(
    model="gpt-4o",
    max_tokens=256,
    messages=[
        {"role": "system", "content": "You are a helpful assistant."},
        {"role": "user",   "content": "Explain gradient descent in one sentence."}
    ]
)

print(response.choices[0].message.content)
print(f"Tokens used: {response.usage.total_tokens}")
```

```result
Gradient descent is an optimization algorithm that iteratively adjusts model parameters in the direction that minimizes the loss function by following the negative gradient.
Tokens used: 52
```

The APIs are nearly identical in structure — swapping between providers is straightforward.

## Wrapping Up Day 2

You can now make real API calls to state-of-the-art LLMs, manage multi-turn conversations, and understand the response structure. Tomorrow on **Day 3: Prompt Engineering**, we go deeper into *how* you phrase your requests — because how you write prompts has an enormous impact on the quality of results you get.
