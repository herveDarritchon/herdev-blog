---
title: "Kotlin ↔ Java : la null-safety, c’est un contrat (et comment Kotlin le fait respecter)"
date: 2026-01-08T21:00:00+01:00
description: "En codebase mixte, Kotlin publie la nullabilité pour Java (annotations + metadata) et la fait respecter à l’exécution (Intrinsics). Là où ça casse : les platform types côté Java→Kotlin, et comment les dompter."
slug: "kotlin-java-interop-null-safety-contract"
categories: [ "jvm", "software" ]
series: "null-safety"
tags: [ "kotlin", "java-kotlin-interop", "null-safety" ]
draft: false
---

## TL;DR

En interop Kotlin ↔ Java, la null-safety **n’est pas magique** :

- **Kotlin→Java** : le compilateur **publie** la nullabilité dans le bytecode (annotations `@NotNull/@Nullable` + `@kotlin.Metadata`), donc l’IDE *voit* le contrat.
- **Java→Kotlin** : si Java passe quand même `null` là où Kotlin exige non-null, Kotlin **fail-fast** à la frontière via `Intrinsics.checkNotNullParameter(...)`.
- **Le vrai point faible** : quand Kotlin consomme des API Java **non annotées** → types “platform” (`String!`) = nullabilité inconnue → c’est là que les bugs se glissent.

<!--more-->

# Kotlin↔Java : la null-safety, version interop (sans folklore)

En Kotlin pur, `String` vs `String?` est une règle de type : *soit c’est non-null, soit c’est nullable*.[^kotlin-null-safety]  
En Java pur, `null` reste autorisé partout (sauf discipline + annotations + outils).

En codebase mixte, la question n’est pas “Kotlin est-il safe ?” mais :

> **Comment Kotlin rend son contrat lisible par Java… et comment il se défend quand Java triche ?**

## 1) Kotlin → Java : Kotlin écrit le contrat dans le bytecode

Quand tu compiles du Kotlin, le compilateur **encode** la nullabilité dans le `.class` :

1) **`@kotlin.Metadata`** : une “carte d’identité” Kotlin que les outils peuvent lire.[^kotlin-metadata-annotation][^kotlin-metadata-jvm]  
2) **Annotations JVM de nullabilité** sur signatures (paramètres / retours) pour que l’écosystème Java comprenne : typiquement `@NotNull` / `@Nullable`.[^kotlin-java-interop]

Conséquence : IntelliJ (et d’autres outils) n’a pas besoin d’“imaginer” tes intentions.  
Il lit ce que le compilateur a écrit, directement dans le bytecode.

### Exemple mental (interop côté Java)

Tu as un Kotlin :

