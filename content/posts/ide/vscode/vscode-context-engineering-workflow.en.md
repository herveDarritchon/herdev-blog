---
title: "VS Code: setting up a context engineering workflow (instructions, agents, handoffs)"
date: 2026-01-11T12:00:00+01:00
description: "A durable workflow to stop “prompt hacking”: versioned project context, a planning agent (read-only), an implementation agent (editing), and guided handoffs — all inside VS Code."
slug: "vscode-context-engineering-workflow"
categories: ["tooling", "engineering", "ai"]
series: "context-engineering"
tags: ["vscode", "copilot", "context-engineering"]
draft: false
---

## TL;DR

- An AI assistant **without context** isn’t “dumb”: it’s **blind**. And your project pays the price (mistakes, busywork loops, inconsistent decisions).
- Context engineering isn’t writing better prompts: it’s **making the right context available, at the right moment**, in a **versioned** way.
- In VS Code, the most robust workflow fits in **3 building blocks**:
  1) **global instructions** (`.github/copilot-instructions.md`)  
  2) a **“Plan” agent** (read-only tools) → produces a stable plan  
  3) an **“Implementation” agent** (editing + terminal) → executes the plan  
  + **handoffs** to move from one to the other without losing the thread.

<!--more-->

## Why a “workflow” (and not just “Copilot”)?

You can get a decent result on a small ticket with a well-written prompt. But as soon as:
- the codebase grows,
- architectural rules aren’t obvious,
- the domain has traps,
- or you want consistency over multiple days/weeks,

… chat becomes a lottery. Not because the model is bad, but because **you’re feeding it unstable, incomplete, and often contradictory context**.

The typical outcome: it starts a “plausible” implementation, then you patch it by hand (or you start over). This workflow aims for the opposite: **fewer iterations, more explicit decisions, and a trail**.

> Key point: the goal isn’t to “put everything into context.” The goal is to **structure** it (stable references + on-demand selection) to avoid saturation and guesswork when the model lacks anchors.

---

## Prerequisites (to reproduce)

- VS Code with GitHub Copilot Chat.
- **Custom agents are available starting with VS Code 1.106** (if you don’t have them, update VS Code).[^vscode-version]  
  Quick check:
  ```bash
  code --version
````

* Enable instruction files for Copilot Chat:

  * Setting `github.copilot.chat.codeGeneration.useInstructionFiles` (workspace or user).[^vscode-instructions]

---

## The target workflow (clear, simple, maintainable)

### 1) “Always there” project context (global instructions)

A **versioned** file that says: “here are the rules and the reference docs.”[^vscode-instructions]

### 2) Read-only planning

An agent that **edits nothing**. It reads, understands, and **produces a plan** (a deliverable).

### 3) Implementation (“execution mode”)

An agent that’s allowed to edit, run tests, and make fixes — **but follows the plan**.

### + Handoffs

Buttons in VS Code to go from Plan → Implementation (and optionally Implementation → Review) with a pre-filled prompt.[^vscode-agents]

---

## Step 1 — Get project context under control

### 1.1 Create (or stabilize) 3 “pillar” docs

If you only maintain three documents, start with these (even if they’re short):

* `docs/ARCHITECTURE.md`

  * architecture goals, modules, dependencies, invariants, conventions
* `docs/PRODUCT.md`

  * business vocabulary, personas, rules, non-goals
* `docs/CONTRIBUTING.md`

  * style, tests, CI, commit conventions, review rules

This isn’t “nice to have.” It’s the workflow’s **fuel**.

### 1.2 Add `.github/copilot-instructions.md`

> VS Code automatically applies this file to all requests in the workspace (if the option is enabled).[^vscode-instructions]

Create:

`/.github/copilot-instructions.md`

```md
# Copilot instructions (project)

## Source of truth (read before deciding)
- Architecture: [docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md)
- Product / domain: [docs/PRODUCT.md](../docs/PRODUCT.md)
- Contribution: [docs/CONTRIBUTING.md](../docs/CONTRIBUTING.md)

## Working rules
- If information is missing: ask a question OR propose a hypothesis + a verification method.
- Do not invent APIs, files, behaviors, or command outputs.
- Prefer small, testable, reversible changes.
- Always propose a validation step (tests, lint, command, reproduction scenario).
```

💡 Useful variant: add tech-scoped instruction files via `*.instructions.md` (e.g., `backend.instructions.md`, `frontend.instructions.md`) with `applyTo:`.[^vscode-instructions]

---

## Step 2 — Create a “Plan” agent (read-only)

### 2.1 Recommended layout

```text
.github/
  agents/
    plan.agent.md
    implementation.agent.md
  prompts/
    plan.prompt.md
docs/
  ARCHITECTURE.md
  PRODUCT.md
  CONTRIBUTING.md
docs/ai/
  plan-template.md
```

### 2.2 Define the agent: `.github/agents/plan.agent.md`

Goal: produce a **actionable** and **verifiable** plan, without touching the code.

```md
---
name: Planner
description: "Generates an implementation plan (without modifying the code)."
# Intentionally limited tools: read / search / usages.
# Adapt to the actual tools available in your VS Code.
tools: ['search', 'fetch', 'usages']
handoffs:
  - label: "Start implementation"
    agent: implementation
    prompt: "Implement the plan above. Make small changes, test often, and flag any deviation from the plan."
    send: false
---

# Role
You are in **planning mode**. You make **no** code changes.

# Method
1) Clarify the request (explicit hypotheses if needed).
2) Read the reference docs (architecture/product/contrib).
3) Inspect the existing code (structure, entry points, usages).
4) Produce a Markdown plan with:
   - Goal & non-goals
   - Hypotheses / unknowns + how to resolve them
   - Steps (small, ordered, testable)
   - Impacts (API, data, perf, security)
   - Test strategy (unit/integration/contracts)
   - Minimal rollback plan
