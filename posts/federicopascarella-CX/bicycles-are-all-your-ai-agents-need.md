---
title: Bicycles Are All Your AI Agents Need
published: false
description: An exploration of how simple, efficient tools can revolutionize your AI agents
tags: 'ai, agents, productivity, technology'
cover_image: ./assets/cover.png
---

## From Condors to Code

Somewhere between a condor and a keyboard lies human genius.
Steve Jobs once told a story about how humans are terrible movers compared to animals. The condor beats us easily in the race of energy efficiency. But then someone put a human on a bicycle—and we soared straight to the top. The bicycle, Jobs said, is “a tool that amplifies our efficiency.” Computers, he added, are bicycles for the mind.
That thought never left me. And now, with AI agents evolving faster than a Tour de France sprint, I can’t help seeing the same pattern repeat: tools are still the key. Only this time, the cyclists are our AI agents, and the bicycles are the functions we build for them.
Yes, our AI companions have something that looks like a brain (the LLM), a short-term memory (context window), and even hands (tools). But just like us, they’re useless without the right equipment. You wouldn’t ride a mountain stage on a unicycle, right? So why make an AI agent do complex work with clunky, poorly designed tools?
Let’s pedal into that thought.

## The Engineering of Great Tools

Behind every efficient AI agent lies a set of humble Python functions that do the heavy lifting. These aren’t glamorous, but they’re what make the agent reliable, fast, and clever. Building them well is both an art and a discipline.

### 1. Single Responsibility: One Tool, One Job

The best functions are like good mechanics: they specialize.
A single-responsibility function doesn't try to save a user and send them a welcome email and make coffee. It does one thing—beautifully.
When you start adding Boolean flags like send_email=True, you're basically handing your agent a Swiss Army knife with half the blades stuck. Instead, write smaller, atomic functions: one for saving, one for emailing. Your agent can chain them together as needed. Simple is elegant—and much easier to debug when something goes sideways.

```python
# ❌ Bad: Swiss Army knife function
def create_user(name, email, send_welcome=True, log=True):
    user = db.save(name, email)
    if send_welcome:
        send_email(email, "Welcome!")
    if log:
        logger.info(f"User {name} created")
    return user

# ✅ Good: Focused, composable tools
def save_user(name: str, email: str) -> dict:
    return db.save(name, email)

def send_welcome_email(email: str) -> bool:
    return send_email(email, "Welcome!")
```

### 2. Clear Interfaces: Talk Straight

Name things like you'd explain them to your mother—if your mother happened to build APIs.
calculate_discount(price, percentage) tells a story. discCalc(p, x) is an unfinished crossword puzzle.
Keep arguments few, explicit, and clear. Don't hide behavior behind mysterious flags or side effects. And for the love of all coders—return data, don't print it. Your AI agent doesn't read your console output; it just stares into the void, confused.

```python
# ❌ Bad: Unclear names and prints output
def discCalc(p, x, t=None):
    result = p - (p * x / 100)
    print(f"Discount applied: ${result}")

# ✅ Good: Clear names and returns data
def calculate_discount(price: float, percentage: float) -> float:
    discounted_price = price - (price * percentage / 100)
    return discounted_price
```

### 3. Structured Outputs: Speak JSON, Not Poetry

Humans love sentences. AI agents prefer structure.
When a tool returns {"city": "London", "temperature": 15}, the agent knows exactly what to do. When it returns "It's about fifteen degrees and cloudy, mate," the poor thing is parsing weather small talk.
Structured outputs—dicts, lists, or JSON—make your agent efficient, predictable, and less prone to hallucinate its way through your data. Predictability is kindness in the AI world.

```python
# ❌ Bad: Unstructured string output
def get_weather(city):
    temp = fetch_temperature(city)
    return f"It's about {temp} degrees in {city}, partly cloudy"

# ✅ Good: MCP tool with structured schema
from pydantic import BaseModel, Field
from mcp.server import Server
from mcp.types import Tool

# 1. Define the output schema
class WeatherData(BaseModel):
    city: str = Field(description="City name")
    temperature: float = Field(description="Temperature in Celsius")
    condition: str = Field(description="Weather condition")
    humidity: int = Field(description="Humidity percentage")

# 2. Register the MCP tool with schema
@server.call_tool()
async def get_weather(city: str) -> WeatherData:
    try:
        temp = await fetch_temperature(city)
        condition = await fetch_condition(city)
    except Exception as e:
        raise ValueError(f"Failed to fetch weather for {city}: {e}")
    
    return WeatherData(
        city=city,
        temperature=temp,
        condition=condition,
        humidity=65
    )

# 3. LLM receives and processes structured data
# Agent: "Get weather for London"
# Tool returns: {"city": "London", "temperature": 15, "condition": "partly_cloudy", "humidity": 65}
# LLM understands: It's 15°C in London with 65% humidity
```

