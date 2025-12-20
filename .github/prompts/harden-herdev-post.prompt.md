---
name: harden-herdev-post
description: Passe 3 — Durcissement anti-invention (affirms non triviales + validations + patch minimal).
argument-hint: "Sélectionne tout le post (ou ouvre le fichier) puis lance ce prompt."
agent: "herdev-post-hardener"
model: GPT-4.1
tools: ["search", "web/fetch", "web/githubRepo", "search/usages", "edit"]
---

Analyse `${selection}` (si vide, utilise `${file}` ou demande à l’utilisateur de tout sélectionner).
Produis le rapport de durcissement au format strict défini dans l’agent.
