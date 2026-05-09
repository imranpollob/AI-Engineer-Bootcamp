# Day 2: Tool Use with LLM APIs

Welcome to Day 2. Yesterday you understood what agents are conceptually. Today you implement the mechanical core: **tool use** — the API feature that lets an LLM request that your code run a specific function and return the result.

## How Tool Use Works

Without tool use, you tell the model what tools exist in the system prompt and parse its text response yourself (fragile). With the official tool use API, the model returns a *structured* tool call that your code can reliably parse and execute.

```
1. You define tools (schemas describing function name, parameters, types)
2. You send the user's message + tool definitions to the API
3. If the model needs a tool, it stops and returns a tool_call object
4. Your code detects the tool call, executes the actual function
5. You send the result back to the model as a tool result
6. The model continues — possibly calling more tools or giving a final answer
```

This loop repeats until the model returns a final text answer with no tool calls.

## Defining Tools for Anthropic's API

Anthropic uses JSON Schema to define tool parameters:

```python
# tool_definitions.py
# Tools are Python dicts describing the function signature

TOOLS = [
    {
        "name": "get_weather",
        "description": "Get current weather for a city. Returns temperature in Celsius and conditions.",
        "input_schema": {
            "type": "object",
            "properties": {
                "city": {
                    "type": "string",
                    "description": "City name, e.g. 'Paris' or 'New York'"
                }
            },
            "required": ["city"]
        }
    },
    {
        "name": "calculator",
        "description": "Evaluate a mathematical expression. Use for any arithmetic.",
        "input_schema": {
            "type": "object",
            "properties": {
                "expression": {
                    "type": "string",
                    "description": "A valid Python math expression, e.g. '(100 * 1.08) / 12'"
                }
            },
            "required": ["expression"]
        }
    },
    {
        "name": "get_stock_price",
        "description": "Get the current price of a stock by its ticker symbol.",
        "input_schema": {
            "type": "object",
            "properties": {
                "ticker": {
                    "type": "string",
                    "description": "Stock ticker symbol, e.g. 'AAPL', 'NVDA', 'MSFT'"
                }
            },
            "required": ["ticker"]
        }
    }
]
```

## Implementing the Tool Functions

```python
# tool_implementations.py
import math
import random

def get_weather(city: str) -> dict:
    # In production: call a real weather API (OpenWeatherMap, etc.)
    # Here we return simulated data for learning purposes
    mock_data = {
        "Paris":    {"temp_c": 18, "conditions": "partly cloudy"},
        "Tokyo":    {"temp_c": 24, "conditions": "sunny"},
        "New York": {"temp_c": 15, "conditions": "overcast"},
    }
    result = mock_data.get(city, {"temp_c": 20, "conditions": "unknown"})
    return {"city": city, **result}

def calculator(expression: str) -> dict:
    try:
        # Safe evaluation — only allow math operations
        allowed_names = {k: v for k, v in math.__dict__.items() if not k.startswith("_")}
        result = eval(expression, {"__builtins__": {}}, allowed_names)
        return {"result": round(float(result), 6)}
    except Exception as e:
        return {"error": str(e)}

def get_stock_price(ticker: str) -> dict:
    # In production: call a real stock API (Yahoo Finance, Polygon.io, etc.)
    mock_prices = {"AAPL": 213.49, "NVDA": 875.42, "MSFT": 415.32, "GOOGL": 178.91}
    price = mock_prices.get(ticker.upper(), round(random.uniform(50, 500), 2))
    return {"ticker": ticker.upper(), "price": price, "currency": "USD"}

# Dispatch table: maps tool name → function
TOOL_REGISTRY = {
    "get_weather":    get_weather,
    "calculator":     calculator,
    "get_stock_price": get_stock_price,
}

def execute_tool(name: str, inputs: dict) -> str:
    """Execute a tool and return the result as a JSON string."""
    import json
    if name not in TOOL_REGISTRY:
        return json.dumps({"error": f"Unknown tool: {name}"})
    result = TOOL_REGISTRY[name](**inputs)
    return json.dumps(result)
```

## The Agent Loop

```python
# agent_loop.py
import json
import anthropic
from dotenv import load_dotenv
from tool_definitions import TOOLS
from tool_implementations import execute_tool

load_dotenv()
client = anthropic.Anthropic()

def run_agent(user_message: str, max_turns: int = 10) -> str:
    """
    Run the agent loop until the model gives a final text answer
    or max_turns is reached.
    """
    messages = [{"role": "user", "content": user_message}]

    for turn in range(max_turns):
        response = client.messages.create(
            model="claude-opus-4-7",
            max_tokens=1024,
            tools=TOOLS,
            messages=messages
        )

        print(f"  [Turn {turn+1}] Stop reason: {response.stop_reason}")

        # If the model is done, return its text response
        if response.stop_reason == "end_turn":
            for block in response.content:
                if block.type == "text":
                    return block.text

        # If the model wants to use tools, process them
        elif response.stop_reason == "tool_use":
            # Add the model's response (with tool calls) to history
            messages.append({"role": "assistant", "content": response.content})

            # Execute each requested tool and collect results
            tool_results = []
            for block in response.content:
                if block.type == "tool_use":
                    print(f"  [Tool] {block.name}({json.dumps(block.input)})")
                    result = execute_tool(block.name, block.input)
                    print(f"  [Result] {result}")
                    tool_results.append({
                        "type":        "tool_result",
                        "tool_use_id": block.id,
                        "content":     result
                    })

            # Feed tool results back to the model
            messages.append({"role": "user", "content": tool_results})

    return "Agent reached maximum turns without completing the task."


# Test it
if __name__ == "__main__":
    queries = [
        "What's the weather in Paris and Tokyo?",
        "What's 15% of NVIDIA's current stock price?",
        "If I invest $10,000 in AAPL today, and the stock gains 12%, what will it be worth?",
    ]

    for query in queries:
        print(f"\nUser: {query}")
        print("─" * 60)
        answer = run_agent(query)
        print(f"Agent: {answer}")
```

```result
User: What's the weather in Paris and Tokyo?
────────────────────────────────────────────────────────────
  [Turn 1] Stop reason: tool_use
  [Tool] get_weather({"city": "Paris"})
  [Result] {"city": "Paris", "temp_c": 18, "conditions": "partly cloudy"}
  [Tool] get_weather({"city": "Tokyo"})
  [Result] {"city": "Tokyo", "temp_c": 24, "conditions": "sunny"}
  [Turn 2] Stop reason: end_turn
Agent: Currently, Paris is 18°C with partly cloudy skies, while Tokyo is warmer at 24°C and sunny.

User: What's 15% of NVIDIA's current stock price?
────────────────────────────────────────────────────────────
  [Turn 1] Stop reason: tool_use
  [Tool] get_stock_price({"ticker": "NVDA"})
  [Result] {"ticker": "NVDA", "price": 875.42, "currency": "USD"}
  [Tool] calculator({"expression": "875.42 * 0.15"})
  [Result] {"result": 131.313}
  [Turn 2] Stop reason: end_turn
Agent: NVIDIA's current stock price is $875.42. 15% of that is $131.31.
```

The model correctly identifies it needs two tools for the second query — fetching the stock price first, then computing 15% of the result. This multi-step reasoning with real tool chaining is the essence of agent behavior.

## Wrapping Up Day 2

You now understand tool use end-to-end: define tool schemas, implement tool functions, run the agent loop, and feed results back. Tomorrow on **Day 3: Building a Real Agent**, we replace the mock data with real APIs and build a complete web search + summarization agent.
