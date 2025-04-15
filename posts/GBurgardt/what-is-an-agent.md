---
title: "AI Agents: It's Simpler Than You Think (Just a Loop)"
published: true
description: "Forget the hype. AI Agents are basically LLMs running inside a loop with access to tools. Let's break down the code and see how tools like Cursor use this."
tags: "ai, programming, agents, llm, function-calling, pragmatic, cursor"
cover_image: ./assets/cover_agent_loop_cursor.png
id: [generate a unique ID, e.g., 2368991]
---

## Forget the Hype: Agents are Loops

"AI Agent" sounds complex, but it's often simpler than you think. At its core, an **AI Agent usually runs a Large Language Model (LLM) inside a loop.**

Think of a basic loop – that's the engine for many agents. This pseudocode shows the main idea:

```python
# Environment holds the current state/context
env = Environment()
# Tools available to the agent (defined with parameters)
tools = Tools(env)
# Instructions for the LLM: goals, rules, tools list, strategy
system_prompt = "Your goals, constraints, available tools, and how to act..."

# The main agent loop
while True:
    # 1. LLM Brain: Decides next action based on prompt + state
    action = llm.run(system_prompt + env.state)

    # 2. System Hands: Runs the *actual code* for the requested action/tool
    env.state = tools.run(action) # This calls the actual code for the tool
```

_This simple loop is the heart of an Agent._

_A simplified view of the Agent's cycle: Think -\> Act -\> Update State -\> Repeat._

Let's break down the loop:

1.  **`llm.run(...)`**: The "brain". It gets instructions (`system_prompt`) and the current situation (`env.state`), then decides the next `action` (text or a tool request).
2.  **`tools.run(action)`**: The "hands". If `action` requests a tool, this executes the _real code_ for that tool and updates the `env.state` with the result.

The loop repeats: LLM sees the new state, decides the next action, the system runs it. (Real agents need a condition to `break` the loop).

## Why Tools? Because LLMs Just Talk

The LLM itself only outputs text or structured data. **It can't directly interact with the outside world** – no Browse, no file editing, no API calls. It's like a brain without hands.

Think about a human developer: they use their brain to solve problems, but need tools to _do_ things – read files, edit code, search the web, run commands.

**Tools** bridge this gap for the Agent. They are functions the `tools.run()` part executes when the LLM asks, giving the LLM ways to interact with the world.

## Defining a Tool: What it is vs. When to Use It

How does the LLM know _which_ tool to request and _when_? It uses two key things:

**1. Tool Definition (Name, Description & Parameters):**

Each tool needs a clear definition, including:

- **Name:** Unique ID (e.g., `readFile`).
- **Description:** What the tool _is_ and _does_ (e.g., "Reads a file's content").
- **Input Parameters:** The _exact_ inputs needed (name, type, description for each). These are fixed.

This definition tells the LLM the tool's capability and required inputs.

```pseudocode
// Example Tool Definition
Tool {
  name: "findFile",
  description: "Searches workspace for files matching a pattern.",
  parameters: [
    { name: "pattern", type: "string", description: "Search pattern (e.g., '*.css'). Required." },
    { name: "recursive", type: "boolean", description: "Search subdirs? Default: true." }
  ]
}
```

_A Tool is defined by its Name, Description, and the specific Parameters it needs._

**2. System Prompt (Usage Context):**

The **main System Prompt** (`system_prompt` in the code) guides the LLM's strategy. It lists available tools and explains _when_ to use them.
_Example instruction:_ _"If you need a file but don't know the path, use `findFile` first..."_

### Real-World Example: The Cursor Editor Agent

Many developers use **Cursor**. When you give it a task, it acts as an Agent using this loop model.

_Agents like Cursor use an LLM brain connected to various tools._

Cursor likely has tools like these (each with code behind it):

- `readFile(path: string)`
- `editFile(path: string, changes: list)`
- `findFile(pattern: string, recursive?: boolean)`
- `runCommand(command: string, workingDir?: string)`
- `webSearch(query: string)`
- etc...

**Tracing a simple Cursor task:** You ask: "Refactor `UserProfile.jsx` to fetch data instead of using props, and update `styles.css`."

The agent might work like this (using its internal `system_prompt` which defines its general coding/refactoring strategy):

1.  **Loop 1:** LLM gets your request (now part of `env.state`) + its system prompt. Strategy: Understand current code first. Requests `readFile(path="UserProfile.jsx")`. System runs it, updates state.
2.  **Loop 2:** LLM analyzes code. Needs styles. Path unknown? Strategy: find it. Requests `findFile(pattern="styles.css")`. System runs it, updates state.
3.  **Loop 3:** LLM has path. Strategy: read before edit. Requests `readFile(path="styles.css")`. System runs it, updates state.
4.  **Loop 4-N (...):** Based on the file contents and its strategy, the LLM plans changes and requests `editFile` for JSX, then `editFile` for CSS. Maybe requests `runCommand` to lint. System executes each, updating state.
5.  **Final Loop:** LLM reviews state (changes applied, linter OK). Task complete per system prompt rules. Loop stops/waits.

This shows the **LLM + Loop + Tools** pattern: LLM plans using instructions and tool definitions, System executes the actual tool code.

## Simple Example: Debugging Agent Revisited

Let's revisit the debugging idea with a clearer separation of instructions and the specific task:

- **System Prompt (`system_prompt`):**
  ```
  You are an AI debugging assistant. Your goal is to fix bugs in Python scripts.
  Available Tools:
  - edit_file(path: string, changes: list): Modifies a file. Use this to apply code corrections.
  - run_tests(path: string): Executes tests for the specified file. Use this after applying a fix to verify it.
  Strategy: Analyze the error message and code context. Use edit_file to correct the code. After editing, use run_tests to confirm the fix.
  ```
- **Initial Task / State (`env.state`):**
  ```
  User Request: Fix this error!
  Error in script.py, line 25: TypeError: unsupported operand type(s) for +: 'int' and 'str'.
  File path: script.py
  ```
- **Loop Cycle 1:**
  - **Think:** `llm.run(system_prompt + env.state)` -\> LLM analyzes the `TypeError` in `script.py` (from state) using its debugging strategy (from prompt). It decides to use the `edit_file` tool.
  - **Act:** LLM outputs action: `{ "tool_name": "edit_file", "parameters": { "path": "script.py", "changes": [{ "line": 25, "new_code": "..." }] } }`.
  - System executes the `edit_file` code. `env.state` updates to: `"Success: File script.py updated. Last error: TypeError..."`.
- **Loop Cycle 2:**
  - **Think:** `llm.run(system_prompt + env.state)` -\> LLM sees "Success..." (from state). Its strategy (from prompt) says to run tests after editing. It decides to use `run_tests`.
  - **Act:** LLM outputs action: `{ "tool_name": "run_tests", "parameters": { "path": "script.py" } }`.
  - System executes the `run_tests` code. `env.state` updates with test results... Loop continues.

## The Pragmatic Takeaway: LLM + Loop + Tools

So, next time you hear "AI Agent," forget the fancy talk. Just picture that simple loop, running with good instructions:

1.  **LLM Thinks (using System Prompt + Tool definitions + Current State)**
2.  **System Acts (running actual code for requested Tools)**
3.  **Repeat**

They work well because the LLM follows instructions and uses tools correctly inside the loop. It's a simple pattern that gets the job done. That's how it really works.