5) End with a “done” checklist (observable signals).
```

> Important: in VS Code, custom agents are discovered via `.agent.md` files under `.github/agents`. Handoffs are declared in the frontmatter.[^vscode-agents]

---

## Step 3 — Create an “Implementation” agent (editing + tests)

Here, you want an agent that:

* follows the plan,
* changes in small batches,
* runs tests,
* documents what it does.

### `.github/agents/implementation.agent.md`

```md
---
name: Implementation
description: "Implements a plan while respecting architecture and validating via tests."
# More permissive tools (editing + execution). Adjust to your environment.
tools: ['search', 'usages', 'fetch', 'terminal', 'edit']
handoffs:
  - label: "Quick review"
    agent: review
    prompt: "Review the changes: consistency, tests, risks, technical debt."
    send: false
---

# Role
You implement an existing plan. If the plan is missing or ambiguous, you stop and ask for a plan (or switch back to Planner).

# Rules
- Don’t “reinterpret” the requirement mid-flight: flag deviations from the plan.
- Small change → test → commit (or at least a checkpoint) → next step.
- If a command or behavior is uncertain: propose a verification, don’t invent.

# Expected output
- Code + tests
- Validation notes (commands, observable results)
- Decision list (and trade-offs)
```

### (Optional) Add a “Review” agent

You can create a third agent (`review.agent.md`) that edits nothing and only:

* reads the diff,
* highlights risks,
* suggests improvements,
* lists missing validations.

---

## Step 4 — Prompt files: industrialize repetitive requests (optional but worth it)

Prompt files let you run “routines” via `/` in the chat.[^vscode-prompt-files]

### Example: `.github/prompts/plan.prompt.md`

```md
---
name: plan
description: "Generate an implementation plan from a request."
agent: Planner
---

Based on the following request, produce an implementation plan compliant with the project instructions.
Request:
${input:demande:Describe the feature / refactor}
```

Usage: in VS Code chat, type `/plan` then fill in the input.[^vscode-prompt-files]

---

## Concrete example (mini user story)

> “Add a `GET /customers/{id}` endpoint with access control and tests.”

1. Select the **Planner** agent
2. Run `/plan` (or write the request)
3. Get a plan: entry points, structure, tests, rollback
4. Click the handoff **“Start implementation”**
5. The **Implementation** agent executes the plan, tests, documents
6. (Optional) Handoff to **Review**

---

## Common pitfalls (and how to avoid them)

### 1) Too many instructions kills the instruction

If your `.github/copilot-instructions.md` is 200 lines long, it’ll be ignored or misapplied. Keep it **short**, point to docs.

### 2) Outdated “pillar” docs = dangerous AI

If `ARCHITECTURE.md` lies, the agent will follow a lie. Treat these files like code: review, versioning, updates.

### 3) Tools too permissive during Plan

Planning doesn’t need editing. Limit tools (otherwise “just one small edit” happens fast).

### 4) Handoffs without guardrails

A handoff with `send: true` can auto-start implementation. If you want human control, keep `send: false`.

---

## Exit checklist (observable signals)

* [ ] `.github/copilot-instructions.md` exists and is applied (option enabled).[^vscode-instructions]
* [ ] The **Planner** agent shows up in the agent list.[^vscode-agents]
* [ ] The **Implementation** agent shows up in the agent list.[^vscode-agents]
* [ ] Planner produces a structured plan (steps + tests + rollback + checklist).
* [ ] Implementation works in small batches, runs tests, and documents.
* [ ] `ARCHITECTURE/PRODUCT/CONTRIBUTING` docs exist and are used as references.

---

## Alternatives (when to choose something else)

* **Solo, tiny scripts, low stakes**: stick to a “generalist” agent — but keep at least a minimal `copilot-instructions.md`.
* **You already have a strong ADR/PRD process**: plug Planner into those docs (don’t duplicate).
* **You don’t want to maintain agents**: VS Code offers a built-in planning agent (but you lose some standardization).

## Sources

[^vscode-version]: Microsoft — Visual Studio Code: versioning and troubleshooting (`Help: About` / `code --version`): [https://code.visualstudio.com/docs/supporting/faq#_how-do-i-find-the-version](https://code.visualstudio.com/docs/supporting/faq#_how-do-i-find-the-version).

[^vscode-instructions]: Microsoft — GitHub Copilot Chat: custom instructions and workspace instruction files (`.github/copilot-instructions.md`, `*.instructions.md`): [https://code.visualstudio.com/docs/copilot/copilot-customization#_custom-instructions](https://code.visualstudio.com/docs/copilot/copilot-customization#_custom-instructions).

[^vscode-agents]: Microsoft — GitHub Copilot Chat: agents and custom agents (`.github/agents/*.agent.md`, handoffs): [https://code.visualstudio.com/docs/copilot/copilot-chat#_agents](https://code.visualstudio.com/docs/copilot/copilot-chat#_agents).

[^vscode-prompt-files]: Microsoft — GitHub Copilot Chat: prompt files (`.github/prompts/*.prompt.md`): [https://code.visualstudio.com/docs/copilot/copilot-customization#_prompt-files](https://code.visualstudio.com/docs/copilot/copilot-customization#_prompt-files).

[^vscode-context-engineering]: Microsoft — "Set up a context engineering flow in VS Code": [https://code.visualstudio.com/docs/copilot/copilot-chat-context#_set-up-a-context-engineering-flow-in-vs-code](https://code.visualstudio.com/docs/copilot/copilot-chat-context#_set-up-a-context-engineering-flow-in-vs-code).