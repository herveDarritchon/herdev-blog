---
name: plan-herdev-series
description: Génère un plan de série + briefs de posts à partir d’un brief libre.
argument-hint: "Brief libre. Optionnel: dossier cible (ex: documentation/editorial/series/<slug>/)."
agent: "herdev-series-planner"
tools: ["search", "web/fetch", "web/githubRepo", "search/usages"]
---

Le brief est le texte libre fourni par l’utilisateur après la commande.

## Exigences
- Sources : officielles/primaires uniquement. Si incertain : [Vérifié doc] / [À vérifier] / [Hypothèse] + méthode.
- Objectif : produire une série cohérente, sans doublons, avec un ordre de lecture clair.
- Si info bloquante : poser 1–3 questions max puis s’arrêter.

## Sortie attendue (2 niveaux)
### A) SERIES_PLAN.md
Produire un plan structuré incluant :
- Contexte + audience
- Scope / non-scope (explicite)
- Objectif de la série
- Post “pilier” + satellites (avec ordre de lecture)
- Tableau des posts : slug / titre / format / intention / prérequis / validations / sources / liens internes
- Stratégie de tags (max 3 par post) + maillage
- Checklist de sortie de la série

### B) Un brief par post
Pour chaque post, produire un bloc `POST_BRIEF_<slug>.md` contenant :
- Intention (problème précis) + résultat attendu
- Format choisi + plan détaillé (sections)
- Validations observables
- Pièges/limites + alternatives
- Sources officielles à consulter (liens)