### 4. Efficiency: Be Fast, Not Furious

Your agent may call your tool hundreds of times a minute. That means every unnecessary loop, every over-fetched dataset, every redundant API call adds up like a bad gym membership.
Use Python's built-ins. Cache where it makes sense. And profile before you optimize—because guessing where the bottleneck is usually feels like fixing a flat tire in the dark.

```python
# ❌ Bad: Inefficient manual loops
def filter_active_users(users):
    result = []
    for user in users:
        if user.get("active"):
            result.append(user)
    return result

# ✅ Good: Python built-ins with caching
from functools import lru_cache
from typing import Tuple, List

@lru_cache(maxsize=128)
def filter_active_users(users_tuple: Tuple[dict, ...]) -> List[dict]:
    return [u for u in users_tuple if u.get("active")]
```

### 5. Robustness: Don't Crash the Ride

Good code doesn't just work—it fails gracefully.
Use proper error handling. Log intelligently (no random print confessions). Validate inputs like a suspicious bouncer at a nightclub.
And please, no silent exceptions. If your tool crashes and no one hears it, the agent still feels the pain.

```python
# ❌ Bad: No validation or error handling
def read_file(path):
    with open(path) as f:
        return f.read()

# ✅ Good: Validates input and handles errors
def read_file(path: str) -> str:
    if not isinstance(path, str) or not path:
        raise ValueError("Path must be a non-empty string")
    try:
        with open(path, 'r') as f:
            return f.read()
    except FileNotFoundError:
        raise FileNotFoundError(f"File not found: {path}")
```

### 6. The Micro-Tooling Mindset

Now, imagine breaking every big process into bite-sized, composable tools: micro-tooling.
Each small function becomes a cog in a larger machine—a clean, auditable, interchangeable part that can be tested, reused, and upgraded independently.
This design pattern, much like microservices but at function scale, is perfect for MCP (Model Context Protocol) systems. It gives agents Lego-like flexibility: they can dynamically chain the right tools, adapt plans, and build reasoning pipelines on the fly.
Think of it as giving your AI an organized toolbox instead of the chaotic drawer everyone has under their sink. MCP just knows where each wrench belongs.

```python
# ❌ Bad: Monolithic data processing
def process_user_data(user_id):
    user = db.fetch(user_id)
    validated = validate(user)
    enriched = api.enrich(validated)
    return transform(enriched)

# ✅ Good: Composable micro-tools for MCP
def fetch_user(user_id: str) -> dict:
    return db.fetch(user_id)

def validate_user(user: dict) -> dict:
    return validate(user)

def enrich_user(user: dict) -> dict:
    return api.enrich(user)
```

### 7. Trade-Offs (Because Nothing’s Ever Free)

Micro-tooling has its challenges:
Too many small tools can create orchestration chaos. Too much chaining can slow things down. And managing hundreds of tiny functions can feel like herding caffeinated squirrels.
But when done right—with clear naming, documentation, and predictable inputs/outputs—it becomes the foundation for scalable, explainable, and maintainable AI systems.

## Conclusion - Pedaling into the Future

At CloudX, we’ve learned that innovation often hides in simplicity. Building efficient AI agents isn’t about giving them infinite intelligence—it’s about giving them great bicycles.
Write clean, focused, well-documented tools. Think in micro-tooling: small parts, big impact. Keep your agent’s toolbox sharp, organized, and light.
Because when your tools are great, your agents stop crawling and start flying—on two beautifully engineered wheels.
So, next time you’re debugging that stubborn Python function, just remember:
You're not fixing a bug. You're tuning a bicycle for the mind of an AI. 🚲

## References

Fede Pascarella
