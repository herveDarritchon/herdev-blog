Intention

Problème précis : Offrir une solution avancée : écrire un serializer/deserializer Jackson personnalisé pour sealed classes quand les autres solutions échouent, et présenter des alternatives (kotlinx.serialization, DTO flattening).

Résultat attendu

Le lecteur pourra implémenter un serializer personnalisé, comprendre ses limites et comparer avec kotlinx.serialization.

Format choisi

GUIDE/TUTO avancé + comparatif technique.

Plan détaillé

1. Quand prévoir un serializer custom
2. Exemple : implémentation de JsonSerializer/JsonDeserializer pour sealed + registration
3. Tests unitaires et edge cases (unknown subtype, forward compatibility)
4. Alternatives : kotlinx.serialization (exemples), flat DTOs
5. Conclusion & recommandations

Validations observables

- Tests unitaires pour custom serializer (roundtrip, unknown subtype handling)

Pièges/limites + alternatives

- Custom serializers demandent maintenance et complexité supplémentaire; préférer config/annotations avant d'y recourir.

Sources officielles

- Jackson docs (custom serializer)
- kotlinx.serialization docs
