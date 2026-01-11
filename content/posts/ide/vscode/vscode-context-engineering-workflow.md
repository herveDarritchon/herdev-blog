---
title: "VS Code : mettre en place un workflow d’ingénierie de contexte (instructions, agents, handoffs)"
date: 2026-01-11T12:00:00+01:00
description: "Un workflow durable pour arrêter de « prompt-hacker » : contexte projet versionné, agent de planification (read-only), agent d’implémentation (édition), et handoffs guidés — le tout dans VS Code."
slug: "vscode-workflow-ingenierie-de-contexte"
categories: ["tooling", "engineering", "ai"]
series: "context-engineering"
tags: ["vscode", "copilot", "context-engineering"]
draft: false
---

## TL;DR

- Un assistant IA **sans contexte** n’est pas “bête” : il est **aveugle**. Et votre projet paie la facture (erreurs, tours de manège, décisions incohérentes).
- L’ingénierie de contexte, ce n’est pas écrire de meilleurs prompts : c’est **rendre le bon contexte disponible, au bon moment**, de façon **versionnée**.
- Dans VS Code, le workflow le plus robuste tient en **3 briques** :
  1) **instructions globales** (`.github/copilot-instructions.md`)  
  2) **agent “Plan”** (outils lecture seule) → produit un plan stable  
  3) **agent “Implementation”** (édition + terminal) → exécute le plan  
  + des **handoffs** pour passer de l’un à l’autre sans perdre le fil.

<!--more-->

## Pourquoi un “flux” (et pas juste “Copilot”) ?

Vous pouvez obtenir un résultat correct sur un petit ticket avec un prompt bien écrit. Mais dès que :
- la base de code grossit,
- les règles d’architecture ne sont pas évidentes,
- le domaine a ses pièges,
- ou vous voulez de la cohérence sur plusieurs jours/semaines,

… le “chat” devient une loterie. Pas parce que le modèle est mauvais, mais parce que **vous lui donnez un contexte instable, incomplet, et souvent contradictoire**.

La conséquence typique : ça part sur une implémentation “plausible”, puis vous rattrapez à la main (ou vous repartez de zéro). Ce flux vise l’inverse : **moins d’itérations, plus de décisions explicites, et une trace**.

> Point clé : on ne cherche pas à “tout mettre dans le contexte”. On cherche à **structurer** (références stables + sélection à la demande) pour éviter la saturation et les hallucinations par manque de repères.

---

## Prérequis (à reproduire)

- VS Code avec GitHub Copilot Chat.
- **Custom agents disponibles à partir de VS Code 1.106** (si vous ne les avez pas, mettez VS Code à jour).[^vscode-version]
  Vérif rapide :
  ```bash
  code --version
  ```

* Activer l’usage des fichiers d’instructions pour Copilot Chat :

    * Setting `github.copilot.chat.codeGeneration.useInstructionFiles` (workspace ou user).[^vscode-instructions]

---

## Le workflow cible (clair, simple, maintenable)

### 1) Contexte projet “toujours là” (instructions globales)

Un fichier **versionné** qui dit : “voici les règles et les docs de référence”.[^vscode-instructions]

### 2) Planification “read-only”

Un agent qui **n’édite rien**. Il lit, il comprend, il **produit un plan** (livrable).

### 3) Implémentation “exécution”

Un agent qui a le droit d’éditer, de lancer les tests, de faire les corrections — **mais qui suit le plan**.

### + Handoffs

Des boutons dans VS Code pour passer Plan → Implémentation (et éventuellement Implémentation → Review) avec un prompt pré-rempli.[^vscode-agents]

---

## Étape 1 — Mettre le contexte projet sous contrôle

### 1.1 Créer (ou stabiliser) 3 docs “piliers”

Si vous ne devez maintenir que trois documents, commencez par ceux-là (même s’ils sont courts) :

* `docs/ARCHITECTURE.md`

    * objectifs d’archi, modules, dépendances, invariants, conventions
* `docs/PRODUCT.md`

    * vocabulaire métier, personas, règles de gestion, non-objectifs
* `docs/CONTRIBUTING.md`

    * style, tests, CI, conventions de commit, règles de review

Ce n’est pas du “nice to have”. C’est **le carburant** du flux.

### 1.2 Ajouter `.github/copilot-instructions.md`

