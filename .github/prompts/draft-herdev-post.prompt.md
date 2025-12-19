---
name: draft-herdev-post
description: Passe 2 — Génère le post complet (DONE + meta) à partir d’un brief (et/ou d’un outline).
argument-hint: "Brief libre (+ colle l’outline si tu l’as)."
agent: "herdev-blog-writer"
model: GPT-4.1
tools: ["search", "web/fetch", "web/githubRepo", "search/usages"]
---

Rédige le post complet en Markdown selon HerDev.
- Inclure TL;DR, <!--more-->, refs officielles, changelog.
- Sections DONE (validation, pièges/limites, alternatives, checklist de sortie).
- Ne pas inventer : baliser [Vérifié doc]/[À vérifier]/[Hypothèse] dès qu’un point est non trivial.
  Si info bloquante : 1–3 questions max puis stop.
