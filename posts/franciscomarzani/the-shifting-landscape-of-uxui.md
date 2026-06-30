---
title: How I Built a Personal AI super-app by Wrapping Codex App Server
published: false
description: 'How Codex app-server turned a terminal AI workflow into a personal super-app with subagents, shared context, prompt improvement, and output explanations.'
tags: 'ai, codex, agents, cli'
cover_image: ./assets/codex-superapp-cover.png
---

# **The Shifting Landscape of UX/UI: An Honest Guide to AI Workflows (From Research to Real Code)**

Let’s be honest: the current anxiety surrounding AI in product design is completely justified. We’ve all seen the automated UI generators, the shifting job market, and the creeping fear that the skills we spent years perfecting are being commoditized.

If your entire value proposition as a designer is just "moving pixels," you have every reason to be worried. AI can generate a polished dashboard layout in seconds (or minutes.

However, looking at the situation realistically, panic won't change the trajectory. The survival strategy isn't to fight the automation of UI; it’s to ruthlessly reallocate our time to the things AI still sucks at: **deep human empathy, business strategy, structural logic, and critical decision-making.**

The role of the designer is shifting from a mechanical builder to a product director. To survive this shift, we need to understand exactly where the current AI ecosystem fits into our workflow—not to replace our creativity, but to absorb the repetitive labor so we can focus on the hard, strategic problems.

Here is a realistic guide on implementing these instruments across every stage of the product cycle, paired with the vital warnings you must understand before diving in.

## **1\. UX Research & Strategy (Drowning in Data)**

Before designing anything, you need to understand the user. But UX research often leads to a bottleneck: hours of audio, endless transcripts, and unstructured data that take days to sort through.

* **The Tools:** **Claude** \+ **FigJam AI**.  
* **The Reality:** AI doesn’t understand your users, but it is unmatched at sorting their text.

### **How to use it defensively:**

1. **The Core Analysis:** Do not let AI do the thinking for you. Instead, feed your user interview transcripts into **Claude** to act as a sorting assistant: *"Analyze these transcripts and extract explicit mentions of user friction during checkout. Group them by frequency."*  
2. **The Synthesis:** Drop those extracted insights into FigJam. Use **FigJam AI** to cluster messy brainstorming sticky notes into thematic groups.

🚨 **CRITICAL PRIVACY WARNING:** Never, under any circumstances, upload raw, un-redacted user transcripts containing PII (Personally Identifiable Information) or proprietary company data to public LLMs. Doing so is a fast track to a security breach, GDPR violations, and legal trouble. Always sanitize your data first: remove names, company details, specific financial figures, and use anonymized placeholders (e.g., "User\_01"). If your company blocks external LLMs entirely, you'll need to rely on locally deployed, self-hosted models.

## **2\. Design System Definition (Bridging the Code-to-Design Gap)**

A design system shouldn't just be a library of pretty components in Figma; it has to match what engineering actually deploys. Disconnects here cause endless friction and wasted development sprints.

* **The Tools:** **Figma** \+ **Claude Design** (and developer-facing tools like Claude Code).  
* **The Reality:** Standardizing design tokens is tedious. Let the machine sync them.

### **How to use it defensively:**

Advanced LLMs can inspect your engineering team's production code repositories. By utilizing Claude to analyze the existing frontend CSS/Tailwind variables, you can map your design tokens (color scales, typography, grid spacing) directly into Figma from day one.

Furthermore, when you need to document the design system for stakeholders who don’t use Figma, **Claude Design** allows you to transform your system constraints into real-world business artifacts—like live documentation, interactive style guides, or presentation decks—ensuring the company’s visual rules remain unbreakable across departments.

## **3\. Ideation & Low-Fidelity Layouts (Breaking the Blank Canvas)**

Staring at a blank canvas while trying to map out a multi-screen user flow is an energy drain. Spending hours building basic wireframes just to realize the logic is flawed is a waste of resources.

* **The Tool:** **Google Stitch** (powered by Gemini).  
* **The Reality:** It generates generic UI, but it maps out structural logic incredibly fast.

### **How to use it defensively:**

Open Stitch’s infinite canvas to engage in fast architectural exploration. Instead of manually drawing boxes for an onboarding flow, use text or real-time voice commands to wireframe concepts instantly:

*"Generate a 5-screen connected flow for a healthcare onboarding process, focusing on insurance intake and prescription management."*

Stitch will output a connected flow of screens based on Gemini. Do not accept it as the final design. Instead, treat it as a sacrificial wireframe. Use your voice to tweak the layout dynamically (*"Remove that card, make the search bar prominent"*). You get a structural baseline to stress-test with your Product Manager before committing to high-fidelity design.

## **4\. UI Design & Refinement (The Human Eye)**

This is where the automation hype usually falls short. AI can generate a generic layout, but it lacks nuance, brand identity, and the contextual understanding of micro-interactions. This is your domain.

* **The Tool:** **Figma Make**.  
* **The Reality:** It is an accelerator for the tedious parts of UI production.

### **How to use it defensively:**

Bring your structural concepts into Figma and use **Figma Make** as a high-speed production assistant:

* **Real Data:** Stop using *Lorem Ipsum*. Use the tool to instantly populate lists, names, and prices with realistic, contextual localized data.  
* **Point-and-Edit:** Select a specific component and use localized prompts for micro-tasks: *"Generate a disabled state for this button"* or *"Translate this card layout to German and automatically scale the Auto-layout padding to prevent text clipping."*

## **5\. Functional Validation & Testing (The Ultimate Handoff)**

The biggest lie in product design is that a clickable Figma prototype behaves like real software. Figma prototypes don't handle real database inputs or complex filtering. Testing on "fake" prototypes often leads to major UX blindspots discovered only after developers spend weeks coding it.

* **The Tool:** **Lovable**.  
* **The Reality:** It builds real, functional code from your design logic.

### **How to use it defensively:**

Instead of trying to wire up 200 prototype spaghetti lines in Figma to simulate a complex search filter, feed your design logic into **Lovable**. Lovable spins up a genuine web application with clean frontend code (HTML \+ Tailwind CSS), hooks up a live backend database (Supabase), and handles user authentication.

Now, put a real user in front of this live URL for your usability test. They can type actual queries, register real accounts, and break the system in real-time. The behavioral data you gather here is accurate to software reality, allowing you to catch structural UX flaws before development begins.

## **The Blind Spots: Where This Workflow Shatters**

Relying too heavily on this workflow will ruin your product. AI operates purely on patterns, which means it has massive, systemic blind spots:

* **The "Say vs. Do" Contradiction:** When synthesizing transcripts, Claude only reads what users *said*. It cannot see that a user hesitated for three minutes, sighed in frustration, or frowned while looking at a button before finally saying, *"Yeah, it looks fine."* If you don't watch the session videos yourself, you will miss the real UX insights.  
* **Flattening the Nuance:** AI synthesis tends to average out data. It strips away the emotional subtext, sarcasm, or cultural context of a user interview, giving you a sanitized, clinical summary that might miss a breakthrough opportunity.  
* **The Hallucination of Edge Cases:** Tools like Lovable and Stitch are notorious for completely ignoring edge cases (like slow network states, empty database returns, or accessibility scaling) unless you explicitly and exhaustively force them to include them.

## **The Reality Check: Cost and Access**

Let’s address the elephant in the room: **this workflow is premium, heavily gated, and expensive.** \* **Figma Make** and advanced AI features require paid enterprise or premium tier seats within Figma's ecosystem.

* **Google Stitch** is notoriously difficult to get full access to, operating on strict, gated enterprise betas or tied deeply to premium Gemini token pricing.  
* **Lovable** is brilliant, but it operates on a credit/token system. Building a complex app or running multiple iterations can quickly eat through free tiers, requiring robust monthly subscriptions that indie designers or small agencies might find hard to justify.

Before pitching this workflow to your leadership, you need to calculate the tooling overhead against the hours saved.

## **The Realistic AI UX/UI Tooling Matrix**

| Process Phase | The Tool | What It Handles (The Machine) | What You Handle (The Human) | Cost / Privacy Risk |
| :---- | :---- | :---- | :---- | :---- |
| **1\. UX Research** | Claude / FigJam AI | Transcribing, keyword tagging, sticky note clustering. | Empathy, finding the hidden narrative, strategic focus. | **High Privacy Risk**. Requires meticulous data scrubbing. |
| **2\. Design System** | Figma \+ Claude Design | Token mapping, repo syncing, documentation generation. | Defining visual constraints, brand identity, component logic. | **Paid Seats Required** for advanced repo syncing. |
| **3\. Ideation** | Google Stitch | Generating instant layout structures and multi-screen flows. | Critique, information architecture validation, flow logic. | **Gated Beta / Premium**. Limited universal access. |
| **4\. UI Design** | Figma Make | Data population, state variations, translation adjustments. | Aesthetic execution, micro-interactions, pixel refinement. | **Paid Subscription**. Native to premium Figma plans. |
| **5\. Validation** | Lovable | Writing clean code, database provisioning, deployment. | Usability observation, behavioral analysis, product iteration. | **Usage-Based Cost**. Can scale in price rapidly. |

## 

## **Conclusion: Play It Smart, Not Scared**

The worry you might be feeling about AI isn't a sign to leave the industry; it’s a clear signal to change how you work.

If you view these tools as a threat and refuse to touch them, you will eventually be outpaced by those who do. If you view them as magic solutions that will do your job for you, you will produce generic, legally vulnerable, and uninspired products that fail in the market.

The sweet spot lies in using them to **ruthlessly automate execution speed**, while keeping your human guardrails up regarding data privacy, cost control, and deep behavioral observation.

The pixel-pushing era is closing. The era of the high-leverage, strategic designer is here. It’s time to adapt the workflow—carefully.

