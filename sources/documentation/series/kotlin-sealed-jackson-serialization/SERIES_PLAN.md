Contexte & audience

Ce micro-série explique comment sérialiser correctement des sealed classes Kotlin avec Jackson, pourquoi on rencontre des problèmes de sous-typage, et les solutions pragmatiques — techniques et architecturales. Public cible : développeurs backend Kotlin utilisant Jackson (Spring Boot, Ktor, etc.), auteurs de bibliothèques, et architectes techniques qui veulent éviter les pièges de sérialisation polymorphe.

Scope (inclus)

- Expliquer les causes courantes des problèmes de sérialisation des sealed classes avec Jackson.
- Présenter les solutions pratiques (configuration, annotations, module Kotlin, custom serializer).
- Donner une alternative architecturale (séparer les couches/DTOs pour éviter la polymorphie côté sérialisation).

Non-scope (explicit)

- Ne pas couvrir en détail d'autres bibliothèques de sérialisation (sauf en alternatives brèves) ; pas de deep dive dans Protobuf/Avro.
- Ne pas réécrire la spec Jackson ; on se base sur la doc officielle et exemples reproduisibles.

Objectif de la série

Permettre au lecteur d'identifier le problème, de choisir et d'appliquer la solution la plus adaptée (rapide fix annotation/config ou refacto architectural), et de valider le résultat par des tests simples.

Post pilier & satellites (ordre de lecture recommandé)

1. Pillier — serialize-sealed-with-jackson : Vue d'ensemble + décisions & trade-offs
2. Satellite — jackson-module-kotlin-setup : Installer/configurer jackson-module-kotlin et les pièges courants
3. Satellite — jsontypeinfo-discriminator : Utiliser @JsonTypeInfo/@JsonSubTypes et options de discriminant
4. Satellite — architecture-avoid-polymorphism : Pourquoi la séparation des couches (DTO/domain) évite tout ça — lien interne
5. Bonus — bonus-custom-serializer-and-alternatives : Sérialiseur personnalisé + alternatives (kotlinx.serialization, flat DTOs)

Tableau des posts

| slug                                     | titre                                                                  | format         | intention                                                                | prérequis                                            | validations                                                          | sources                                            | liens internes            |
| ---------------------------------------- | ---------------------------------------------------------------------- | -------------- | ------------------------------------------------------------------------ | ---------------------------------------------------- | -------------------------------------------------------------------- | -------------------------------------------------- | ------------------------- |
| serialize-sealed-with-jackson            | Sérialiser les sealed classes Kotlin avec Jackson (pilier)             | GUIDE (pilier) | Expliquer le problème et présenter les solutions et choix                | Kotlin + Jackson de base, gradle/maven               | Suite de tests JUnit (serialize/deserialise roundtrip)               | Jackson docs, Kotlin docs, StackOverflow (exemple) | link vers satellites      |
| jackson-module-kotlin-setup              | Installer & configurer jackson-module-kotlin                           | FICHE/TUTO     | Montrer l'installation, pitfalls (kotlin-reflect), config d'ObjectMapper | Gradle/Maven, projet Kotlin minimal                  | Tests d'objectMapper.readValue/ writeValueAsString                   | repo jackson-module-kotlin, Spring Boot docs       | dépend du pilier          |
| jsontypeinfo-discriminator               | Utiliser @JsonTypeInfo/@JsonSubTypes et choisir un discriminant        | TUTO           | Montrer annotations, options (PROPERTY, WRAPPER_OBJECT...) et impact     | Connaissance annotations Jackson                     | Tests démontrant différentes stratégies de discriminant              | Jackson annotations javadoc                        | dépend du pilier          |
| architecture-avoid-polymorphism          | Éviter la sérialisation polymorphe par une séparation des couches      | CADRE/REX      | Présenter la stratégie architecturale (DTOs, mappers) et bénéfices       | Connaissance basique d'architecture hexagonale/clean | Checklist d'audit: find usages de sealed dans API/DTOs               | Post interne sur model-layer-segregation           | lien vers pilier et tutos |
| bonus-custom-serializer-and-alternatives | Bonus : serializer personnalisé & alternatives (kotlinx.serialization) | GUIDE/TUTO     | Fournir une solution custom et montrer alternatives (kotlinx)            | Java/Kotlin avancé                                   | Tests unitaires pour custom serializer + comparaison perf/complexité | Jackson docs, kotlinx.serialization docs           | dépend du pilier          |

Stratégie de tags & maillage

- Tags recommandés (max 3 par post) :
  - serialize-sealed-with-jackson: kotlin, jackson, serialization
  - jackson-module-kotlin-setup: kotlin, jackson, tooling
  - jsontypeinfo-discriminator: jackson, annotations, kotlin
  - architecture-avoid-polymorphism: architecture, dto, design
  - bonus-custom-serializer-and-alternatives: jackson, kotlinx-serialization, advanced

Maillage : chaque satellite renvoie au pilier; le pilier renvoie aux satellites. Le post architecture est lié depuis le pilier comme solution à plus long terme (lien interne : /model-layer-segregation/).

Checklist de sortie de la série

- [ ] Tous les posts écrits en français et relus
- [ ] Exemples de code testés localement (mini-projet Gradle) [commande fournie dans briefs]
- [ ] Liens officiels ajoutés et vérifiés
- [ ] Liens internes (pilier↔satellites) ajoutés
- [ ] Meta : tags et dates de publication renseignés
