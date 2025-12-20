Intention

Problème précis : Montrer comment installer et configurer `jackson-module-kotlin`, quels artefacts inclure (kotlin-reflect?) et les pièges fréquents (Spring Boot auto-configuration, ObjectMapper global).

Résultat attendu

Le lecteur aura un mini-projet prêt (Gradle) avec `jackson-module-kotlin` correctement configuré et des tests qui passent.

Format choisi

FICHE / TUTO : pas trop long, instructions pas-à-pas, snippets de build.gradle(.kts) et configuration ObjectMapper.

Plan détaillé

1. Ajouter la dépendance (Gradle / Maven)
2. Exemple minimal de configuration d'ObjectMapper
3. Spring Boot : vérifications et override du ObjectMapper
4. Pièges : kotlin-reflect, module registration order
5. Tests rapides

Validations observables

- Exemple Gradle command: ./gradlew test passe
- Assertions dans tests montrant que classe sealed se (dé)serialise

Pièges/limites + alternatives

- kotlin-reflect augmente la taille ; montrer comment s'en passer si possible (réduction de reflexion) [À vérifier]

Sources officielles

- https://github.com/FasterXML/jackson-module-kotlin
- Spring Boot docs (Jackson auto-config)
