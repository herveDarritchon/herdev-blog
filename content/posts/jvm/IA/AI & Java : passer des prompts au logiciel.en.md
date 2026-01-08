---
title: "AI & Java: from prompts to real software — memory, tools, JSON, and Quarkus"
date: 2026-01-06
description: "What a Quarkus/LangChain4j demo really shows: stop 'chatting with an LLM' and start building a reliable application (memory, tools, structured outputs, guardrails)."
tags:
  [
    "java",
    "quarkus",
    "ai",
    "rag",
    "architecture",
    "langchain4j",
  ]
series: "ai-agents-and-java"
---

> Inspired by the vJUG CONNECT talk “AI & Java: From Structured Prompts to Smarter Apps” ([YouTube](https://www.youtube.com/watch?v=8l1obGG_6jQ)).

## TL;DR

If you take away one thing: **prompt engineering isn’t the point**. The point is **context engineering** and the **contract** between your code and the model:

- **memory** (short/long term, consistency, conflicts),
- **tools** (functions the LLM can call),
- **structured outputs** (JSON/records/enums instead of free text),
- **observability + timeouts + fallbacks** (because “works in a demo” means nothing in prod).

The talk is blunt: this isn’t magic — it’s software.

---

## 1) “The model remembers”… no. It has a context window.

A key moment in the talk resets expectations: what people often call “LLM memory” is mostly **working memory** — _the tokens currently inside the context window_ and the relationships inferred from them.

Direct consequence for us as devs: your job isn’t “finding the right prompt”. Your job is **deciding what goes into the window**, and **how**.

That’s where the talk switches to the more accurate term: **context engineering**. And context isn’t just text:

- **tool calls** (the model asks your code for facts),
- **retrieved data** (RAG),
- **constraints** (JSON-only output, implicit schemas, rules),
- **persistent memory** (application-level state, not “LLM magic”).

---

## 2) RAG + tools + agents: the slippery slope (and why you go there anyway)

The speaker gives a practical RAG reminder: in the real world (example mentioned: sensitive domains like medical records), you don’t dump everything into the model. You **retrieve the right amount of info**, at the right time, then ask the LLM to answer.

Next step is almost inevitable: once you provide **functions** (tools) to the model — “here are the actions available to answer” — you quickly slide into an **agent**:

- state,
- a loop,
- tool/API calls,
- and memory management.

And the warning is clear: you’ll keep adding **more and more tools**. It works… until you must handle:

- consistency,
- conflicts,
- errors,
- and performance.

In short: you’re back to **architecture problems**.

---

## 3) Agent memory: two dimensions that will break you in production

The talk proposes a simple, engineering-friendly framing: agent memory blows up along **two axes**.

### Axis 1 — Time

Your agent must deal with:

- **short-term** (what it must do now),
- **long-term** (what persists across sessions),
- and especially **reconciliation**.

Example: today “I like pepperoni pizza”, yesterday “I was more into burgers”. When asked again “what do I like?”, the agent must know:

- preferences are time-dependent,
- they can evolve,
- and you need an arbitration strategy.

So: **entity reconciliation** + **conflict resolution**. Not sexy. Mandatory.

### Axis 2 — Space (multi-agent)

With multiple agents/tasks in parallel:

- who writes what?
- what is the “scope of truth” for each piece of info?
- how do you sync/merge state?

Again, the speaker insists: this isn’t “AGI”. It’s **software**. The same distributed-systems questions come back: concurrency, consistency, journaling, arbitration.

---

## 4) The trap: iterating on the same bad inputs (and thinking the LLM will “eventually get it”)

A very real moment in the talk: the “horrible loop”.
You ask the model to fix something.
It answers.
You ask again.
It answers.
Repeat.

Why? Because you keep feeding it **the same inputs**, the same ambiguity, the same implicit constraints. The model won’t “fix” a spec problem.

Key takeaway (faithful paraphrase): **implementation code mixes the “what” and the “how”**. You lose intent, you lose reproducibility, and you end up tinkering.

---

## 5) The fix: structure _before_ coding (Knowledge.md, usage specs, persisted decisions)

The “spec-first” demo is one of the most actionable parts:

- an agent reads a **Knowledge.md** (context, rules, baseline),
- it detects **usage specs** (usage contracts),
- it asks questions (bootstrapping),
- and it writes a **spec** + a decisions file (e.g., “agents.md”) to persist choices.

The point isn’t the exact tooling (the talk mentions “Tessle”, Claude, and an MCP server). The point is the method:

> **A prompt is not a message. It’s a versioned artifact.**  
> And important decisions must survive the session.

That changes everything: you stop “chatting” and start **building a pipeline**.

---

## 6) On the Java side: Quarkus + LangChain4j, i.e. clean integration

The live-coding section makes the approach concrete: start with a Maven project, DI, and an “AI service” registered via annotations. The value: you move from raw model calls to a **typed service** your code can call.

### Minimal example (illustrative)

A greeter becomes “smart”, then gets rules.

```java
import dev.langchain4j.service.SystemMessage;
import dev.langchain4j.service.UserMessage;
import io.quarkiverse.langchain4j.RegisterAiService;

@RegisterAiService
public interface Greeter {
    @SystemMessage("You are a helpful greeter. Always answer in French.")
    String greet(@UserMessage String name);
}
````

The talk also shows using a **Dev UI** for quick testing, and highlights something you’ll want in prod anyway: **log requests/responses** when things go sideways (and they will).

---

## 7) Tools: when the LLM must stop guessing and call your code

A simple but effective demo: “give me the current time”.
Sure, your code could just call `LocalTime.now()`. But the architectural point is: you give the model a **verifiable capability** via a tool.

And there’s a classic gotcha shown live: forget the tool annotation… and nothing works. You’ll waste time “debugging the AI” when you simply missed an annotation.

```java
import dev.langchain4j.agent.tool.Tool;
import jakarta.enterprise.context.ApplicationScoped;
import java.time.LocalTime;

@ApplicationScoped
public class TimeTools {

    @Tool("Return the current local time")
    public String time() {
        return LocalTime.now().toString();
    }
}
```

Once you have tools, you’re effectively giving the model a toolbox — and you’re back to the real topic: **context engineering**.

---

## 8) Structured outputs: ditch free text, ship JSON (records, enums)

This is the core of “From Structured Prompts to Smarter Apps”: as long as you accept free text, you can do a demo. In production, you’ll suffer.

The talk illustrates the switch: instead of returning `String`, return an **object** (record) with an **enum**, and force a machine-usable JSON output.

```java
public enum PetType { DOG, CAT, FISH, BIRD }

public record Pet(String name, PetType type) {}
```

And on the AI service side:

```java
@RegisterAiService
public interface PetService {
    @SystemMessage("Return only JSON.")
    Pet suggestPet(@UserMessage String constraints);
}
```

The speaker stresses you can add **guardrails** (JSON-only checks, validation, etc.). And you must add **timeouts** and **fallbacks** — because an app waiting forever for a “smart” answer is a dead app.

---

## 9) Front + back: integration matters (Quinoa) and “media type” will bite you

A great live moment: an endpoint returns “something not JSON”, it breaks, and someone in the room points out… the **media type**.

It’s basic, and that’s the point: once you plug an LLM into an app, physics still applies. You must control:

* endpoints,
* contracts,
* formats,
* front/back integration.

That’s why an integration framework (here: Quarkus + a front extension like Quinoa) matters — it reduces gray zones.

---

## 10) Architecture checklist: stop playing, start shipping

If you want to turn an LLM demo into a real system, this is the bare minimum:

### Contract

* **Structured outputs** (JSON/records/enums), not “parseable text”.
* “JSON only” + strict validation + explicit error handling.

### Context

* RAG: *retrieve the right amount*, not “the whole database”.
* Tools: prefer calling your code over hallucination.
* Versioned context: Knowledge.md / usage specs / persisted decisions.

### Memory

* Separate short-term vs long-term.
* Plan for reconciliation, conflicts, time-awareness.
* In multi-agent setups: scopes of truth + merge strategy.

### Resilience

* **Timeouts** everywhere.
* **Fallbacks** (acceptable degradation > total failure).
* Observability: req/rsp logs, tracing, latency metrics, failure rates.

### Security

* Don’t connect an LLM directly to sensitive systems.
* Govern tools (permissions, filtering, audit).

---

## Conclusion

This talk does something rare: it puts AI back in its place.
An LLM is powerful… but **it doesn’t replace architecture**. It just makes architecture visible earlier — and more brutally.

So if you want “smarter apps” in Java:

* stop chasing “better prompts”,
* start designing **contracts**, **memory**, **tools**, and **guardrails**,
* and treat your AI system like any distributed system: **observable, resilient, testable**.
