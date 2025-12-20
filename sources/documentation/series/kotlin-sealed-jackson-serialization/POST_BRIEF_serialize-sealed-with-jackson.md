Intention

Problème précis : Quand on sérialise des sealed classes Kotlin avec Jackson, Jackson ne retrouve pas toujours le bon sous-type au moment de la désérialisation, ce qui mène à des erreurs ou à la perte d'information. Ce post explique pourquoi cela arrive, comment le reproduire et quelles sont les solutions rapides et pérennes.

Résultat attendu

Le lecteur sera capable de :
- Reproduire l'échec sur un mini-projet Kotlin+Jackson
- Appliquer une solution rapide (annotations/config)
- Évaluer l'option de refactoriser (séparer DTO/domain) en fonction du contexte

Format choisi

GUIDE (pilier) : explication conceptuelle + exemples code (Gradle), tests JUnit, comparatif de solutions, recommandations.

Plan détaillé (sections)

1. Intro & cas d'usage
2. Reproduction minimale (code) — sealed class, sous-types, ObjectMapper default -> fail
3. Pourquoi ça casse ? (kotlin reflection, visibility, module kotlin) — explications techniques
4. Solutions rapides
   - jackson-module-kotlin
   - @JsonTypeInfo + @JsonSubTypes
   - registration manuelle de sous-types
5. Solution pérenne : séparer DTOs / domain — lien vers post architecture
6. Bonus pointer vers serializer custom et kotlinx.serialization
7. Checklist & tests à exécuter

Validations observables

- Tests unitaires fournis : roundtrip serialization/deserialization passe pour chaque solution
- Exemple gradle : commandes pour exécuter les tests

Pièges / limites + alternatives

- Jackson peut nécessiter kotlin-reflect pour certains cas [À vérifier] — indiquer méthode pour réduire reflexion
- Alternatives : utiliser kotlinx.serialization (avantages/inconvénients) — pointer vers le bonus

Sources officielles

- https://github.com/FasterXML/jackson-module-kotlin
- https://github.com/FasterXML/jackson-docs/wiki
- https://kotlinlang.org/docs/serialization.html [À vérifier pour details interop]
