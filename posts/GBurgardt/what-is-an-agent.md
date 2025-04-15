---
title: "AI Agents: It's Simpler Than You Think (Just a Loop)"
published: true
description: "Forget the hype. AI Agents are basically LLMs running inside a loop with access to tools. Let's break down the code and see how tools like Cursor use this."
tags: "ai, programming, agents, llm, function-calling, pragmatic, cursor"
cover_image: ./assets/cover_agent_loop_cursor.png
id: [generate a unique ID, e.g., 2368991]
---

## Forget the Hype: Agents are Loops

"AI Agent" sounds complex. But at its core, an **AI Agent is often just a program running a Large Language Model (LLM) inside a loop.**

Think of a simple `while True` loop. That's basically the engine of many agents. Look at this pseudocode:

```python
# Set up the context/state
env = Environment()
# Define the tools the agent can use (including their parameters)
tools = Tools(env)
# Tell the LLM its goal and how to behave (including tool usage strategy)
system_prompt = "Your goals, constraints, available tools, and how to act..."

# The main agent loop
while True:
    # 1. LLM thinks: decides the next action based on prompt and current state
    action = llm.run(system_prompt + env.state)

    # 2. System acts: executes the action (if it's a tool call) and updates the state
    env.state = tools.run(action)
```

_This simple loop is the heart of an Agent._

Let's break it down:

1.  **`llm.run(system_prompt + env.state)`**: This is the "brain". The LLM gets instructions (`system_prompt`) and the current situation (`env.state`). It decides the next `action`—either generating text or requesting a tool.
2.  **`tools.run(action)`**: This is the "hands". If the `action` is a tool request (like "edit file"), this function executes the _real_ action and updates the state (`env.state`) with the result (e.g., "File updated"). If it's just text, it might record it.

The loop continues: LLM sees updated state -> Decides next action -> System executes -> Repeat. (Real implementations add a way to `break` this loop).

## Why Tools? Because LLMs Just Talk

The LLM itself only outputs text or structured data (like JSON). **It can't directly interact with the outside world** – it can't browse websites, modify files on your disk, or call APIs. It's like a brain without hands or senses.

Think about a human developer: they use their brain to think and solve problems, but they also rely on tools to actually _do_ things – reading files, editing code, searching the web, checking documentation, running terminal commands, and so on.

**Tools** bridge this same gap for the Agent. They are the specific functions the `tools.run()` part of the loop executes when the LLM asks, giving the LLM "hands" and "senses" to interact with the world beyond just generating text.

## Defining a Tool: What it is vs. When to Use It

So the LLM can request actions with `tools`. But how does it know _which_ tool to request and _when_? This involves two key pieces of information: the tool's definition and the agent's system prompt.

**1. Tool Definition (Name, Description & Parameters):**

Each tool available to the agent needs to be clearly defined. **Crucially, this definition includes the specific input parameters the tool requires.** A typical tool definition has:

- **Name:** A unique identifier (e.g., `readFile`, `editFile`).
- **Description:** Text explaining **what the tool is and what it does fundamentally**. (e.g., For `editFile`: "Modifies the content of a specified file...").
- **Input Parameters:** A specific list of the inputs the tool needs. Each parameter is defined with its own name, type (e.g., string, boolean), and description, telling the LLM exactly what information to provide for that input. **These parameters are fixed when the tool is defined.**

This complete definition helps the LLM understand the tool's capability and exactly what information it needs to gather and provide when requesting the tool.

```pseudocode
// Example definition for a tool, including explicitly defined parameters
Tool {
  name: "findFile",
  description: "Searches the workspace directory for files matching a given pattern or name.",
  parameters: [
    {
      name: "pattern",
      type: "string",
      description: "The file pattern to search for (e.g., '*.css', 'README.md'). Required."
    },
    {
      name: "search_recursively",
      type: "boolean",
      description: "Whether to search in subdirectories. Defaults to true."
    }
    // Other parameters defined here...
  ]
}
```

**2. System Prompt (Usage Context):**

The **main System Prompt** you provide to the agent (the `system_prompt` variable in our loop example) usually guides _when_ to use which tool. It tells the LLM its overall goal, constraints, lists the available tools, and often provides strategies for their use.

For example, the system prompt might say:

_"You have tools `findFile`, `readFile`, `editFile`. When you need a file but don't know the path, use `findFile` with an appropriate `pattern`. Once you have the path, use `readFile` before using `editFile`..."_

**How it comes together:**

