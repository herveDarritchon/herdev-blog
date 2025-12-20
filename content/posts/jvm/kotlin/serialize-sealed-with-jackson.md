---
title: "Sérialiser les sealed classes Kotlin avec Jackson : causes, solutions, arbitrages"
date: 2025-12-19
tags: ["kotlin", "jackson", "serialization"]
series: "kotlin-sealed-jackson-serialization"
draft: false
description: "GUIDE pilier : comprendre et résoudre les problèmes de sérialisation des sealed classes Kotlin avec Jackson. Repro, solutions rapides, arbitrages, checklist."
---

> **Pour qui / Quand / Résultat / Commande**
>
> - **Pour qui** : Développeurs Kotlin/JVM (Spring Boot, Ktor, etc.) confrontés à des erreurs de (dé)sérialisation de sealed classes avec Jackson.
> - **Quand** : Dès qu’une sealed class ou hiérarchie polymorphe doit être (dé)sérialisée en JSON côté API ou persistence.
> - **Résultat** : Comprendre pourquoi ça casse, appliquer un fix rapide, arbitrer une refacto si besoin, valider par des tests reproductibles.
> - **Commande** :

```shell
./gradlew test
```

## TL;DR

Jackson ne gère pas nativement la (dé)sérialisation polymorphe des sealed classes Kotlin. Sans configuration adaptée, la désérialisation échoue ou perd le sous-type. Ce guide explique pourquoi, comment le reproduire, et propose des solutions rapides (module Kotlin, annotations, registration manuelle) ainsi qu’une alternative architecturale (séparer DTO/domain). Tests fournis pour valider chaque approche.

<!--more-->
 
## 1. Intro & cas d’usage

Les sealed classes Kotlin sont idéales pour modéliser des hiérarchies fermées (ex : résultats, événements, commandes). Mais lors de la (dé)sérialisation JSON avec Jackson, le sous-type n’est pas toujours retrouvé, menant à des erreurs ou à la perte d’information. Exemple typique :

```kotlin
sealed class Command {
	data class Create(val name: String) : Command()
	data class Delete(val id: Int) : Command()
}
```

## 2. Reproduction minimale (code)

### Projet minimal (Gradle)

{{< codesnip
path="external/herdev-labs/kotlin/jackson-and-sealed-class/build.gradle.kts"
lang="gradle"
start="12"
end="19"

>}}

`Command.kt` :

```kotlin
sealed class Command {
	data class Create(val name: String) : Command()
	data class Delete(val id: Int) : Command()
}
```

`CommandTest.kt` :

{{< codesnip
path="external/herdev-labs/kotlin/jackson-and-sealed-class/src/test/kotlin/scenario01_notyping/Scenario01_NoTypingTest.kt"
lang="kotlin"
start="1"
end="46"
 >}}

**Commande de test** :

```shell
./gradlew test --tests CommandTest
```

## 3. Pourquoi ça casse ?

- Jackson ne sait pas, sans aide, comment retrouver le sous-type lors de la désérialisation.
- Les sealed classes Kotlin ne génèrent pas de métadonnées de type utilisables par Jackson (pas de discriminant JSON).
La première étape pour améliorer la compatibilité entre Jackson et Kotlin est d’ajouter le module officiel `jackson-module-kotlin`. Ce module permet à Jackson de mieux comprendre les spécificités du bytecode Kotlin (nullabilité, data classes, sealed, etc.).

**Limite importante** : ce module ne règle pas à lui seul le problème du polymorphisme des sealed classes. Il est nécessaire mais pas suffisant si vous attendez de Jackson qu’il retrouve automatiquement le bon sous-type lors de la désérialisation.

**Quand l’utiliser ?**
- Toujours, dans tout projet Kotlin utilisant Jackson, même sans sealed classes.
- Obligatoire pour éviter des bugs subtils sur les data classes et la gestion des valeurs par défaut.
- Sans module Kotlin, Jackson ignore les particularités du bytecode Kotlin (nullabilité, sealed, etc.).
- Parfois, l’absence de `kotlin-reflect` ou une mauvaise visibilité des sous-types aggrave le problème ([À vérifier] : voir logs Jackson, activer le debug).

## 4. Solutions rapides

### a) Ajouter le module Kotlin

Pour activer la désérialisation polymorphe, il faut indiquer à Jackson comment distinguer les sous-types dans le JSON. Cela se fait via les annotations `@JsonTypeInfo` (pour choisir le discriminant) et `@JsonSubTypes` (pour lister les sous-types connus).

**Principe** :
- Jackson ajoute une propriété (ex : `type`) dans le JSON lors de la sérialisation.
- Lors de la désérialisation, il lit cette propriété pour instancier le bon sous-type.

**Avantages** :
- Solution robuste et standard, compatible avec la plupart des clients.
- Permet d’ajouter facilement de nouveaux sous-types (avec annotation).

**Limites** :
- Couplage fort entre le code et le format JSON (toute évolution de la hiérarchie nécessite de mettre à jour les annotations).
- Risque d’oubli si la hiérarchie évolue sans refactorer les annotations.

{{< codesnip
path="external/herdev-labs/kotlin/jackson-and-sealed-class/src/test/kotlin/lab/ObjectMappers.kt"
lang="kotlin"
start="10"
end="10"

>}}

### b) Annotations @JsonTypeInfo + @JsonSubTypes

```kotlin
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "type")
@JsonSubTypes(
	JsonSubTypes.Type(value = Command.Create::class, name = "create"),
	JsonSubTypes.Type(value = Command.Delete::class, name = "delete")
)
sealed class Command { ... }
```

{{< codesnip
path="external/herdev-labs/kotlin/jackson-and-sealed-class/src/test/kotlin/scenario02_property/Scenario02_PropertyTest.kt"
lang="kotlin"
start="1"
end="60"

>}}

