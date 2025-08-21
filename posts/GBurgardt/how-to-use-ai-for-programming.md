---
title: "How to Use AI for Programming: A Clarity First Approach"
published: false
description: "Learn why clear communication beats 'magic prompts' when coding with AI. Discover how treating AI as a literal collaborator improves your programming skills."
tags: ai, programming, productivity, tutorial
cover_image: ./assets/cover_placeholder.png
---

# How to Use AI for Programming

When we code using AI we ask ourselves: "what's the best prompt?" or "what magic prompt should I use?".

We'd be better off asking: "what kind of interaction is this?". Trying to understand the nature of the interaction between us and the model.

Maybe the problem isn't the technology, but us.

## A Thought Experiment

Imagine you hire a remote programmer. Brilliant, but with some quirks:

- Never worked on your project before (0 context)
- Extremely literal. If you don't explicitly tell them, they never assume anything.
- Doesn't infer context
- Completely loses their memory every day, returning to their initial state

How would you communicate with them?

You'd probably:

- Explain all the necessary context, very detailed
- Be very specific with requirements
- Not assume they'll "figure out" anything. You explain everything
- Expect some iterations before the final result
- Maybe save context files to resend them every day

That's the best way to interact with an AI model.

## AI As a Mirror

The model isn't just a task executor. It's also a mirror of your clarity when communicating a problem.

If you give it vague instructions, you get vague results simply because it faithfully reflects how vague your thinking was.

Most of the time when the model "doesn't understand" the problem isn't the model. It's that we ourselves weren't clear about what we wanted.

## Clarity As a Skill

The real skill isn't "writing good prompts". It's thinking clearly about problems and communicating that clarity. This is a fundamental skill for any programmer.

### Example

**What we usually do:**

```
Optimize this function
```

**Why it fails:** Optimize in what sense? Speed? Memory? Readability? There's no success criteria.

**What we should do:**

```
The processOrders() function in orders.js takes 5 seconds with 1000 orders.
I need it to take less than 1 second.
Orders come from the database already sorted by date.
You can assume there are no duplicate orders.
Logs: <<detailed logs>>
```

This is much clearer and less abstract. It describes:

- The problem (5 seconds is too much)
- The measurable goal (less than 1 second)
- Constraints (already sorted)
- Assumptions (no duplicates)

## The Work It Doesn't Save You

AI won't save you the work of thinking, to communicate effectively you first need to clearly understand what you're going to communicate. The clarification process is valuable work in programming.

## Problem Decomposition

One of the skills that improves working with AI is problem decomposition.

Instead of:

```
Implement a complete authentication system
```

You learn to think:

```
Step 1: Define the User model with minimum required fields: <fields>
Step 2: Create the registration endpoint with basic validation (validation type, etc)
[etc...]
```

## The Limitations

AI can only handle 3-4 files well at a time. It's a limitation but with its bright side:

It forces you to:

- Keep responsibilities separated
- Create clear interfaces
- Avoid coupling
- Think in small modules

It incentivizes you to follow good architecture practices.

## Iteration vs Perfection

With AI don't expect perfection on the first try. Like with a human collaborator, the process is:

1. First approximation
2. Specific feedback
3. Refinement
4. Repeat

### The Importance of Context

AI needs all the context possible, don't skimp.

```
CONTEXT: Users report the checkout page hangs
SYMPTOM: The "Pay" button stays in "Processing..." state indefinitely
FILE: checkout.js, handlePayment() function
SUSPICION: Probably missing a catch to handle API errors
TASK: Add robust error handling and visual feedback to the user
```

### The Value of Programming with AI

Programming with AI trains you in thinking clearly and communicating precisely. It forces you to decompose problems into manageable pieces and be explicit with your requirements while constantly verifying results.

These seem like fundamental skills for any dev regardless of language.

## Final Reflection

AI doesn't save you from thinking, or at least you shouldn't use it that way. It's the opposite, every prompt you write is an opportunity to clarify your understanding. Every response you receive is feedback on your clarity. Every iteration is a chance to improve.

Next time you use AI and don't get the expected result, before blaming the model, ask yourself:

- Did I really have clarity on what I wanted?
- Did I decompose the problem into manageable parts?

These models are honest, literal collaborators. They give you exactly what you ask for, but they demand clarity. Learning to be clear is learning to think well. AI used properly makes you a better programmer.
