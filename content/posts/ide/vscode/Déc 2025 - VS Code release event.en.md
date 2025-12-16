---
title: "VS Code : de l'éditeur + chat0 au poste de pilotage d'agents (et d'outils)"
date: 2025-12-15T09:00:00+01:00
slug: "vscode-cockpit-agents-mcp-models"
draft: false
categories:
  - "Tooling"
  - "AI"
  - "Development"
tutorials:
  - "VS Code"
categories_weight: 44
series:
  - "VS Code - Notes de terrain"
tags:
  - "VS Code"
  - "Copilot"
  - "AI"
  - "Development"
  - "Productivity"
tags_weight: 41
weight: 4
---

## TL;DR

- VS Code chat is evolving into an orchestration console: recent sessions, archive, filters, and a dedicated agent sidebar.
- Background and cloud agents become usable day-to-day thanks to isolation (Git worktrees) and smooth handoffs.
- NES (Next Edit Suggestions) improves with rename/refactor detection via the language server.
- "Bring Your Own Key" focuses on model choice and managing noise (hide, auto-select).
- MCP reaches maturity: integrated registry, auto-start, downloadable resources, URL elicitation, tasks, sampling — the tools ecosystem is becoming structured.

---

## Why this matters

The recent release presented at a "VS Code release event" shows a shift: VS Code aims to be more than an editor with chat — it wants to be the cockpit where you run, track and combine agents, models and tools. The important change is the workflow: from "ask for code" to "delegate tasks, isolate work, review and merge".

---

## 1) Chat becomes an agent hub (not just conversation)

The chat panel now shows a list of recent sessions (local/background/cloud), with archive, search, filters and an agent sidebar. This addresses the real problem of managing multiple agent conversations and finding the right context.

The design assumes parallel work and provides session management primitives for a professional workflow.

---

## 2) Delegate safely: background vs cloud (worktrees are the secret)

Isolation is the key technical improvement.

- You can hand off a conversation to background or cloud agents.
- A background agent can operate in a dedicated Git worktree so its changes don’t contaminate your main working copy.
- Under the hood, background agents are powered by Copilot CLI integrated with the editor.

Cloud agents are intended for longer workflows: they can create a branch and open a PR, which enforces review and reduces noisy changes in the main workspace.

Practical takeaway: always use an isolated worktree when you let an agent modify code automatically.

---

## 3) NES becomes refactor-aware via the language server

NES (Next Edit Suggestions) now integrates with language servers: if an edit looks like a rename, the system can hand off the operation to the language server for a proper rename/refactor. This reduces false positives and makes generated edits safer.

The trend is clear: move from autocomplete-like behavior to IDE-native actions that respect code semantics.

---

## 4) Model choice and noise reduction

The "Bring Your Own Key" demo highlights model management rather than adding more models. Key features:

- Manage Models flow
- Ability to hide or pin models to reduce noise
- An Auto mode to pick models when you don’t want to choose manually

Note: BYOK does not mean total independence — some features still rely on service-side models and you may need to stay connected to Copilot for certain flows.

---

## 5) MCP: a structured tools ecosystem

MCP introduces a registry view (search `@mcp`) in Extensions. You can install MCP servers, which can be auto-started and offer icons, downloadable resources, URL-based elicitation, tasks, and sampling. This is powerful but raises governance questions: which servers are allowed, what rights do they have, and how do you audit their outputs?

Recommendation: start small (1–2 servers), define policy for allowed MCP servers and review outputs produced by tools.

---

## 6) Bonus: open-weights via Hugging Face provider

An extension provider demo showed integration with Hugging Face to use open-weights models in Copilot Chat with options like "fastest" or "cheapest". Because this is an extension, it can evolve faster than the core VS Code release.

---

## Practical workflow suggestions

1. Use background agents + worktrees for changes that touch the same files.
2. Use cloud agents (branch + PR) for longer tasks that need review.
3. Prefer model+tool-capable agents by default.
4. For MCP, start with a small registry and define governance for installing and running servers.

---

## Conclusion

VS Code is positioning itself as an orchestrator for sessions, agents, models and tools, not just an editor with AI features. The opportunity is real if you adopt delegation, isolation and code review as the contract around agent usage; otherwise the risk is chaotic sessions, unreviewed changes and uncontrolled tool access.
