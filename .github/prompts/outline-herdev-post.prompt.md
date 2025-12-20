---
name: outline-herdev-post
description: Passe 1 — Outline 1 écran (sections + promesse), sans rédiger le post.
argument-hint: "Brief libre (problème, résultat attendu, contrainte clé, contexte)."
agent: "herdev-blog-writer"
model: GPT-4.1
tools: ["search", "web/fetch", "web/githubRepo", "search/usages", "edit"]
---

À partir du brief libre, produis uniquement :
- le **format** choisi (FICHE/TUTO/GUIDE/CADRE/REX) + 1 phrase de justification
- un **plan** (titres H2/H3)
- pour chaque section : **promesse** (1 phrase) + **critère observable** (1 ligne)

Stop. Ne rédige pas le post.
Si info bloquante : pose 1–3 questions max puis stop.
