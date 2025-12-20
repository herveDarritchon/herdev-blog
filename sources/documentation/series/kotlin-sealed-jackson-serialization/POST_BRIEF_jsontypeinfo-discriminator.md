Intention

Problème précis : Expliquer comment utiliser `@JsonTypeInfo` et `@JsonSubTypes` pour fournir explicitement un discriminant de type à Jackson, et comparer les stratégies (PROPERTY vs WRAPPER_OBJECT vs EXISTING_PROPERTY).

Résultat attendu

Le lecteur saura choisir et implémenter la stratégie de discriminant adaptée à son API (compatibilité, lisibilité, contrainte JSON schema).

Format choisi

TUTO + comparaison pratique : code examples montrant chaque stratégie et leurs effets sur le payload JSON.

Plan détaillé

1. Rappel du problème
2. @JsonTypeInfo — explications des modes
3. Exemples pour PROPERTY (type) et WRAPPER_OBJECT
4. Gestion des noms de sous-types (@JsonTypeName / @JsonSubTypes)
5. Compatibilité contrats API / migration
6. Tests et validation

Validations observables

- Tests JUnit montrant JSON produit et lecture correcte

Pièges/limites + alternatives

- Certains formats (ex: clients stricts) ne veulent pas de champ "type" explicite — proposer wrapper ou field existing

Sources officielles

- Jackson annotations javadoc
