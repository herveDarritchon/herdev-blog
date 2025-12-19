# Workspace invariants (apply everywhere)

## 1) Exactitude (non négociable)
- Ne jamais inventer : flags, options CLI, chemins, noms de fichiers, sorties, erreurs, comportements, APIs.
- Si une info dépend d’une version / OS / shell / config :
    - marquer clairement **[Hypothèse]** et donner une méthode pour trancher (commande / check doc / test minimal).
- “Testé sur …” uniquement si versions + date + logs/outils sont fournis. Sinon : **“À reproduire”** + étapes de vérification.

## 2) Marquage des points non triviaux (docs/procédure)
Quand vous affirmez quelque chose de non évident (comportement, compat, perf, sécurité) dans une explication :
- **[Vérifié doc]** : renvoyer vers la doc primaire (ou un extrait cité par le contexte).
- **[À vérifier]** : proposer une commande / un test reproductible.
- **[Hypothèse]** : expliciter l’hypothèse + comment la valider / l’invalider.

## 3) Inputs minimum (pour éviter le flou)
Si la demande est ambiguë, demander **1 à 3 questions max** (OS, shell, versions, contexte projet, contrainte clé).
Sinon : avancer en listant des hypothèses explicites.

## 4) Commandes dangereuses
Pour toute action risquée (destructive, sécurité, prod, données) :
- préconditions
- vérif avant/après
- rollback minimal

## 5) Règle de résolution des contradictions
En cas de contradiction entre fichiers d’instructions, appliquer **la règle la plus spécifique au périmètre du fichier édité** (ex : instructions ciblées sur `content/**/*.md` > règles globales).
