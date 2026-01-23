---
title: "Kotlin 2.3: 5 genuinely useful changes (and what to enable right now)"
date: 2026-01-23
draft: false
description: "Kotlin 2.3.0 (2025-12-16): 5 practical changes across the language, stdlib, and multiplatform tooling — with snippets and compiler flags."
tags:
  - kotlin
  - jvm
  - multiplatform
  - kmp
  - wasm
  - tooling
---

Kotlin 2.3.0 is a **maturity** release: fewer flashy features, more guardrails, better DX, and real performance work. It won’t “change how you code” by itself — but it *will* prevent entire classes of bugs and trim boilerplate if you flip the right switches.

## TL;DR

- **Catch ignored return values** (opt-in) → fewer silent logic bugs.  
- **Explicit backing fields** (experimental) → goodbye `_state` / `state`, plus smart-cast on `field`.  
- **UUID v7 + parseOrNull** (experimental UUID API) → time-sortable IDs, parsing without exceptions.  
- **Kotlin/Native + Swift export** → more idiomatic Swift enums and `vararg`, plus release builds up to **~40%** faster.  
- **Kotlin/Wasm** → binaries up to **~13%** smaller + `KClass.qualifiedName` enabled by default thanks to compact Latin-1 string storage.

<!--more-->

---

## 1) Fewer “silent” bugs: the *unused return value checker* (opt-in)

### The problem
In Kotlin (like everywhere else), it’s easy to write an expression that produces a value… and then drop it on the floor. It shows up in `if` branches, “transforming” calls, or pipelines where you *thought* you were mutating but actually created a new value.

Kotlin 2.3.0 introduces a **checker** that can warn when an expression returns a value (other than `Unit`/`Nothing`) and that value is **not used**.

### A typical bug
A classic: a branch “builds” a string but doesn’t return it (and execution continues down a path that assumes a different shape for `name`).

```kotlin
fun formatGreeting(name: String): String {
  if (name.isBlank()) return "Hello, anonymous user!"

  if (!name.contains(' ')) {
    // Kotlin 2.3 warning (if checker enabled): unused result
    "Hello, " + name.replaceFirstChar(Char::titlecase) + "!"
  }

  val (first, last) = name.split(' ') // <- if no space, destructuring fails
  return "Hello, $first! Or should I call you Dr. $last?"
}
````

It’s not `split()` that “blows up”: it’s the **destructuring** that fails because the list doesn’t have 2 elements. With the checker enabled, the compiler points at the *real* bug: the branch that computes a value and then ignores it.

### Enable it (a realistic rollout)

The checker is **disabled by default**.

* “Progressive” mode: `check` (start here)
* “Strict” mode: `full` (only if your codebase is already disciplined, otherwise you’ll drown PRs in warnings)

Gradle (Kotlin DSL):

```kotlin
kotlin {
  compilerOptions {
    freeCompilerArgs.add("-Xreturn-value-checker=check")
  }
}
```

### Marking your APIs (the real leverage)

In `check` mode, Kotlin mostly warns on APIs that are explicitly “marked”. You can mark your own APIs with `@MustUseReturnValues` (file / class / function).

And for cases where ignoring a result is *fine* (e.g. `add()`), you can annotate with `@IgnorableReturnValue`.

### Local suppression (when ignoring is intentional)

Assign to the special underscore variable:

```kotlin
val _ = computeValue()
```

That’s a documented suppression mechanism.

---

## 2) Goodbye `_state` pattern: *Explicit Backing Fields* (experimental)

### The problem

The “public read-only property + private mutable backing” pattern is everywhere (Flow/StateFlow, `List` exposed while stored as `MutableList`, etc.). Until now, it meant maintaining two properties (and two names).

Before:

```kotlin
private val _city = MutableStateFlow("")
val city: StateFlow<String> get() = _city

fun updateCity(newCity: String) { _city.value = newCity }
```

Kotlin 2.3.0 (experimental):

```kotlin
val city: StateFlow<String>
  field = MutableStateFlow("")

