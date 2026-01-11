---
title: "VS Code: Agent Skills — capture know-how without bloating context"
date: 2026-01-11T13:00:00+01:00
description: "Understand and deploy Agent Skills (SKILL.md) in VS Code: semantic triggering, progressive disclosure, repo structure, and orchestration with custom agents + handoffs."
slug: "vscode-agent-skills"
categories: ["tooling", "engineering", "ai"]
series: "context-engineering"
tags: ["vscode", "github-copilot", "agents", "agent-skills", "context-engineering"]
draft: false
---

## TL;DR

- An LLM without context isn’t “bad”: it’s **general-purpose**. *Agent Skills* let you **inject procedural know-how** at the right moment, without drowning the context window.
- A skill = **a folder** (portable) with `SKILL.md` (YAML `name` + `description` required) + resources (scripts, templates, examples).
- Loaded **on demand** via **semantic triggering** (match on `description`) + **progressive disclosure** across 3 levels (metadata → instructions → resources).
- In VS Code 1.108, support is **experimental** and controlled via `chat.useAgentSkills`.[^vscode-agent-skills]
- Combine *Skills* (the **what**) + *Custom Agents* (the **who**) + *Handoffs* (the **workflow**) to stop “prompt-hacking”.

<!--more-->

## From “chat” to a specialized teammate (no magic)

We often expect an AI assistant to *already* know your conventions, release scripts, CI, how you diagnose bugs, PR rules… but:

- the context window is finite;
- the model is general-purpose;
- and “always-on” instructions eventually turn into noise.

**Agent Skills** target that exact problem: making a **specialized operating procedure** available **only** when it’s relevant.

## Agent Skills: an operational definition

An *Agent Skill* is a **folder** that contains:

- a **`SKILL.md`** file (required): metadata + instructions;
- optional resources (scripts, templates, examples) the agent can load if needed.

It’s an **open standard**, designed to be reusable across products (not just “a VS Code setting”). See the Agent Skills standard: https://agentskills.io/home[^standard-agent-skills]

### Minimal `SKILL.md` format

The core is YAML front matter: `name` + `description` are required.[^github-agent-skills]

```yaml
---
name: webapp-testing
description: Guide to create, run, and debug Playwright tests. Use when asking for E2E tests or diagnosing a browser test.
---
````

Then you write the procedure (Markdown): steps, expected inputs/outputs, and references to resources in the folder.

## Why it works: semantic triggering + progressive disclosure

The mechanism is simple and robust:

1. **Discovery**: Copilot only knows `name` + `description` (lightweight).
2. **Instruction loading**: if your request semantically matches the `description`, the body of `SKILL.md` is injected.
3. **Resource access**: additional files are loaded **only if** `SKILL.md` references them.

Result: you can install lots of skills without paying the context cost all the time.

## Skills vs instructions: stop mixing everything

*Custom instructions* are fine for **stable, cross-cutting** rules (style, conventions).
*Agent Skills* are for **specialized workflows** that should load **on demand**: CI debugging, release steps, incident playbooks, test generation, migrations, etc.[^vscode-agent-skills]

**A blunt heuristic**:

* if it’s “always true” → instructions;
* if it’s “true when I do X” → a skill.

## Setup in VS Code (To reproduce)

### 1) Enable support (1.108+)

In VS Code 1.108, it’s behind a flag:

* Setting: `chat.useAgentSkills`[^vscode-release-1-108]

### 2) Pick a location

VS Code detects:

* **Project skills**: `.github/skills/` (recommended) or `.claude/skills/` (legacy)
* **Personal skills**: `~/.github/skills/` (recommended) or `~/.claude/skills/` (legacy)

> Note: on GitHub Copilot CLI / coding agent you may also see `~/.copilot/skills` as the personal directory (depending on the tool).[^github-agent-skills]

### 3) Recommended repo structure

```text
.github/
  skills/
    hugo-post/
      SKILL.md
      templates/
        post.md
    ci-debug/
      SKILL.md
      checklists/
        triage.md
  agents/
    planner.agent.md
    implementer.agent.md
```

## Concrete example: 2 skills that pay off immediately

### Skill 1 — `hugo-post`: generate a consistent Hugo post (front matter + sections)

**Goal**: prevent blank-page syndrome and enforce a durable structure (TL;DR, `<!--more-->`, refs, changelog).

`./.github/skills/hugo-post/SKILL.md`

```md
---
name: hugo-post
description: Create or refactor a Hugo article (YAML front matter + TL;DR + <!--more--> + sections + refs + changelog). Use when asked for a “ready-to-commit” blog post.
---

# Hugo post (HerDev) — Procedure

## When to use
- Creating a new article
- Rewriting/normalizing an existing article
- Standardizing front matter / slug / tags

## Procedure
1. Propose a stable **slug** (kebab-case) and a sober SEO **description**.
2. Generate the skeleton from the template: `./templates/post.md`
3. Fill in:
   - TL;DR (4–6 bullets)
   - `<!--more-->` after the TL;DR
   - doc+procedure oriented sections
   - pitfalls + limits + alternatives
   - exit checklist (observable signals)
   - References (primary links)
   - Changelog

