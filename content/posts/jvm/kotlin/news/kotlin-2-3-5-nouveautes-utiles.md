---
title: "Kotlin 2.3 : 5 nouveautés vraiment utiles (et quoi activer tout de suite)"
date: 2026-01-23
draft: false
description: "Kotlin 2.3.0 (16/12/2025) : 5 changements concrets côté langage, stdlib et multiplateforme. Avec snippets et options de compilation."
tags:
  - kotlin
  - jvm
  - multiplatform
  - kmp
  - wasm
  - tooling
---

Kotlin 2.3.0 est une release de **maturité** : moins de “features tape-à-l’œil”, plus de garde-fous, de DX et de perf. Elle ne “change pas votre façon de coder” *par magie* — mais elle vous évite des classes entières de bugs et réduit du boilerplate si vous activez les bons interrupteurs. :contentReference[oaicite:0]{index=0}

## TL;DR

- **Détecter les valeurs de retour ignorées** (opt-in) → moins de bugs silencieux. :contentReference[oaicite:1]{index=1}  
- **Explicit backing fields** (expérimental) → fin du duo `_state` / `state` et smart-cast sur `field`. :contentReference[oaicite:2]{index=2}  
- **UUID v7 + parseOrNull** (API UUID expérimentale) → IDs triables temporellement, parsing sans exceptions. :contentReference[oaicite:3]{index=3}  
- **Kotlin/Native + Swift export** → enums et `vararg` plus naturels côté Swift + builds release jusqu’à **~40%** plus rapides. :contentReference[oaicite:4]{index=4}  
- **Kotlin/Wasm** → binaires jusqu’à **~13%** plus légers + `KClass.qualifiedName` activé par défaut, grâce au stockage compact Latin-1. :contentReference[oaicite:5]{index=5}  

<!--more-->

---

## 1) Moins de bugs “silencieux” : le *unused return value checker* (opt-in)

### Le problème
En Kotlin (comme ailleurs), il est facile d’écrire une expression qui produit une valeur… puis de l’abandonner. Ça arrive dans des branches `if`, des appels de fonctions “transformantes”, ou des chaînes de traitement où vous pensiez muter mais vous avez en fait créé une nouvelle valeur.

Kotlin 2.3.0 introduit un **vérificateur** qui peut avertir quand une expression retourne une valeur (autre que `Unit`/`Nothing`) et que cette valeur n’est **pas utilisée**. :contentReference[oaicite:6]{index=6}

### Exemple typique
Bug classique : une branche “fabrique” une chaîne, mais ne la retourne pas (et le code continue sur un chemin qui suppose une autre forme de `name`).

```kotlin
fun formatGreeting(name: String): String {
  if (name.isBlank()) return "Hello, anonymous user!"

  if (!name.contains(' ')) {
    // Warning Kotlin 2.3 (si checker activé) : résultat ignoré
    "Hello, " + name.replaceFirstChar(Char::titlecase) + "!"
  }

  val (first, last) = name.split(' ') // <- si pas d'espace, destructuring = crash
  return "Hello, $first! Or should I call you Dr. $last?"
}
````

Ce n’est pas `split()` qui “explose” : c’est le **destructuring** qui échoue parce que la liste n’a pas 2 éléments. Le checker vous met le nez sur la branche fautive au moment de compiler.

### Activation (et stratégie réaliste)

Le checker est **désactivé par défaut**. ([Kotlin][1])

* Mode “progressif” : `check` (recommandé en premier)
* Mode “rude” : `full` (utile en codebase très disciplinée, sinon vous allez juste noyer les PR)

Gradle (Kotlin DSL) :

```kotlin
kotlin {
  compilerOptions {
    freeCompilerArgs.add("-Xreturn-value-checker=check")
  }
}
```

### Marquer vos API (le vrai levier)

En mode `check`, Kotlin remonte surtout des warnings pour les fonctions “marquées”. Vous pouvez marquer les vôtres avec `@MustUseReturnValues` (à l’échelle fichier / classe / fonction). ([Kotlin][1])

Et pour les cas où ignorer est *normal* (ex: `add()`), vous pouvez annoter avec `@IgnorableReturnValue`. ([Kotlin][1])

### Supprimer localement (quand ignorer est volontaire)

Assignation à la variable spéciale `_` :

```kotlin
val _ = computeValue()
```

C’est explicitement documenté comme mécanisme de suppression. ([Kotlin][2])

---

## 2) Fin du pattern `_state` : *Explicit Backing Fields* (expérimental)

### Le problème

Le pattern “propriété publique read-only + backing privé mutable” est omniprésent (Flow/StateFlow, listes exposées en `List` mais stockées en `MutableList`, etc.). Jusqu’ici, vous deviez entretenir deux propriétés (et deux noms).

Avant :

```kotlin
private val _city = MutableStateFlow("")
val city: StateFlow<String> get() = _city

