---
title: 'How I Built a Personal AI SuperAPP by Wrapping Codex App Server'
published: false
description: 'How codex app-server turned a terminal AI workflow into a personal super-app with subagents, shared context, prompt improvement, and output explanations.'
tags: 'ai, codex, agents, cli'
cover_image: ./assets/codex-superapp-cover.png
canonical_url: https://app.kycode.dev/articulos#2
---

![Hero banner showing a Codex App Server wrapper becoming a personal AI super-app](./assets/codex-superapp-cover.png)

For months I used Codex like everyone else: one terminal, one session, one long output. Then I found `codex app-server`: the same engine exposed as JSON-RPC over stdio. It gave me a more useful idea: building my own interface for the work I actually do.

## What a super-app means today

OpenAI has been pretty explicit about this: a [**unified AI superapp**](https://openai.com/index/next-phase-of-enterprise-ai/) is a place where agents, tools, context, and history live together. Instead of jumping from chat to terminal, from terminal to browser, from browser to docs, everything happens in one working surface.

Theo built a very concrete version of that intuition with [T3 Chat](https://t3.chat): he got tired of waiting for the perfect chat app and built his own. My version started from a similar feeling: Codex was already great, but the way I actually work needed a different interface on top.

## I built my own super-app

Some context first. Over the last months I've been building a desktop app that wraps Codex. It runs several agent sessions in parallel in a grid, improves my prompts before the agent sees them, explains the agent's output in plain language, and spawns subagents with one click.

I never planned a product. I automated my own frictions, one at a time, until the wrapper became the place where I actually work. Theo did the same thing with [T3 Chat](https://t3.chat): no existing AI chat fit him, so he built his own. Same energy here.

My point is that you can build yours too. The reason is simple: the engine is already exposed, sitting on your machine, waiting. You just need to find the door.

## The super-app starts with a small door

Most people use Codex the way it's marketed: an agent chatting in your terminal. But the same binary ships another mode:

```bash
codex app-server
```

That subcommand turns the CLI into a server that speaks **JSON-RPC over stdio**.

And you only need a little to do something real with it:

- `thread/start`: open a session
- `turn/start`: give it work
- `turn/steer`: inject a message into a turn that's already running

There are more (`thread/resume`, `turn/interrupt`, `thread/settings/update`), but three are enough for Codex to stop being a window and become an engine.

## First need: subagents in ~10 lines

The feature I needed was simple: one click, and a fresh Codex instance inherits my session's context and chases a parallel idea while my main session keeps its focus.

```javascript
const context = packTimeline(parent.timeline, 48_000); // newest-first
const briefing = [
  "You are a sub-agent delegated from an active session…",
  houseRules,
  context,
  newTask,
].join("\n\n");

const child = await rpc("thread/start", { cwd });
await rpc("turn/start", { threadId: child.id, input: briefing });
```

That's the whole thing. The important part was turning one of my frictions into a button I could press.

That's the mental shift: a super-app can be small and concrete; it gathers the operations that used to be scattered across your head, your terminal, your notes, and your prompts.

![Illustrative screenshot of a main Codex session delegating context to a subagent session](./assets/subagent-session-screenshot.png)

_Example: one main session keeps focus while a delegated subagent inherits context and tests edge cases._

## Context is everything

Spawning a second agent was never the problem. A second agent **that knows nothing** is the problem.

The briefing packs project name, working directory, current focus, and a snapshot of the parent's timeline capped at **48,000 characters**, newest entries first. The child's window shows only the task you typed; the runtime receives the full context.

And the door swings both ways. When the child is created, the parent gets a coordination note with `turn/steer` or `turn/start`. When the child finishes or gets stuck, it reports back explicitly: `task_completed`, `blocked`.

## Needs that appear once the wrapper exists

Subagents were the first one. Then I looked at the rest of my day and saw the same pattern: small repeated frictions that no tool was going to prioritize for me.

![Conceptual illustration of a personal AI super-app ecosystem: connected modules around a central engine](./assets/codex-superapp-ecosystem.png)

**My prompts get improved before Codex ever sees them.** When I hit send, a dedicated investigator digs through the repo and extracts high-signal context. Then another model rewrites my messy draft into a precise prompt and shows me why it rewrote it that way.

**Codex writes code; Claude explains it.** Codex writes great code. Claude makes it easier to understand. So when a turn finishes, my wrapper hands the raw output to Claude with one simple rule: don't summarize, re-explain.

![Example of a prompt transformer restructuring a vague user prompt into a clear XML-style task brief](./assets/transformer-screenshot-1-prompt-comparison.png)

_Example: how the transformer restructures a vague prompt into clarity._

![Example of Claude turning dense Codex output into a clear explanation with next steps](./assets/transformer-screenshot-2-explanation-modal.png)

_Example: how Claude explains what Codex output was missing and how to improve it._

Notice the pattern: GPT writes, Claude explains, Codex executes, and my wrapper coordinates. At that point, the UI layer turns into something else: a personal super-app for working with agents.

## Build yours

Start with a friction before thinking about a platform.

1. Launch `codex app-server`.
2. Talk to it over JSON-RPC.
3. Pick one action you repeat every day.
4. Turn that action into an interface.

The big version of the AI world is moving toward super-apps: one place where agents, tools, and context mix. But the useful version can start on your machine, with a button that solves something that bothered you yesterday.

A year ago I wrote that [agents are just loops](https://dev.to/cloudx/forget-the-hype-agents-are-loops-1n3i). Here's this year's version: **a super-app is loops, context, and the right model in the right place**.

Next time you think "I wish Codex could do this", it probably can. You just have to wrap it.

Questions? Leave a comment below.