## Writing constraints
- No hype, no “magic” promises.
- Say “To reproduce” if not tested.
- Prefer “you/we” only if your editorial voice allows it; otherwise “you/one”.
```

`./.github/skills/hugo-post/templates/post.md`

```md
---
title: ""
date: ""
description: ""
slug: ""
categories: []
series: ""
tags: []
draft: true
---

## TL;DR

- 
- 
- 
- 

<!--more-->

## Context

## Procedure

## Validation

## Pitfalls and limits

## Alternatives

## Exit checklist

- [ ] 
- [ ] 

## References

- 

## Changelog

- YYYY-MM-DD: created
```

### Skill 2 — `ci-debug`: CI failure diagnosis playbook

**Goal**: enforce a routine (repro → logs → hypotheses → fix → verify).

`./.github/skills/ci-debug/SKILL.md`

```md
---
name: ci-debug
description: Diagnose a failing CI pipeline (GitHub Actions, etc.). Use when asked “why is CI failing?” or “fix the build.”
---

# CI Debug — Playbook

## Process
1. Identify the failing job/step + exact error message.
2. Classify the failure:
   - dependency/versions
   - cache
   - permissions
   - flakiness (tests)
3. Reproduce locally if possible (same versions).
4. Propose **one** minimal fix + **one** check (evidence).
5. Add a guardrail (test, assertion, version pin, doc).

## Expected output
- Probable cause (with clues)
- Minimal fix
- Validation (command / expected log)
```

## Orchestration: Skills (the what) + Custom Agents (the who) + Handoffs (the workflow)

Skills don’t replace custom agents. They **feed** them.[^vscode-custom-agents]

* A **“Planning”** agent can rely on an “ADR architecture” skill.
* An **“Implementation”** agent can load a “DB migration” skill.
* A **“Review”** agent can trigger a “security checklist” skill.

### Example handoff (plan → implementation)

Custom agents are `.agent.md` files under `.github/agents/`.[^vscode-custom-agents]

`./.github/agents/planner.agent.md`

```md
---
name: Planner
description: Produces an implementation plan in verifiable steps.
tools: ['search', 'fetch']
handoffs:
  - label: Start implementation
    agent: implementer
    prompt: Implement the plan above. Start with tests, then code, then verification.
    send: false
---
You output a numbered plan with validation criteria for each step.
```

`send: false` keeps a human in the loop. `send: true` is tempting… and risky until your pipeline is proven.

## Governance: treat skills like code

A skill is not a sticky note. It’s a **production artifact**:

* **Version** `.github/skills/` like code.
* Require **code review**: a skill can encourage running scripts.
* Avoid vague descriptions (“helps with everything”): it breaks semantic triggering.
* Keep skills small: 1 task = 1 skill. Otherwise they become “manuals” that no one loads well.

## Common pitfalls (and how to avoid them)

* **Too broad a description** → random triggering.
  *Fix*: explicitly state “when to use” + 3 typical use cases.
* **Monolithic skill** → long instructions, rarely loaded fully.
  *Fix*: split into playbook skills.
* **Unbounded scripts** → destructive commands risk.
  *Fix*: preconditions, before/after verification commands, minimal rollback, never auto-approve by default.
* **Tool divergence** (VS Code vs CLI) → different personal paths.
  *Fix*: document “where it lives” per tool (and stick to the standard + compatibility).

## Exit checklist

* [ ] `chat.useAgentSkills` enabled (VS Code 1.108+)
* [ ] At least one skill in `.github/skills/<skill>/SKILL.md`
* [ ] `name` in kebab-case; `description` includes “when to use”
* [ ] Skill contains a procedure + expected outputs
* [ ] Resources referenced via relative paths (if used)
* [ ] Security review completed (scripts, commands, access)
* [ ] (Optional) 2 custom agents + 1 plan → implementation handoff

## Sources

[^vscode-agent-skills]: Microsoft — Use Agent Skills in VS Code: [https://code.visualstudio.com/docs/copilot/customization/agent-skills](https://code.visualstudio.com/docs/copilot/customization/agent-skills).

[^vscode-release-1-108]: Microsoft — VS Code 1.108 release notes: [https://code.visualstudio.com/updates/v1_108](https://code.visualstudio.com/updates/v1_108).

[^vscode-custom-agents]: Microsoft — Custom agents in VS Code: [https://code.visualstudio.com/docs/copilot/customization/custom-agents](https://code.visualstudio.com/docs/copilot/customization/custom-agents).

[^github-agent-skills]: GitHub Docs — About Agent Skills: [https://docs.github.com/en/copilot/concepts/agents/about-agent-skills](https://docs.github.com/en/copilot/concepts/agents/about-agent-skills).

[^standard-agent-skills]: Agent Skills — Overview: [https://agentskills.io/home](https://agentskills.io/home).