Une alternative aux annotations consiste à enregistrer explicitement les sous-types auprès du `ObjectMapper`. Cela centralise la configuration et évite de polluer le code métier avec des annotations Jackson.

**Principe** :
- On utilise `registerSubtypes()` pour déclarer tous les sous-types connus.
- Jackson s’appuie alors sur cette liste pour la désérialisation polymorphe.

**Avantages** :
- Découplage du code métier et de la configuration Jackson.
- Plus simple à maintenir si la hiérarchie évolue souvent (une seule liste à mettre à jour).

**Limites** :
- Risque d’oubli d’un sous-type (échec silencieux ou exception à l’exécution).
- Moins explicite pour un lecteur du code (la configuration est centralisée ailleurs).

### c) Registration manuelle des sous-types

```kotlin
val mapper = jacksonObjectMapper()
mapper.registerSubtypes(Command.Create::class.java, Command.Delete::class.java)
```

{{< codesnip
path="external/herdev-labs/kotlin/jackson-and-sealed-class/src/test/kotlin/scenario03_existingprop/Scenario03_ExistingPropTest.kt"
lang="kotlin"
start="1"
end="60"

>}}

## 5. Solution pérenne : séparer DTOs / domain

Dans les architectures robustes, il est recommandé de ne pas exposer directement les sealed classes du domaine métier à la (dé)sérialisation JSON. À la place, on crée des DTOs (Data Transfer Objects) simples, adaptés à l’API, puis on effectue un mapping explicite entre ces DTOs et les classes du domaine.

**Pourquoi cette approche ?**
- Elle isole le domaine métier des contraintes techniques de la sérialisation (annotations, discriminants, etc.).
- Elle facilite l’évolution de l’API sans impacter le cœur métier.
- Elle permet de contrôler précisément le format JSON exposé (noms, structure, compatibilité).

**Quand la privilégier ?**
- Pour toute API publique ou projet long terme.
- Si la hiérarchie de sealed classes est amenée à évoluer.
- Si l’on souhaite garantir la compatibilité ascendante du contrat JSON.

**Limite** : nécessite d’écrire du code de mapping (souvent simple mais parfois fastidieux).

{{< codesnip
path="external/herdev-labs/kotlin/jackson-and-sealed-class/src/test/kotlin/scenario05_envelope/Scenario05_EnvelopeTest.kt"
lang="kotlin"
start="1"
end="60"

>}}

## 6. Bonus : serializer custom & alternatives

- Écrire un `JsonSerializer`/`JsonDeserializer` custom pour gérer le polymorphisme.
- Alternative : [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html) (interopérabilité, multiplateforme, mais nécessite adaptation si projet existant Jackson).

## 7. Checklist & tests à exécuter

{{< paperchecklist >}}

- [x] Tests roundtrip (serialize → deserialize) passent pour chaque solution
- [x] Commande de test fournie (`./gradlew test`)
- [x] Liens internes/externes vérifiés
- [x] Alternatives et pièges documentés

{{< /paperchecklist >}}

Cette checklist garantit que chaque solution proposée a été validée par un test automatisé, et que le guide reste à jour même en cas d’évolution des dépendances. Elle sert aussi de référence rapide pour vérifier la robustesse de vos propres implémentations.

## Decision tree : quelle solution choisir ?

| Critère                            | Solution rapide          | Solution pérenne (DTO) | Custom/Alternative    |
| ---------------------------------- | ------------------------ | ---------------------- | --------------------- |
| Projet existant, peu de sous-types | Annotations/registration |                        |                       |
| Projet long terme, API publique    |                          | DTO + mapping          |                       |
| Besoin multiplateforme             |                          |                        | kotlinx.serialization |
| Besoin de contrôle fin sur JSON    |                          |                        | Serializer custom     |

- **Si besoin d’un fix immédiat** : annotations ou registration.
- **Si refonte ou API publique** : séparer DTO/domain.
- **Si multiplateforme** : considérer kotlinx.serialization.

## Pièges, limites, alternatives

- **Piège** : oublier d’enregistrer un sous-type → désérialisation échoue silencieusement.
- **Limite** : maintenance des annotations/registration si la hiérarchie évolue.
- **À vérifier** : certains cas nécessitent `kotlin-reflect` (voir logs Jackson, activer le debug pour traquer les erreurs de réflexion).
- **Alternative** : [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html) — plus simple pour de nouveaux projets, mais migration coûteuse si base Jackson existante.

## Checklist de sortie

---

## Tester sur ta machine (lab reproductible)


```bash
cd external/herdev-labs/kotlin/jackson-and-sealed-class
./gradlew test
```


TODO Créer une release ZIP (voir README du lab)

Ce laboratoire contient tous les scénarios évoqués dans ce guide, sous forme de tests automatisés. Chaque solution présentée ici correspond à un ou plusieurs tests du lab, ce qui permet de vérifier concrètement le comportement de Jackson selon la configuration choisie.

**À quoi s’attendre ?**
- Les tests couvrent les cas d’échec (par défaut) et les solutions (module Kotlin, annotations, registration, DTO).
- Un test qui échoue signale un problème de configuration ou une évolution inattendue de Jackson/Kotlin.
- Le lab sert de référence pour valider vos propres évolutions ou migrations.

---

## Références officielles

- [jackson-module-kotlin (GitHub)](https://github.com/FasterXML/jackson-module-kotlin)
- [Jackson Wiki](https://github.com/FasterXML/jackson-docs/wiki)
- [Kotlin Serialization](https://kotlinlang.org/docs/serialization.html)

## Changelog

- 2025-12-19 : Création du guide pilier (première version)
- 2025-12-19 : Ajout decision tree, checklist, liens internes/externes