> VS Code applique automatiquement ce fichier à toutes les demandes dans le workspace (si l’option est activée).[^vscode-instructions]

Créez :

`/.github/copilot-instructions.md`

```md
# Instructions Copilot (projet)

## Source de vérité (à lire avant de décider)
- Architecture : [docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md)
- Produit / métier : [docs/PRODUCT.md](../docs/PRODUCT.md)
- Contribution : [docs/CONTRIBUTING.md](../docs/CONTRIBUTING.md)

## Règles de travail
- Si une information manque : poser une question OU proposer une hypothèse + méthode de vérification.
- Ne pas inventer d’API, de fichiers, de comportements, ni de sorties de commande.
- Privilégier des changements petits, testables, et réversibles.
- Toujours proposer une validation (tests, lint, commande, scénario de reproduction).
```

💡 Variante utile : ajouter des instructions ciblées par techno via `*.instructions.md` (ex : `backend.instructions.md`, `frontend.instructions.md`) avec `applyTo:`.[^vscode-instructions]

---

## Étape 2 — Créer un agent “Plan” (lecture seule)

### 2.1 Arborescence recommandée

```text
.github/
  agents/
    plan.agent.md
    implementation.agent.md
  prompts/
    plan.prompt.md
docs/
  ARCHITECTURE.md
  PRODUCT.md
  CONTRIBUTING.md
docs/ai/
  plan-template.md
```

### 2.2 Définir l’agent : `.github/agents/plan.agent.md`

Objectif : produire un plan **actionnable** et **vérifiable**, sans toucher au code.

```md
---
name: Planner
description: "Génère un plan d’implémentation (sans modifier le code)."
# Outils volontairement limités : lecture / recherche / usages.
# Adaptez selon les outils réellement disponibles dans votre VS Code.
tools: ['search', 'fetch', 'usages']
handoffs:
  - label: "Démarrer l’implémentation"
    agent: implementation
    prompt: "Implémente le plan ci-dessus. Fais des changements petits, teste souvent, et signale tout écart au plan."
    send: false
---

# Rôle
Vous êtes en **mode planification**. Vous ne faites **aucune** modification de code.

# Méthode
1) Clarifier le besoin (hypothèses explicites si nécessaire).
2) Lire les docs de référence (architecture/produit/contrib).
3) Inspecter le code existant (structure, points d’entrée, usages).
4) Produire un plan en Markdown avec :
   - Objectif & non-objectifs
   - Hypothèses / inconnues + comment trancher
   - Étapes (petites, ordonnées, testables)
   - Impacts (API, données, perf, sécurité)
   - Stratégie de tests (unit/intégration/contrats)
   - Plan de rollback minimal
5) Finir par une checklist “sortie” (signaux observables).
```

> Important : dans VS Code, les agents custom sont détectés via des fichiers `.agent.md` dans `.github/agents`. Les *handoffs* se déclarent dans le frontmatter.[^vscode-agents]

---

## Étape 3 — Créer un agent “Implementation” (édition + tests)

Ici, on veut un agent qui :

* suit le plan,
* modifie en petits lots,
* lance les tests,
* documente ce qu’il fait.

### `.github/agents/implementation.agent.md`

```md
---
name: Implementation
description: "Implémente un plan en respectant l’architecture et en validant par les tests."
# Outils plus permissifs (édition + exécution). Ajustez à votre environnement.
tools: ['search', 'usages', 'fetch', 'terminal', 'edit']
handoffs:
  - label: "Revue rapide"
    agent: review
    prompt: "Passe en revue les changements : cohérence, tests, risques, dette technique."
    send: false
---

# Rôle
Vous implémentez un plan existant. Si le plan est absent ou ambigu, vous stoppez et demandez un plan (ou vous repassez sur Planner).

# Règles
- Ne pas “réinterpréter” le besoin en cours de route : signaler les écarts au plan.
- Changement petit → test → commit (ou au minimum checkpoint) → suite.
- Si une commande ou un comportement n’est pas certain : proposer une vérif, ne pas inventer.

# Sortie attendue
- Code + tests
- Notes de validation (commandes, résultats observables)
- Liste des décisions (et trade-offs)
```

### (Optionnel) Ajouter un agent “Review”

Vous pouvez créer un troisième agent (`review.agent.md`) qui n’édite rien et ne fait que :