```kotlin
class Animal(val id: Long, var name: String) {
  fun speak(sound: String) { /* ... */ }
}
````

Côté Java, l’IDE peut te prévenir si tu fais :

```java
a.speak(null); // warning: passing null to @NotNull
```

Ce warning n’est pas un “bonus UX”. C’est une lecture du contrat publié.

## 2) Java → Kotlin : Kotlin met un portique à l’entrée (runtime guard)

Ok, mais si un dev Java ignore l’avertissement ?

Kotlin ne “prie” pas pour que ça passe. Il **bloque à l’entrée** : le compilateur injecte un check au début des fonctions/constructeurs exposés qui reçoivent un paramètre non-nullable.[^kotlin-java-interop][^kotlin-intrinsics-src]

Forme typique côté bytecode/decompilation :

```java
Intrinsics.checkNotNullParameter(sound, "sound");
```

Effet : un `NullPointerException` est levé **immédiatement**, à la frontière, avant de laisser `null` contaminer l’état interne.

> C’est une stratégie de défense : *fail-fast au point de rupture inter-langage*.

Pour le vérifier toi-même sans “croire” l’article :

* IntelliJ : *Show Kotlin Bytecode* → *Decompile*.[^idea-decompiler]

## 3) Là où ça casse vraiment : Java non annoté → Platform types (`T!`)

Le piège numéro 1 en interop n’est pas “Java passe null à Kotlin” (ça crashe net).
Le piège, c’est l’inverse :

> **Kotlin appelle une API Java non annotée. Kotlin ne peut pas savoir si c’est nullable.**

Kotlin marque alors le type comme **platform type** (`String!`) : “je ne sais pas”.[^kotlin-java-interop]
Et “je ne sais pas”, en pratique, veut dire :

* tu peux traiter la valeur comme non-null… jusqu’au jour où tu prends un NPE
* ou tu peux la traiter comme nullable… et propager de la défensive partout

C’est le point où la null-safety Kotlin devient une **illusion** si tu laisses des APIs Java “muettes”.

## 4) Règles simples (et non négociables) pour une codebase mixte

### Règle A — Annote les frontières Java publiques (sinon tu perds)

Si un module Java est consommé par Kotlin, mets de la nullabilité **sur les APIs publiques** (params + retours).
Sans ça, Kotlin bascule en `T!` et tu reviens au monde “au feeling”.

### Règle B — Côté Kotlin, traite `T!` comme “hostile”

Quand tu vois un platform type (souvent via une API Java), considère que :

* soit tu **normalises** à la frontière (`val x = javaCall() ?: return ...`)
* soit tu **wrap** (adapter Kotlin) qui re-publie un contrat clair

Ne laisse pas `T!` traverser ton domaine. C’est une fuite de contrat.

### Règle C — Préfère “adapter” plutôt que “parsemer des `!!`”

Le `!!` est un aveu : tu dis “j’accepte le crash ici”.
Parfois c’est ok, mais en interop ça devient vite un champ de mines. Mets le crash au bon endroit (frontière), pas au milieu.

### Règle D — Teste explicitement les cas `null` aux frontières

Tu veux que le crash arrive **là où tu l’attends**, avec un message exploitable.
Un test Java qui appelle Kotlin avec `null` sur un paramètre non-nullable est un test légitime : il valide que le portique est en place.

## 5) Ce que ça change (concret) dans ta manière de coder

* Kotlin te donne un contrat fort **dans ton code Kotlin**.
* En interop, ce contrat n’est fort que si tu **publies** (annotations) et **normalises** (adapters) aux frontières.
* Le bytecode + Intrinsics font le job côté “Java triche”.
* Mais côté “Java ne dit rien”, c’est à toi d’imposer le contrat (sinon `T!` gagne).

Autrement dit : **en codebase mixte, la null-safety n’est pas une feature, c’est une discipline d’architecture.**

---

## Références

[^kotlin-null-safety]: Kotlin docs — Null safety (`T` vs `T?`) : [https://kotlinlang.org/docs/null-safety.html](https://kotlinlang.org/docs/null-safety.html).

[^kotlin-java-interop]: Kotlin docs — Java interop (annotations, platform types, pièges) : [https://kotlinlang.org/docs/java-interop.html](https://kotlinlang.org/docs/java-interop.html).

[^kotlin-metadata-jvm]: Kotlin docs — Metadata JVM : [https://kotlinlang.org/docs/metadata-jvm.html](https://kotlinlang.org/docs/metadata-jvm.html).

[^kotlin-metadata-annotation]: Kotlin stdlib API — `kotlin.Metadata` : [https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-metadata/](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/-metadata/).

[^kotlin-intrinsics-src]: JetBrains/Kotlin — `Intrinsics.checkNotNullParameter` : [https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/jvm/runtime/kotlin/jvm/internal/Intrinsics.java](https://github.com/JetBrains/kotlin/blob/master/libraries/stdlib/jvm/runtime/kotlin/jvm/internal/Intrinsics.java).

[^idea-decompiler]: JetBrains — Decompiler : [https://www.jetbrains.com/help/idea/decompiler.html](https://www.jetbrains.com/help/idea/decompiler.html).