fun updateCity(newCity: String) {
  // smart-cast: inside the class, city is seen as the field type
  city.value = newCity
}
```

This cuts boilerplate and removes the `_x`/`x` naming dance. The key: the compiler can **smart-cast** to the `field` type within the appropriate scope.

### Enabling it

It’s **Experimental** and requires a flag:

```kotlin
kotlin {
  compilerOptions {
    freeCompilerArgs.add("-Xexplicit-backing-fields")
  }
}
```

### When to use it (and when not to)

* ✅ Great for “mutable internally / read-only API” patterns.
* ❌ Pointless if you truly have two different properties with different semantics — keep them separate.

---

## 3) Stdlib: UUID v7 (sortable) + `parseOrNull()` (no exceptions)

The stdlib UUID API is still **experimental** (`@ExperimentalUuidApi`), but Kotlin 2.3.0 makes it notably better:

* `Uuid.parseOrNull()` (and variants): return `null` instead of throwing.
* `Uuid.generateV4()` and especially `Uuid.generateV7()`: generate UUID v7 (timestamp + randomness) that are time-sortable.
* `Uuid` implements `Comparable` (handy for ordering/sorting).

### Why v7 matters in databases

UUID v7 encodes a timestamp (ms) in its prefix, yielding IDs that are **globally orderable-ish** (great for indexes and append-heavy inserts).

### Two warnings (don’t hand-wave them)

* v7 is **partly predictable** (it contains time), so it’s not a “secret token”.
* Strict monotonicity is guaranteed **within a single process**, not across multiple machines/processes.

---

## 4) Kotlin/Native: more idiomatic Swift export + faster release builds

If you ship KMP to iOS, Kotlin 2.3.0 improves interop via **Swift export**:

* Kotlin `enum`s are exported as **native Swift enums**.
* Kotlin `vararg` maps to **Swift variadic parameters**.

Performance-wise: release tasks like `linkRelease*` (e.g. `linkReleaseFrameworkIosArm64`) can be **up to ~40% faster**, depending on project size.

⚠️ Swift export is still **Experimental** and not presented as “production-ready” in the dedicated docs.

---

## 5) Kotlin/Wasm: smaller binaries + `qualifiedName` by default

Kotlin 2.3.0 enables **fully qualified names** by default on Wasm (so `KClass.qualifiedName` works without extra config), without inflating binaries thanks to a string-storage optimization.

The important detail:

* Before: string literals stored as **UTF-16**.
* Now: literals containing only **Latin-1** are stored as **UTF-8**, shrinking metadata.

Reported outcome:

* up to **~13%** size reduction,
* and still **~8%** smaller even *with* FQNs enabled, compared to previous versions.

---

## Bonus: what becomes “Stable” in 2.3.0

Two things that matter because they’re now treated as reliable, mainstream features:

* **Nested type aliases**
* **`when` exhaustiveness checks** based on data-flow analysis

---

## A concrete action plan (no wishful thinking)

1. Upgrade to **Kotlin 2.3.0**, then enable **only** `-Xreturn-value-checker=check` first (not `full`).
2. Add `@MustUseReturnValues` to **critical APIs** (where ignoring the return is a bug), and `@IgnorableReturnValue` where ignoring is fine.
3. If your team accepts experimental features: try `-Xexplicit-backing-fields` in a “safe” module (UI/state) to kill the `_x` pattern.
4. If you store UUIDs as primary keys: evaluate `Uuid.generateV7()` (and accept the timestamp leakage).
5. If you ship Wasm: upgrade — you get tangible size/DX wins.

---

## Exit checklist (observable signals)

* [ ] The build emits “unused return value” warnings for *real* suspects (not massive noise).
* [ ] You can remove at least one `_state/state` pair via explicit backing fields (in a pilot module).
* [ ] `Uuid.parseOrNull()` replaces `try/catch` parsing in hot paths.
* [ ] In KMP iOS, exported APIs feel more idiomatic in Swift (enums/vararg) **without manual wrappers**.
* [ ] In Wasm, you measure a real artifact size reduction (at least on a non-trivial app).

[1]: https://kotlinlang.org/docs/unused-return-value-checker.html "Unused return value checker | Kotlin Documentation"
[2]: https://kotlinlang.org/docs/whatsnew23.html "What's new in Kotlin 2.3.0 | Kotlin Documentation"
[3]: https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.uuid/-uuid/?utm_source=chatgpt.com "Uuid | Core API"
[4]: https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.uuid/-uuid/-companion/generate-v7.html "generateV7 | Core API – Kotlin Programming Language"
[5]: https://kotlinlang.org/docs/native-swift-export.html "Interoperability with Swift using Swift export | Kotlin Documentation"
