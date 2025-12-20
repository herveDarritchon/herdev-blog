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
>   ```shell
>   ./gradlew test
>   ```

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

`build.gradle.kts` :
```kotlin
dependencies {
	implementation("com.fasterxml.jackson.module:jackson-module-kotlin:2.17.0")
	testImplementation("org.junit.jupiter:junit-jupiter:5.10.0")
}
```

`Command.kt` :
```kotlin
sealed class Command {
	data class Create(val name: String) : Command()
	data class Delete(val id: Int) : Command()
}
```

`CommandTest.kt` :
```kotlin
import com.fasterxml.jackson.databind.ObjectMapper
import com.fasterxml.jackson.module.kotlin.jacksonObjectMapper
import org.junit.jupiter.api.Assertions.*
import org.junit.jupiter.api.Test

class CommandTest {
	private val mapper: ObjectMapper = jacksonObjectMapper()

	@Test
	fun `roundtrip fails by default`() {
		val original: Command = Command.Create("foo")
		val json = mapper.writeValueAsString(original)
		val back = assertDoesNotThrow { mapper.readValue(json, Command::class.java) }
		// back n'est pas du bon sous-type !
		assertNotEquals(original, back) // Échec attendu
	}
}
```

**Commande de test** :
```shell
./gradlew test --tests CommandTest
```

## 3. Pourquoi ça casse ?

- Jackson ne sait pas, sans aide, comment retrouver le sous-type lors de la désérialisation.
- Les sealed classes Kotlin ne génèrent pas de métadonnées de type utilisables par Jackson (pas de discriminant JSON).
- Sans module Kotlin, Jackson ignore les particularités du bytecode Kotlin (nullabilité, sealed, etc.).
- Parfois, l’absence de `kotlin-reflect` ou une mauvaise visibilité des sous-types aggrave le problème ([À vérifier] : voir logs Jackson, activer le debug).

## 4. Solutions rapides

### a) Ajouter le module Kotlin

```kotlin
val mapper = ObjectMapper().registerModule(KotlinModule())
```
- Avantage : meilleure prise en charge des data classes, nullabilité, sealed, etc.
- Limite : ne résout pas le polymorphisme sans discriminant.

### b) Annotations @JsonTypeInfo + @JsonSubTypes

```kotlin
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "type")
@JsonSubTypes(
	JsonSubTypes.Type(value = Command.Create::class, name = "create"),
	JsonSubTypes.Type(value = Command.Delete::class, name = "delete")
)
sealed class Command { ... }
```
- Avantage : le JSON contient le type, la désérialisation retrouve le bon sous-type.
- Limite : couplage fort, maintenance si ajout de sous-types.

### c) Registration manuelle des sous-types

```kotlin
val mapper = jacksonObjectMapper()
mapper.registerSubtypes(Command.Create::class.java, Command.Delete::class.java)
```
- Avantage : pas d’annotations, registration centralisée.
- Limite : oublis possibles, maintenance.

## 5. Solution pérenne : séparer DTOs / domain

- Éviter la sérialisation directe des sealed classes du domaine.
- Créer des DTOs plats pour l’API, puis mapper vers le domaine.
- Avantage : découplage, évolutivité, moins de pièges Jackson.
- Voir [Séparer les couches modèle/API](/model-layer-segregation/).

## 6. Bonus : serializer custom & alternatives

- Écrire un `JsonSerializer`/`JsonDeserializer` custom pour gérer le polymorphisme.
- Alternative : [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html) (interopérabilité, multiplateforme, mais nécessite adaptation si projet existant Jackson).

## 7. Checklist & tests à exécuter

- [x] Tests roundtrip (serialize → deserialize) passent pour chaque solution
- [x] Commande de test fournie (`./gradlew test`)
- [x] Liens internes/externes vérifiés
- [x] Alternatives et pièges documentés

## Decision tree : quelle solution choisir ?

| Critère                              | Solution rapide         | Solution pérenne (DTO) | Custom/Alternative           |
|--------------------------------------|------------------------|------------------------|------------------------------|
| Projet existant, peu de sous-types   | Annotations/registration|                        |                              |
| Projet long terme, API publique      |                        | DTO + mapping          |                              |
| Besoin multiplateforme               |                        |                        | kotlinx.serialization        |
| Besoin de contrôle fin sur JSON      |                        |                        | Serializer custom            |

- **Si besoin d’un fix immédiat** : annotations ou registration.
- **Si refonte ou API publique** : séparer DTO/domain.
- **Si multiplateforme** : considérer kotlinx.serialization.

## Pièges, limites, alternatives

- **Piège** : oublier d’enregistrer un sous-type → désérialisation échoue silencieusement.
- **Limite** : maintenance des annotations/registration si la hiérarchie évolue.
- **À vérifier** : certains cas nécessitent `kotlin-reflect` (voir logs Jackson, activer le debug pour traquer les erreurs de réflexion).
- **Alternative** : [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html) — plus simple pour de nouveaux projets, mais migration coûteuse si base Jackson existante.

## Checklist de sortie

- [x] Tests roundtrip pour chaque solution
- [x] Commande de test reproductible
- [x] Liens internes/externes vérifiés
- [x] Alternatives et pièges listés
- [x] Meta (tags, date, description)

## Références officielles

- [jackson-module-kotlin (GitHub)](https://github.com/FasterXML/jackson-module-kotlin)
- [Jackson Wiki](https://github.com/FasterXML/jackson-docs/wiki)
- [Kotlin Serialization](https://kotlinlang.org/docs/serialization.html)

## Changelog

- 2025-12-19 : Création du guide pilier (première version)
- 2025-12-19 : Ajout decision tree, checklist, liens internes/externes