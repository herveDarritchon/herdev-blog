---
title: "VS Code : de l'éditeur + chat… au poste de pilotage d'agents (et d'outils)"
date: 2025-12-15T09:00:00+01:00
slug: "vscode-cockpit-agents-mcp-models"
draft: false
categories:
  - "Tooling"
  - "AI"
  - "Development"
tutorials:
  - "VS Code"
categories_weight: 44
series:
  - "VS Code — Notes de terrain"
tags:
  - "VS Code"
  - "Copilot"
  - "AI"
  - "Development"
  - "Productivity"
tags_weight: 41
weight: 4
---

## TL;DR

- Le chat de VS Code devient une **console d’orchestration** : sessions récentes, archive, filtres, sidebar dédiée.
- Les agents “background” et “cloud” deviennent **utilisables au quotidien** grâce à l’isolation (Git worktrees) et au handoff fluide.
- NES (Next Edit Suggestions) monte d’un cran : **détection de rename + refactor via language server**.
- “Bring Your Own Key” n’est pas un gadget : c’est **la stratégie “choix de modèles”**, avec gestion du bruit (hide, auto).
- MCP passe un cap : **registry intégré**, démarrage auto, resources téléchargeables, URL elicitation, tasks, sampling… bref, l’écosystème outils se structure.

---

## Pourquoi j’en parle

La dernière release de l’année a été présentée lors d’un live “VS Code release event”, avec un message assez clair : VS Code ne veut plus être “un éditeur avec un chat”, mais **le cockpit où tu lances, suis et combines des agents, des modèles et des outils**. Et oui, la release venait juste d’être livrée la veille du live.

Ce qui m’intéresse ici, ce n’est pas l’effet waouh. C’est le changement de **workflow** : quand tu passes de “je demande un bout de code” à “je délègue des tâches, j’isole, je review, je merge”.

---

## 1) Le chat devient un hub d’agents (pas juste une conversation)

Première évolution visible : dans le panneau chat, tu as une **liste de sessions récentes** (local / background / cloud), avec **archive**, **recherche**, **filtres**, et une **agent sidebar** pour naviguer proprement. Ça règle un vrai problème : quand tu utilises plusieurs agents, tu te retrouves vite avec un bazar de fils de discussion impossibles à retrouver.

La logique sous-jacente est importante : on assume que tu vas travailler **en parallèle**, et qu’il faut donc des primitives “gestion de sessions” dignes d’un outil pro.

---

## 2) Déléguer sans se marcher dessus : background vs cloud (et l’arme secrète = worktrees)

Le point qui change le plus la donne côté dev : **l’isolation**.

- Tu peux envoyer une conversation vers le **background** ou le **cloud** via un bouton de handoff.
- Si tu démarres un background agent, tu peux choisir un **Git worktree isolé** pour que l’agent fasse ses modifs sans casser ta copie de travail.
- Et “sous le capot”, le background agent est **powered by Copilot CLI**, désormais intégré à l’éditeur.

Le cloud agent, lui, vise l’étape d’après : sur certains scénarios, il peut **créer une branche et une PR**, ce qui force naturellement un mode “review/merge” plus sain qu’un patch balancé en vrac dans ton workspace.

**Mon take (sans sucre)** : si tu laisses un agent modifier ton workspace principal sans isolation, tu vas finir par détester l’expérience. Le worktree rend ça enfin praticable.

---

## 3) NES devient “refactor-aware” grâce au language server

NES (Next Edit Suggestions) a eu droit à deux upgrades qui vont dans le bon sens :

- un **nouveau modèle de prod** pour rendre l’expérience plus solide,
- et surtout une intégration “intelligente” : si VS Code détecte que ton edit ressemble à un **rename**, il peut le **handoff au language server** pour faire un vrai rename refactor (plus précis, moins d’étapes).

Ce que je retiens : on sort progressivement du “autocomplete++” pour aller vers des actions IDE-native, ancrées dans la sémantique du code.

---

## 4) Le vrai sujet : le choix des modèles (et la fin de la liste infinie)

La démo “Bring Your Own Key” est assez transparente : l’objectif n’est pas “ajouter 200 modèles”, c’est **mettre le contrôle côté dev**.

- Nouveau flow **Manage Models**.
- Possibilité de **masquer** des modèles pour garder une liste utilisable.
- Ajout d’un mode **Auto** pour réduire la fatigue du “quel modèle je prends ?”.

### Nuance importante (à connaître avant de vendre ça en interne)

- BYOK **ne consomme pas** ton budget Copilot, mais
- tu dois **quand même être connecté** à Copilot, et certaines features “sans model picker” s’appuient sur des modèles de service en arrière-plan.

Donc : BYOK = liberté, oui. Indépendance totale, non.

---

## 5) MCP : l’écosystème outils passe un cap (registry, resources, URL elicitation, tasks, sampling)

C’est peut-être la partie la plus “future-facing”.

Déjà, VS Code expose un **MCP registry** directement dans la vue Extensions : tu tapes `@mcp`, tu vois des serveurs, tu installes, et ils peuvent être **démarrés automatiquement** au premier usage dans le chat.

Ensuite, la démo des nouveautés de spec montre des briques très concrètes :

- **icônes custom** pour identifier serveurs/outils,
- **resources** retournées par les tools, visibles et **téléchargeables**,
- **URL mode elicitation** (le serveur te demande un input via une page web),
- **tasks** pour rendre les opérations plus fiables quand le réseau est instable,
- **sampling** (le serveur déclenche une requête LLM “au nom du client”), avec un chantier en cours pour inclure l’usage d’outils dans le sampling.

**Mon take (direct)** : MCP, c’est puissant… donc ça va devenir un sujet de **gouvernance** (quels serveurs autorisés, quels droits, quelles données transitent, quelle traçabilité). Si tu n’anticipes pas ça, tu vas te prendre un mur dès que ça sort du “jouet perso”.

---

## 6) Bonus : des open-weights dans Copilot Chat via Hugging Face (extension provider)

Autre démonstration intéressante : une extension “provider” Hugging Face qui permet d’utiliser des **open-weights models** dans Copilot Chat, avec des options type “fastest” / “cheapest”, et surtout une capacité à itérer vite car l’intégration est **hors release VS Code** (extension = cadence plus rapide).

---

## Comment je traduis ça en workflow (concret)

1. **Tâches qui touchent les mêmes fichiers ?** → background agent + worktree.  
2. **Tâches longues / besoin de PR “propre” ?** → cloud agent (branche + PR).  
3. **Agents = modèles tool-capable** par défaut (sinon tu perds 50% de l’intérêt).  
4. **MCP : start small** (1–2 serveurs), puis standardise : qui a le droit d’installer quoi, et comment tu reviews ce qui est produit.  

---

## Conclusion

On est en train d’assister à un repositionnement net : VS Code veut devenir **l’orchestrateur** (sessions, agents, modèles, outils), pas juste un éditeur “augmenté”.

Le risque, c’est de tomber dans le “tout IA partout” sans discipline : sessions en pagaille, changements non isolés, modèles choisis au hasard, outils MCP branchés sans cadre.

L’opportunité, elle, est énorme si tu assumes le bon contrat : **délégation + isolation + review**. Et là, oui, tu commences à transformer l’IA en productivité réelle — pas en démo.
