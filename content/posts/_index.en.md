---
title: "Blog"
description: "Articles and field notes on software architecture, JVM, Hugo, and tooling (VS Code). Goal: understand, decide, maintain."
---

This is a long-lived knowledge base: **decision frameworks**, **repeatable procedures**, and real-world notes on what actually breaks when a project grows.

No hype, no “10 tips”. We focus on **how things work**, the **trade-offs**, and the criteria that help you choose (or intentionally reject) an option.

## Browse by theme

### Software architecture
Structure, modularize, control technical debt, and keep a system readable as it scales.

- separation of concerns, boundaries, coupling/cohesion
- reversible vs irreversible decisions, ADRs, debt signals
- constraint-driven refactoring (not aesthetic refactoring)

→ Go to: [Architecture]({{< relref "posts/architecture/_index.md" >}})

### Hugo and blogging platform
Build a maintainable content platform: i18n, structure, pipeline, and conventions that actually help.

- organize content/resources without future pain
- clean i18n (EN/FR), summaries, sections, taxonomies
- hygiene: templates, naming, editorial consistency

→ Go to: [Hugo]({{< relref "posts/hugo/_index.md" >}})

### JVM
Practical Kotlin/Java: real constraints, integration, tooling, and choices that still make sense in two years.

- serialization, data modeling, compatibility
- build/tooling (when it helps, when it slows you down)
- maintainability-oriented design decisions

→ Go to: [JVM]({{< relref "posts/jvm/_index.md" >}})

### IDE and tooling (VS Code)
Your day-to-day cockpit: workflow, automation, agents/tools — and the boundaries you must enforce.

- configurations that genuinely change your daily work
- “useful” AI for engineering (tests, review, docs, refactor) + guardrails
- routines: quality gates, linting, CI, feedback loops

→ Go to: [IDE & Tooling]({{< relref "posts/ide/_index.md" >}})

## How to read this blog
- If you want speed: start with the theme that hurts right now.
- If you want order: read *architecture* first, then *tooling* (the rest follows).
- If you want a durable publishing setup: start with *Hugo* (structure + i18n + pipeline).

> Contribution conventions (file structure, archetypes, publishing workflow) belong in a dedicated “Contributing” page — not here.
