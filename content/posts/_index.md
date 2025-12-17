---
title: "Blog"
description: "Articles et notes de terrain : architecture, JVM, Hugo, outillage (VS Code). Objectif : comprendre, décider, maintenir."
---

Ici, tu trouveras des contenus faits pour durer : **des cadres de décision**, des **procédures reproductibles**, et des retours de terrain sur ce qui casse quand un projet grossit.

Ce n’est pas une collection d’astuces. C’est une base de connaissance : on explique **le pourquoi**, on explicite les **trade-offs**, et on te donne des critères pour choisir (ou refuser) une option.

## Explorer par thème

### Architecture logicielle
Structurer, modulariser, limiter la dette, et garder un système lisible à mesure qu’il grandit.

- séparation des responsabilités, découpage, couplage/cohésion
- décisions réversibles vs irréversibles, ADRs, signaux de dette
- refactoring guidé par contraintes (et pas par esthétique)

→ Aller à la section : [Architecture]({{< relref "posts/architecture/_index.md" >}})

### Hugo et plateforme de blog
Construire une plateforme éditoriale maintenable : i18n, arborescence, pipeline, conventions utiles (pas décoratives).

- organiser les contenus et ressources sans se tirer une balle dans le pied
- i18n propre (FR/EN), résumés, sections, taxonomies
- hygiène : templates, naming, cohérence éditoriale

→ Aller à la section : [Hugo]({{< relref "posts/hugo/_index.md" >}})

### JVM
Kotlin/Java côté pratique : contraintes réelles, intégration, outillage, et choix qui tiennent dans le temps.

- sérialisation, modèles de données, compatibilité
- build/outillage (quand ça aide, quand ça ralentit)
- choix de design orientés maintenabilité

→ Aller à la section : [JVM]({{< relref "posts/jvm/_index.md" >}})

### IDE et outillage (VS Code)
Le poste de pilotage du dev moderne : workflow, automatisation, agents/outils, et limites à poser.

- configuration qui change vraiment le quotidien (pas du tuning cosmétique)
- intégration IA “utile” (tests, revue, doc, refactor) + garde-fous
- routines : qualité, lint, CI, feedback loops

→ Aller à la section : [IDE & Tooling]({{< relref "posts/ide/_index.md" >}})

## Comment lire ce blog
- Si tu veux **aller vite**, commence par le thème qui te fait le plus mal aujourd’hui.
- Si tu veux **mettre de l’ordre**, lis d’abord *architecture* puis *outillage* (le reste suit).
- Si tu veux **industrialiser l’écriture**, passe par *Hugo* : structure + i18n + pipeline.

> Les conventions de contribution (structure des fichiers, archetypes, workflow de publication) doivent vivre dans une page dédiée “Contribuer”, pas ici.
