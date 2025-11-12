---
title: Bicycles Are All Your AI Agents Need
published: true
description: How simple, reliable tools make AI agents effective
tags: 'ai, agents, productivity, technology'
cover_image: ./assets/cover.png
id: 3017254
---

## From Condors to Code

Somewhere between a condor and a keyboard lies human genius. Steve Jobs once told a story about how humans are terrible movers compared to animals. The condor beats us easily in the race of energy efficiency, but put a person on a bicycle and they fly.
The bicycle, Jobs said, is "a tool that amplifies our efficiency." Computers, he added, are bicycles for the mind.
That thought never left me. And now, with AI agents evolving super-fast, I can't help seeing the same pattern repeat.
My humble view is that tools are still the key. Only this time, the cyclists are our AI agents with its brain (the LLM), and the bicycles are the functions we build for them.
With the right tools, an agent moves with purpose. With clumsy tools, it stalls.

## The Engineering of Great Tools

Great agents sit on top of small, sharp Python functions. They are plain, predictable, and fast.

### 1. Single Responsibility

Specialize each function. Do one job well, then compose.

```python
# Bad: Swiss-army function
def create_user(name, email, send_welcome=True, log=True):
    user = db.save(name, email)
    if send_welcome:
        send_email(email, "Welcome!")
    if log:
        logger.info(f"User {name} created")
    return user

# Good: Focused, composable tools
def save_user(name: str, email: str) -> dict:
    return db.save(name, email)

def send_welcome_email(email: str) -> bool:
    return send_email(email, "Welcome!")
```

### 2. Clear interfaces

Name things so intent is obvious. Keep arguments explicit. Return data instead of printing.

```python
# Bad: Vague names and side effects
def discCalc(p, x, t=None):
    result = p - (p * x / 100)
    print(f"Discount applied: ${result}")

# Good: Straight names and returns
def calculate_discount(price: float, percentage: float) -> float:
    return price - (price * percentage / 100)
```

### 3. Structured outputs

Agents prefer structure. Return dicts or JSON, not prose.

```python
# Bad: Unstructured string
def get_weather(city):
    temp = fetch_temperature(city)
    return f"It's about {temp} degrees in {city}, partly cloudy"

# Good: MCP tool with schema
from pydantic import BaseModel, Field
from mcp.server import Server
from mcp.types import Tool

server = Server("weather")

class WeatherData(BaseModel):
    city: str = Field(description="City name")
    temperature: float = Field(description="Temperature in Celsius")
    condition: str = Field(description="Weather condition")
    humidity: int = Field(description="Humidity percentage")

@server.call_tool()
async def get_weather(city: str) -> WeatherData:
    temp = await fetch_temperature(city)
    condition = await fetch_condition(city)
    return WeatherData(city=city, temperature=temp, condition=condition, humidity=65)
```

### 4. Efficiency

Use built-ins, cache where it helps, and profile before optimizing.

```python
# Bad: Manual loops
def filter_active_users(users):
    result = []
    for user in users:
        if user.get("active"):
            result.append(user)
    return result

# Good: Built-ins plus caching
from functools import lru_cache
from typing import Tuple, List

@lru_cache(maxsize=128)
def filter_active_users(users_tuple: Tuple[dict, ...]) -> List[dict]:
    return [u for u in users_tuple if u.get("active")]
```

### 5. Robustness

Validate inputs and fail loudly with helpful errors.

```python
# Bad: No validation
def read_file(path):
    with open(path) as f:
        return f.read()

# Good: Validation and clear errors
def read_file(path: str) -> str:
    if not isinstance(path, str) or not path:
        raise ValueError("Path must be a non-empty string")
    try:
        with open(path, "r") as f:
            return f.read()
    except FileNotFoundError:
        raise FileNotFoundError(f"File not found: {path}")
```

### 6. The micro-tooling mindset

Break big jobs into small tools you can test and swap. MCP benefits from chains of simple, named steps.

```python
# Bad: Monolith
def process_user_data(user_id):
    user = db.fetch(user_id)
    validated = validate(user)
    enriched = api.enrich(validated)
    return transform(enriched)

# Good: Composable steps
def fetch_user(user_id: str) -> dict:
    return db.fetch(user_id)

def validate_user(user: dict) -> dict:
    return validate(user)

def enrich_user(user: dict) -> dict:
    return api.enrich(user)
```

### 7. Trade-offs

Hundreds of tiny tools can create orchestration overhead. Clear names, steady input and output shapes, and basic docs keep things manageable.

## Show, Don't Tell: Two Decision Flows

A concrete example makes the difference clear. Here is the same task, done with weak tools and with strong tools.

**Task:** Extract newly signed customers from a CSV in cloud storage, enrich each with firmographic data, and email an account summary.

### Agent with poorly designed tools

1. Calls a generic `process_file()` that auto-detects type and tries to parse everything.
2. Uses one do everything `enrich_user()` that accepts many flags, then times out on third party rate limits.
3. Prints logs to stdout, returns a mixed string summary, and the agent fails to decide what to send.

### Decision flow with weak tools

* Input: blob path
* Branch: auto-detect format, guess schema
* Loop: enrich with side effects
* Output: unstructured string
* Failure mode: retries loop, hallucinates missing fields, no clear errors

### Agent with well designed tools

1. `load_csv(path, schema)` returns a typed dataframe.
2. `batch_enrich(users, provider, rate_limit)` yields structured rows with retry metadata.
3. `render_account_summary(users)` returns JSON for `send_email(to, subject, body_html)`.

### Decision flow with strong tools

* Input: explicit path and schema
* Transform: strict parser
* Enrich: idempotent, rate limited, returns status per row
* Render: deterministic template
* Output: email send result with IDs

Result: same goal, three clean steps, easy to test and to explain.

## Conclusion

I believe that innovation often hides in simplicity. Building efficient AI agents isn't about giving them infinite intelligence; it's about giving them great tools. Write them clean, focused, and well-documented; think in micro-tooling: small parts, big impact.
So, next time you're debugging that stubborn Python function, just remember: you're not fixing a bug. You're tuning a bicycle for the mind of an AI.