* lire le diff,
* relever risques,
* proposer améliorations,
* lister les validations manquantes.

---

## Étape 4 — Prompt files : industrialiser les demandes répétitives (optionnel mais rentable)

Les prompt files servent à lancer des “routines” via `/` dans le chat.[^vscode-prompt-files]

### Exemple : `.github/prompts/plan.prompt.md`

```md
---
name: plan
description: "Générer un plan d’implémentation à partir d’une demande."
agent: Planner
---

À partir de la demande suivante, produis un plan d’implémentation conforme aux instructions du projet.
Demande :
${input:demande:Décris la feature / refacto}
```

Utilisation : dans le chat VS Code, tapez `/plan` puis complétez.[^vscode-prompt-files]

---

## Exemple concret (mini user story)

> “Ajouter un endpoint `GET /customers/{id}` avec contrôle d’accès et tests.”

1. Sélectionner l’agent **Planner**
2. Lancer `/plan` (ou écrire la demande)
3. Obtenir un plan : points d’entrée, structure, tests, rollback
4. Cliquer le handoff **“Démarrer l’implémentation”**
5. L’agent **Implementation** exécute le plan, teste, documente
6. (Optionnel) Handoff vers **Review**

---

## Pièges classiques (et comment les éviter)

### 1) Trop d’instructions tue l’instruction

Si votre `.github/copilot-instructions.md` fait 200 lignes, il va être ignoré ou mal appliqué. Gardez-le **court**, pointez vers les docs.

### 2) Docs “piliers” obsolètes = IA dangereuse

Si `ARCHITECTURE.md` ment, l’agent suivra un mensonge.
Traitez ces fichiers comme du code : review, versioning, mise à jour.

### 3) Outils trop permissifs en phase Plan

Le Plan n’a pas besoin d’édition. Limitez ses outils (sinon un “petit edit” arrive vite).

### 4) Handoffs sans “garde-fous”

Un handoff avec `send: true` peut lancer une implémentation automatiquement. Si vous tenez à la validation humaine, laissez `send: false`.

---

## Check-list de sortie (signaux observables)

* [ ] Le fichier `.github/copilot-instructions.md` existe et est appliqué (option activée).[^vscode-instructions]
* [ ] L’agent **Planner** est visible dans la liste des agents.[^vscode-agents]
* [ ] L’agent **Implementation** est visible dans la liste des agents.[^vscode-agents]
* [ ] Le Planner produit un plan structuré (étapes + tests + rollback + checklist).
* [ ] L’Implementation exécute en petits lots, lance les tests, et documente.
* [ ] Les docs `ARCHITECTURE/PRODUCT/CONTRIBUTING` sont présentes et utilisées comme références.

---

## Alternatives (quand choisir autre chose)

* **Vous êtes solo, petits scripts, faible enjeu** : restez sur l’agent “généraliste”, mais gardez au moins un `copilot-instructions.md` minimal.
* **Vous avez déjà un processus d’ADR/PRD béton** : branchez le Planner sur ces docs (ne dupliquez pas).
* **Vous ne voulez pas maintenir des agents** : VS Code propose un agent de planification intégré (mais vous perdez une partie de la standardisation).

## Sources

[^vscode-version]: Microsoft — Visual Studio Code: versioning et dépannage (`Help: About` / `code --version`): <https://code.visualstudio.com/docs/supporting/faq#_how-do-i-find-the-version>.
[^vscode-instructions]: Microsoft — GitHub Copilot Chat: custom instructions et fichiers d’instructions du workspace (`.github/copilot-instructions.md`, `*.instructions.md`): <https://code.visualstudio.com/docs/copilot/copilot-customization#_custom-instructions>.
[^vscode-agents]: Microsoft — GitHub Copilot Chat: agents et custom agents (`.github/agents/*.agent.md`, handoffs): <https://code.visualstudio.com/docs/copilot/copilot-chat#_agents>.
[^vscode-prompt-files]: Microsoft — GitHub Copilot Chat: prompt files (`.github/prompts/*.prompt.md`): <https://code.visualstudio.com/docs/copilot/copilot-customization#_prompt-files>.
[^vscode-context-engineering]: Microsoft — "Set up a context engineering flow in VS Code": <https://code.visualstudio.com/docs/copilot/copilot-chat-context#_set-up-a-context-engineering-flow-in-vs-code>.

