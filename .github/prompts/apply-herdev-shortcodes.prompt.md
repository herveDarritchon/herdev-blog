---
name: apply-herdev-shortcodes
description: "Remplace les blocs de code du post par des shortcodes Hugo pointant vers external/herdev-labs (codesnip en mode range start/end)."
argument-hint: "Recommandé: sélectionner tout le post"
agent: herdev-post-shortcodes-integrator
---

Référence : [HOWTO](../../documentation/HOWTO.md)

Post à traiter :

## Sélection (prioritaire)
${selection}

## Fallback (UTILISER UNIQUEMENT si la sélection est vide)
${file}

Contraintes supplémentaires (spécifiques à ce prompt) :
- Utiliser `codesnip` **en mode range** (`start/end`) et ne pas utiliser `name=`.
- Ne pas modifier le code du lab pour ajouter des marqueurs.
