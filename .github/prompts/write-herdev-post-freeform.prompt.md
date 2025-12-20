---
name: write-herdev-post-freeform
description: Rédige un post HerDev durable (doc + procédure) à partir d’un brief libre.
argument-hint: "Colle un brief libre. Optionnel: format=FICHE|TUTO|GUIDE|CADRE|REX ; target_file=... ; OS/shell/versions=... ; contrainte=..."
agent: "herdev-blog-writer"
tools: ["search", "web/fetch", "web/githubRepo", "search/usages", "edit"]
---

Vous rédigez pour **HerDev Blog** à partir d’un brief libre.

## Mode
- `mode=outline` : plan (≤ 1 écran) + decision tree + validations prévues.
- `mode=draft` (défaut) : post complet prêt à coller.
- `mode=harden` : le brief contient déjà un draft ; produire :
    1) audit ([Vérifié doc]/[À vérifier]/[Hypothèse] + méthode)
    2) patch des sections concernées (uniquement les blocs modifiés)

## Source de vérité
- Invariants : `.github/copilot-instructions.md`
- Règles blog : `instructions/herdev-blog.instructions.md`

## Entrée (déduire du brief, sans sur-interroger)
- problème précis + résultat attendu
- persona + contexte + contraintes
- versions si pertinent
- format si fourni (sinon choisir)

### Si des infos essentielles manquent
Poser **1–3 questions max** et **s’arrêter**. Interdit de produire un post partiel.

## Sources
- Priorité aux sources officielles/primaires.
- StackOverflow : ok comme symptôme/exemple, jamais comme source principale.
- Info dépendante d’une version/config : marquer [À vérifier]/[Hypothèse] + méthode de validation.

## Sortie (draft)
- Bloc d’entrée obligatoire : **Pour qui / Quand / Résultat / Commande**
- TL;DR
- `<!--more-->`
- Repro + validations observables (tests/commandes/outputs)
- Decision tree + matrice trade-offs si plusieurs options
- Pièges + limites + alternatives
- Checklist
- Refs (primaires)
- Changelog

### Rendu
- si `target_file` : commencer par `# Fichier: <path>` puis contenu
- sinon : contenu complet uniquement
- créer un fichier markdown valide dans tous les cas dans le répertoire adéquat dans une sous arborescence de `content/posts`