fun updateCity(newCity: String) { _city.value = newCity }
```

Après Kotlin 2.3.0 (expérimental) :

```kotlin
val city: StateFlow<String>
  field = MutableStateFlow("")

fun updateCity(newCity: String) {
  // smart-cast : city est vu comme le type du field dans la portée privée
  city.value = newCity
}
```

Cette syntaxe **réduit le boilerplate** et vous évite la gymnastique `_x`/`x`. Le point clé : le compilateur peut faire un **smart-cast** vers le type du `field` dans la portée “privée” de la déclaration. ([Kotlin][2])

### Activation

C’est **Experimental** et ça demande un flag :

```kotlin
kotlin {
  compilerOptions {
    freeCompilerArgs.add("-Xexplicit-backing-fields")
  }
}
```

([Kotlin][2])

### Quand l’utiliser (et quand éviter)

* ✅ Très bon pour le pattern “mutable interne / API read-only”.
* ❌ Inutile si vous avez réellement deux propriétés avec deux logiques distinctes (là, gardez deux propriétés).

---

## 3) Stdlib : UUID v7 (triables) + `parseOrNull()` (sans exceptions)

L’API UUID de la stdlib reste **expérimentale** (annotation `@ExperimentalUuidApi`), mais Kotlin 2.3.0 l’améliore nettement : ([Kotlin][2])

* `Uuid.parseOrNull()` (et variantes) : parsing qui retourne `null` au lieu de throw. ([Kotlin][2])
* `Uuid.generateV4()` et surtout `Uuid.generateV7()` : génération d’UUID v7 (timestamp + aléa) triables par temps. ([Kotlin][2])
* `Uuid` est `Comparable` (utile pour trier/ordonner). ([Kotlin][3])

### Pourquoi v7 est intéressant en base

Un UUID v7 encode un timestamp (en ms) dans son préfixe, ce qui donne des identifiants **globalement ordonnables** (très utile pour les index et l’insertion “en fin”). ([Kotlin][4])

### Deux mises en garde (à ne pas ignorer)

* Le v7 est **partiellement prédictible** (il contient du temps), donc ce n’est pas un “token secret”. ([Kotlin][4])
* La monotonie stricte est garantie **dans la durée de vie d’un process**, pas entre machines/process séparés. ([Kotlin][4])

---

## 4) Kotlin/Native : Swift export plus idiomatique + builds release plus rapides

Si vous faites du KMP iOS, Kotlin 2.3.0 améliore l’interop via **Swift export** :

* Les `enum` Kotlin sont exportées en **enums Swift natifs**.
* Les fonctions `vararg` Kotlin sont mappées en **paramètres variadiques Swift**. ([Kotlin][2])

Côté perf : les tâches release `linkRelease*` (ex: `linkReleaseFrameworkIosArm64`) peuvent être **jusqu’à ~40%** plus rapides selon la taille du projet. ([Kotlin][2])

⚠️ Swift export est **Experimental** et *pas présenté comme “ready prod”* dans la doc dédiée. ([Kotlin][5])

---

## 5) Kotlin/Wasm : binaires plus légers + `qualifiedName` par défaut

Kotlin 2.3.0 active par défaut les **fully qualified names** côté Wasm (donc `KClass.qualifiedName` sans config), sans gonfler le binaire grâce à une optimisation de stockage des chaînes. ([Kotlin][2])

Le détail qui compte :

* Avant : littéraux de chaînes stockés en **UTF-16**.
* Maintenant : les littéraux ne contenant que du **Latin-1** sont stockés en **UTF-8**, ce qui réduit les métadonnées. ([Kotlin][2])

Résultat annoncé :

* Jusqu’à **~13%** de réduction de taille,
* et encore **~8%** de gain même avec les FQNs activés, comparé aux versions précédentes. ([Kotlin][2])

---

## Bonus : ce qui passe “Stable” en 2.3.0

Deux éléments qui changent la vie… parce qu’ils deviennent fiables/standard :

* **Nested type aliases** (alias de type imbriqués),
* **Vérification d’exhaustivité** des `when` basée sur l’analyse de flux de données. ([Kotlin][2])

---

## Plan d’action concret (sans se raconter d’histoires)

1. **Passez à Kotlin 2.3.0**, puis activez **uniquement** `-Xreturn-value-checker=check` en premier (pas `full`). ([Kotlin][2])
2. Ajoutez `@MustUseReturnValues` sur **vos API critiques** (celles où ignorer le retour est un bug), et `@IgnorableReturnValue` là où ignorer est normal. ([Kotlin][1])
3. Si votre équipe accepte l’expérimental : testez `-Xexplicit-backing-fields` sur un module “safe” (ex: UI/state) pour supprimer le pattern `_x`. ([Kotlin][2])
4. Si vous stockez des UUID en clé primaire : regardez `Uuid.generateV7()` (mais assumez la fuite de timestamp). ([Kotlin][4])
5. Si vous livrez du Wasm : upgrade, c’est du gain “gratuit” côté taille et introspection runtime. ([Kotlin][2])

---

## Checklist de sortie (signaux observables)

* [ ] Le build affiche des warnings “unused return value” sur des cas réellement suspects (et pas du bruit massif). ([Kotlin][1])
* [ ] Vous pouvez supprimer au moins une paire `_state/state` via explicit backing fields (sur un module pilote). ([Kotlin][2])
* [ ] `Uuid.parseOrNull()` remplace vos `try/catch` de parsing dans les zones chaudes. ([Kotlin][2])
* [ ] En KMP iOS, l’API exportée côté Swift est plus idiomatique (enums/vararg) **sans patch manuel**. ([Kotlin][2])
* [ ] En Wasm, vous constatez une baisse mesurable de la taille des artefacts (à minima sur une app réelle). ([Kotlin][2])

---

## Références

* Kotlin — *What’s new in Kotlin 2.3.0* (release 16/12/2025). ([Kotlin][2])
* Kotlin — *Unused return value checker* (modes `disable/check/full`, annotations, suppression). ([Kotlin][1])
* Kotlin Stdlib — `Uuid.generateV7()` (propriétés, monotonicité intra-process, avertissements). ([Kotlin][4])
* Kotlin — *Swift export* (statut Experimental). ([Kotlin][5])

[1]: https://kotlinlang.org/docs/unused-return-value-checker.html "Unused return value checker | Kotlin Documentation"
[2]: https://kotlinlang.org/docs/whatsnew23.html "What's new in Kotlin 2.3.0 | Kotlin Documentation"
[3]: https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.uuid/-uuid/?utm_source=chatgpt.com "Uuid | Core API"
[4]: https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.uuid/-uuid/-companion/generate-v7.html "generateV7 | Core API – Kotlin Programming Language"
[5]: https://kotlinlang.org/docs/native-swift-export.html "Interoperability with Swift using Swift export | Kotlin Documentation"