In each loop cycle (`llm.run(...)`), the LLM considers the current `env.state`, its goal, the **usage instructions from the system prompt**, and the **full definitions (name, description, required parameters) of the available tools**. Based on all this, it decides if a tool is needed. If yes, it uses the complete definition to formulate the correct `action` request, ensuring it provides values for the required parameters based on the context (e.g., `{ "tool_name": "findFile", "parameters": { "pattern": "*.css", "search_recursively": true } }`). The `tools.run(action)` part then takes over execution.

### Real-World Example: The Cursor Editor Agent

Many developers use **Cursor**, an AI-powered code editor. When you give it a complex task, it acts exactly like the loop model we described. It's an Agent using multiple tools, each defined with its name, description, and input parameters, all guided by a system prompt.

Think about the tools Cursor might have (each defined with specific parameters):

- `readFile(path: string)`: Reads the content of a file at the given path.
- `editFile(path: string, changes: list)`: Modifies a file by applying a list of changes.
- `findFile(pattern: string, recursive?: boolean)`: Searches for files matching a pattern.
- `readFileChunk(path: string, startLine: int, endLine: int)`: Reads a specific range of lines.
- `runCommand(command: string, workingDir?: string)`: Executes a shell command.
- `webSearch(query: string)`: Searches the web.
- `moveFile(sourcePath: string, destinationPath: string)`: Renames or moves a file.

**Let's trace a simple task in Cursor:** You ask it, "Refactor the `UserProfile.jsx` component to fetch user data from `/api/users/:id` instead of props, and update its styling in `styles.css`."

The Cursor agent's system prompt likely guides the refactoring process. The agent might work internally like this:

1.  **Loop 1 (Think):** Analyzes request + strategy. Needs `UserProfile.jsx`.
    **Loop 1 (Act):** Requests `readFile` tool with `path="UserProfile.jsx"`. `tools.run` executes, state updates.
2.  **Loop 2 (Think):** Reads code. Needs styles. Strategy: Find related files if path unknown.
    **Loop 2 (Act):** Requests `findFile` tool with `pattern="styles.css"`. `tools.run` finds it, state updates.
3.  **Loop 3 (Think):** Has path. Strategy: Read before edit.
    **Loop 3 (Act):** Requests `readFile` tool with `path="styles.css"`. `tools.run` executes, state updates.
4.  **Loop 4 (Think):** Has context. Plans JSX changes. Strategy: Apply changes.
    **Loop 4 (Act):** Requests `editFile` tool with `path="UserProfile.jsx"` and the necessary `changes`. `tools.run` applies them. State updates.
5.  **Loop 5 (Think):** Plans CSS changes. Strategy: Apply changes.
    **Loop 5 (Act):** Requests `editFile` tool with `path="styles.css"` and `changes`. `tools.run` applies them. State updates.
6.  **Loop 6 (Think):** Strategy: Lint after edits.
    **Loop 6 (Act):** Requests `runCommand` tool with `command="npx eslint --fix UserProfile.jsx"`. `tools.run` executes. State updates.
7.  **Loop 7 (Think):** Reviews state. Goal achieved per prompt. Loop stops or waits.

This Cursor example shows the **LLM + Loop + Tools** pattern clearly. The LLM acts as the brain, following the **system prompt strategy** and using complete **tool definitions (including parameters)** to plan steps, while the system executes those tools, cycling until the goal is achieved.

## Simple Example: Debugging Agent Revisited

_(Keeping the previous simple example for contrast)_

Let's apply this to the debugging idea:

- **Goal & Strategy (in `system_prompt`):** "Fix the bug in `script.py`. Use `edit_file` to apply corrections..."
- **Tool Available:** `edit_file(path: string, changes: list)` (Description: "Modifies the content...")
- **Loop Cycle 1:**
  - `env.state` contains the error message.
  - `action = llm.run(...)`: LLM analyzes state + strategy, decides to edit. Outputs: `{ "tool_name": "edit_file", "parameters": { "path": "script.py", "changes": [...] } }`
  - `env.state = tools.run(action)`: System executes edit. State updates to: `"Success: File script.py updated."`
- **Loop Cycle 2:**
  - `env.state` contains "Success...".
  - `action = llm.run(...)`: LLM sees success. System prompt might say "After editing, run tests". LLM requests `run_tests` tool (assuming it's defined with its parameters).
  - ...and the loop continues.

## The Pragmatic Takeaway: LLM + Loop + Tools

So, next time you hear "AI Agent," forget the fancy talk. Just picture that simple loop, running with good instructions:

1.  **LLM Thinks (using System Prompt instructions + Tool definitions)**
2.  **System Acts (running the requested Tools with their inputs)**
3.  **Repeat**

They work well because the LLM follows the main instructions (the system prompt) and uses the available tools correctly inside that loop. It's a simple pattern that gets the job done. That's how it really works.
