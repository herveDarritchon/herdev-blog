---
title: "Spring Boot + Kotlin : NoClassDefFoundError: jakarta/servlet/Filter (root cause + fix)"
date: 2025-12-20T19:10:00+01:00
slug: "kotlin-spring-no-default-package"
draft: false
series: "kotlin-spring-no-default-package"
tags:
  - "spring boot"
  - "kotlin"
description: "Erreur Spring Boot fréquente en Kotlin : warning \"@ComponentScan of the default package\" puis crash \"NoClassDefFoundError: jakarta/servlet/Filter\" (souvent sans starter web). Diagnostic immédiat : ta classe @SpringBootApplication est dans le default package. Fix : mets-la dans un vrai package racine."
---

## TL;DR (fix it in 60 seconds)

If you see:

- `** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.`
- then `NoClassDefFoundError: jakarta/servlet/Filter`

➡️ Verdict: **your `@SpringBootApplication` class is in the _default package_ (no `package ...`).**  
➡️ Fix: **declare a real root package + move the file into the matching folder** (and align your tests).

<!--more-->

---

## Quick diagnosis (copy/paste → verdict)

### Do you have one of these messages? (or several)

```text
** WARNING ** : Your ApplicationContext is unlikely to start due to a @ComponentScan of the default package.
Failed to introspect Class [org.springframework.boot.web.servlet.support.ErrorPageFilterConfiguration]
Caused by: java.lang.NoClassDefFoundError: jakarta/servlet/Filter
Error processing condition on org.springframework.boot.autoconfigure.context.PropertyPlaceholderAutoConfiguration...
````

✅ **Very likely verdict**: **your `@SpringBootApplication` class is in the default package.**

---

## Why this error is misleading (and why you’ll waste time)

The last line pushes you toward the wrong rabbit hole:

```text
NoClassDefFoundError: jakarta/servlet/Filter
```

Classic reflex: *“OK, I’ll add `spring-boot-starter-web`.”*

Wrong move in most cases.

* If you’re building a **non-web** app (CLI/batch), adding `starter-web` just hides the real issue.
* You’ll pull in Tomcat/Servlet, change startup behavior, and create unnecessary debt.

Your problem isn’t “servlet is missing”.
Your problem is: **Spring is scanning way too broadly**, because your app lives in the default package.

---

## Root cause: the default package blows up the scan scope

### The mechanism (simple)

`@SpringBootApplication` triggers, among other things, a `@ComponentScan`.

* If your main class is in `com.example.demo`, Spring scans `com.example.demo` and its subpackages.
* If your main class is in the **default package** (no `package ...`): Spring has no clear boundary → massive / unpredictable scan.

Result: Spring may start **introspecting classes from your dependencies**, including Servlet auto-config (e.g. `ErrorPageFilterConfiguration`).
And since you don’t have `jakarta.servlet` on the classpath (which is normal without web), you get:

```text
NoClassDefFoundError: jakarta/servlet/Filter
```

👉 `jakarta.servlet.Filter` is a **secondary symptom**. The root cause is the **package**.

---

## Reproducing the bug (to understand, not to suffer)

### 1) Dependencies (no web)

Example `build.gradle.kts`:

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/build.gradle.kts"
lang="gradle"

>}}

### 2) Main class in the default package (the bug)

`src/main/kotlin/DemoApplication.kt` **without** a `package ...` line:

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/src/main/kotlin/DemoApplication.kt"
lang="kotlin"

>}}

### 3) Expected outcome

* warning `@ComponentScan of the default package`
* then a crash with `NoClassDefFoundError: jakarta/servlet/Filter`

---

## The clean fix (avoid this class of side effects / avoid scan drift)

### Step 1 — Put it in a real package (mandatory)

Move the file into a coherent folder, for example:

`src/main/kotlin/fr/behaska/labs/kotlinspringnodefaultpackage/DemoApplication.kt`

And add a package declaration at the top:

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/src/main/kotlin/fr/behaska/labs/kotlinspringnodefaultpackage/DemoApplicationWithDefaultPackage.kt"
lang="kotlin"

>}}

### Step 2 — Put everything else under that package (or subpackages)

* `fr.behaska.labs.kotlinspringnodefaultpackage.controller`
* `fr.behaska.labs.kotlinspringnodefaultpackage.service`
* etc.

### Step 3 — Align your tests (or you’ll hit the other classic)

If your tests aren’t in the same package tree, you can get:

```text
Unable to find a @SpringBootConfiguration by searching packages upwards from the test
```

Example:

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/src/test/kotlin/DemoApplicationTest.kt"
lang="kotlin"

>}}

Fix: place tests in `fr.behaska.labs.kotlinspringnodefaultpackage` (or a subpackage):

`src/test/kotlin/fr/behaska/labs/kotlinspringnodefaultpackage/...`

{{< codefile
path="external/herdev-labs/kotlin/kotlin-spring-no-default-package/src/test/kotlin/fr/behaska/labs/kotlinspringnodefaultpackage/DemoApplicationTest.kt"
lang="kotlin"

>}}

---

## “Package directive does not match the file location”: normal (and you should listen to the IDE)

Kotlin (the language) allows a `package` that doesn’t match the on-disk folder.

But your IDE (IntelliJ, for example) warns you because:

* navigation/refactor will become a trap,
* you’ll look at the folder and assume “it’s in the right package” (when it isn’t),
* Spring relies on the **declared package**, not your filesystem layout.

For a real project (and even more for a blog lab): **package = folder**. Period.

---

## Debug checklist (fast, pragmatic)

### Checklist: “I want to understand in 30 seconds”

{{< paperchecklist >}}

* [ ] Does my `DemoApplication.kt` start with `package ...`?
* [ ] Does the path match the package?
  Example: `fr.behaska.labs.demo` ↔ `src/main/kotlin/fr/behaska/labs/demo/`
* [ ] Do I see the warning `@ComponentScan of the default package`?
  If yes: **stop everything and fix the package first**.
* [ ] Are my tests in the same root package (or below it)?

{{< /paperchecklist >}}

### Checklist: “I want to lock in that this is a non-web app”

If your app is CLI/batch, you can make the intent explicit:

`src/main/resources/application.properties`:

```properties
spring.main.web-application-type=none
```

This doesn’t replace the package fix, but it prevents surprises if someone adds a web dependency later.

---

## FAQ (the usual questions)

### “Why is Spring trying to load Servlet stuff if I don’t have web?”

Because your scan is out of control: it’s introspecting auto-configuration classes it should never touch in a properly packaged app.

### “Can I fix it by adding `spring-boot-starter-web`?”

You can… but you’re just hiding the issue and changing your app. You just added a web server to silence a symptom.

### “What if I really don’t want to move my packages?”

You can force the scan boundary:

```kotlin
@SpringBootApplication(scanBasePackages = ["fr.behaska.labs.kotlinspringnodefaultpackage"])
class DemoApplication
```

But it’s still a bad idea: you end up with a project that behaves differently from Spring conventions, and you’ll pay for it as friction.

---

## Sources (official / widely recognized)

Spring Boot Reference — Structuring your code / default package
[https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html](https://docs.spring.io/spring-boot/reference/using/structuring-your-code.html)

Stack Overflow — answer related to this warning (Spring Boot maintainer)
[https://stackoverflow.com/questions/28211049/spring-boot-gs-componentscan-and-classnotfoundexception-for-connectionfactory](https://stackoverflow.com/questions/28211049/spring-boot-gs-componentscan-and-classnotfoundexception-for-connectionfactory)
