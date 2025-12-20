---
title: "Sérialiser les sealed classes Kotlin avec Jackson : comprendre, corriger, et aller plus loin"
date: 2025-12-19
tags: ["kotlin", "jackson", "serialization"]
series: "kotlin-sealed-jackson-serialization"
draft: false
---

## TL;DR

Jackson ne gère pas nativement la sérialisation/désérialisation des sealed classes Kotlin. Ce post explique pourquoi, comment reproduire le bug, et propose trois solutions : configurer `jackson-module-kotlin`, utiliser les annotations `@JsonTypeInfo`/`@JsonSubTypes`, ou séparer les couches (DTO/domain). Bonus : alternatives et sérialiseur custom.

<!--more-->

---

## 1. Intro & cas d’usage

Vous avez une sealed class Kotlin :

```kotlin
sealed class Animal {
	data class Dog(val name: String) : Animal()
	data class Cat(val age: Int) : Animal()
}
```

Vous voulez la sérialiser/désérialiser avec Jackson ? Par défaut, ça casse : Jackson ne sait pas retrouver le bon sous-type.

---

## 2. Reproduction minimale

### Exemple de code

```kotlin
val mapper = ObjectMapper()
val dog = Animal.Dog("Rex")
val json = mapper.writeValueAsString(dog)
val result = mapper.readValue(json, Animal::class.java) // Erreur !
```

**Résultat attendu** : roundtrip OK  
**Résultat réel** : exception ou mauvaise instance (souvent `LinkedHashMap`).

---

## 3. Pourquoi ça casse ?

- Kotlin sealed classes : types fermés, sous-types connus à la compilation.
- Jackson : attend des annotations ou une config pour gérer le polymorphisme.
- Sans `jackson-module-kotlin`, Jackson ignore les sous-types.
- Problème aggravé si le module Kotlin n’est pas enregistré ou si la réflexion est limitée.

---

## 4. Solutions rapides

### a) Ajouter `jackson-module-kotlin`

```kotlin
val mapper = ObjectMapper()
mapper.registerModule(KotlinModule())
```

Gradle :

```groovy
implementation("com.fasterxml.jackson.module:jackson-module-kotlin:2.15.0")
```

### b) Utiliser les annotations

```kotlin
@JsonTypeInfo(use = JsonTypeInfo.Id.NAME, include = JsonTypeInfo.As.PROPERTY, property = "type")
@JsonSubTypes(
	JsonSubTypes.Type(value = Animal.Dog::class, name = "dog"),
	JsonSubTypes.Type(value = Animal.Cat::class, name = "cat")
)
sealed class Animal { ... }
```

### c) Enregistrer les sous-types manuellement

```kotlin
mapper.registerSubtypes(Animal.Dog::class.java, Animal.Cat::class.java)
```

---

## 5. Solution pérenne : séparer DTOs/domain

Plutôt que d’exposer des sealed classes dans l’API, créez des DTOs plats et des mappers.  
Avantage : plus de souci de polymorphisme, contrats stables, maintenance facilitée.

[Voir le post dédié sur la séparation des couches](/model-layer-segregation/)

---

## 6. Bonus : sérialiseur custom & alternatives

- Écrire un `JsonSerializer`/`JsonDeserializer` spécifique si besoin (voir post bonus).
- Utiliser [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html) pour une gestion native.
- Refactoriser en DTOs simples.

---

## 7. Checklist & tests à exécuter

- [ ] Ajouter `jackson-module-kotlin` dans le build
- [ ] Annoter la sealed class avec `@JsonTypeInfo`/`@JsonSubTypes`
- [ ] Vérifier la registration des sous-types
- [ ] Exécuter les tests unitaires : roundtrip OK pour chaque sous-type
- [ ] Auditer l’API : éviter d’exposer des sealed classes directement

### Exemple de test JUnit

```kotlin
@Test
fun `should serialize and deserialize sealed class`() {
	val mapper = ObjectMapper().registerModule(KotlinModule())
	val dog = Animal.Dog("Rex")
	val json = mapper.writeValueAsString(dog)
	val result = mapper.readValue(json, Animal::class.java)
	assertEquals(dog, result)
}
```

---

## Validations observables

- Les tests unitaires passent pour chaque solution.
- Le JSON produit contient le discriminant de type.
- La désérialisation retourne le bon sous-type.

---

## Pièges / limites

- Nécessité de `kotlin-reflect` dans certains cas : augmente la taille du build.
- Les annotations peuvent polluer le modèle domain.
- Les solutions custom sont plus complexes à maintenir.
- Attention à la compatibilité avec les clients stricts (champ `type`).

---

## Alternatives

- Utiliser [kotlinx.serialization](https://kotlinlang.org/docs/serialization.html) pour une gestion native.
- Refactoriser en DTOs simples.
- Sérialiseur custom (voir post bonus).

---

## Références officielles

- [Jackson module Kotlin](https://github.com/FasterXML/jackson-module-kotlin)
- [Jackson docs](https://github.com/FasterXML/jackson-docs/wiki)
- [Kotlin serialization](https://kotlinlang.org/docs/serialization.html)
- [StackOverflow: sealed class subtyping in Jackson](https://stackoverflow.com/questions/68938577/kotlin-sealed-class-subtyping-in-jackson)

---

## Changelog

- 2025-12-19 : Création du post, exemples et checklist validés.
- [À compléter lors des prochaines mises à jour]

---
