---
title: "VS Code : Agent Skills — capitaliser du savoir-faire sans saturer le contexte"
date: 2026-01-11T13:00:00+01:00
description: "Comprendre et déployer les Agent Skills (SKILL.md) dans VS Code : déclenchement sémantique, divulgation progressive, structure de repo, et orchestration avec des custom agents + handoffs."
slug: "vscode-agent-skills"
categories: ["tooling", "engineering", "ai"]
series: "context-engineering"
tags: ["vscode", "github-copilot", "agents", "agent-skills", "context-engineering"]
draft: false
---

## TL;DR

- Un LLM sans contexte n’est pas “mauvais” : il est **généraliste**. Les *Agent Skills* servent à **injecter du savoir-faire procédural** au bon moment, sans noyer la fenêtre de contexte.
- Une skill = **un dossier** (portable) avec `SKILL.md` (YAML `name` + `description` obligatoires) + ressources (scripts, templates, exemples).  
- Chargement **à la demande** via **déclenchement sémantique** (match sur `description`) + **divulgation progressive** en 3 niveaux (métadonnées → instructions → ressources).  
- Dans VS Code 1.108, le support est **expérimental** et se pilote avec `chat.useAgentSkills`.[^vscode-agent-skills]
- Combinez *Skills* (le **quoi**) + *Custom Agents* (le **qui**) + *Handoffs* (le **workflow**) pour arrêter de “prompt-hacker”.

<!--more-->

## Du “chat” au coéquipier spécialisé (sans magie)

On attend souvent d’un assistant IA qu’il sache *déjà* vos conventions, vos scripts de release, votre CI, votre manière de diagnostiquer un bug, vos règles de PR… sauf que :

- la fenêtre de contexte est finie ;
- le modèle est généraliste ;
- et les instructions “toujours actives” finissent en bruit.

Les **Agent Skills** attaquent précisément ce problème : **rendre disponible un mode opératoire spécialisé** uniquement quand il est pertinent.

## Agent Skills : définition opérationnelle

Une *Agent Skill* est un **dossier** contenant :

- un fichier **`SKILL.md`** (obligatoire) : métadonnées + instructions ;
- des ressources optionnelles (scripts, templates, exemples) que l’agent peut charger si nécessaire.

C’est un **standard ouvert**, pensé pour être réutilisable d’un produit à l’autre (pas juste “un réglage VS Code”). Voir le standard Agent Skills : https://agentskills.io/home[^standard-agent-skills]

### Format minimal de `SKILL.md`

Le cœur, c’est le frontmatter YAML : `name` + `description` sont obligatoires.[^github-agent-skills]

```yaml
---
name: webapp-testing
description: Guide pour créer, exécuter et déboguer des tests Playwright. À utiliser quand on demande des tests E2E ou le diagnostic d’un test navigateur.
---
````

Ensuite, vous écrivez la procédure (Markdown) : étapes, entrées/sorties attendues, références aux ressources du dossier.

## Pourquoi ça marche : déclenchement sémantique + divulgation progressive

La mécanique est simple et robuste :

1. **Découverte** : Copilot ne connaît que `name` + `description` (léger).
2. **Chargement des instructions** : si votre demande “matche” sémantiquement la `description`, le corps de `SKILL.md` est injecté.
3. **Accès aux ressources** : les fichiers annexes ne sont chargés **que si** `SKILL.md` les référence.

Résultat : vous pouvez installer beaucoup de skills sans “payer” le coût contextuel en permanence.

## Skills vs instructions : arrêtez de tout mettre au même endroit

Les *custom instructions* servent bien pour des règles **stables et transverses** (style, conventions).
Les *Agent Skills* sont faites pour des workflows **spécialisés** et **déclenchés à la demande** : diagnostic CI, release, playbook incident, génération de tests, migration, etc.[^vscode-agent-skills]

**Heuristique franche** :

* si c’est “toujours vrai” → instructions ;
* si c’est “vrai quand je fais X” → skill.

## Mise en place dans VS Code (À reproduire)

### 1) Activer le support (1.108+)

Dans VS Code 1.108, c’est derrière un flag :

* Paramètre : `chat.useAgentSkills`[^vscode-release-1-108]

### 2) Choisir l’emplacement

VS Code détecte :

* **Skills de projet** : `.github/skills/` (recommandé) ou `.claude/skills/` (legacy)
* **Skills personnelles** : `~/.github/skills/` (recommandé) ou `~/.claude/skills/` (legacy)

> Note : côté GitHub Copilot CLI / coding agent, vous verrez aussi `~/.copilot/skills` comme répertoire personnel (selon l’outil).[^github-agent-skills]

### 3) Structure de repo recommandée

```text
.github/
  skills/
    hugo-post/
      SKILL.md
      templates/
        post.md
    ci-debug/
      SKILL.md
      checklists/
        triage.md
  agents/
    planner.agent.md
    implementer.agent.md
```

## Exemple concret : 2 skills “rentables” tout de suite

### Skill 1 — `hugo-post` : générer un post Hugo cohérent (front matter + sections)

**Objectif** : vous empêcher de repartir de zéro et garantir une structure durable (TL;DR, `<!--more-->`, refs, changelog).

`./.github/skills/hugo-post/SKILL.md`

```md
---
name: hugo-post
description: Créer ou refactorer un article Hugo (front matter YAML + TL;DR + <!--more--> + sections + refs + changelog). Utiliser quand on demande un blog post “prêt à committer”.
---

# Hugo post (HerDev) — Procédure

## Quand utiliser
- Création d’un nouvel article
- Réécriture/normalisation d’un article existant
- Uniformisation front matter / slug / tags

## Procédure
1. Proposer un **slug** stable (kebab-case) et une **description** SEO sobre.
2. Générer le squelette depuis le template : `./templates/post.md`
3. Remplir :
   - TL;DR (4–6 bullets)
   - `<!--more-->` après le TL;DR
   - sections orientées “doc+procédure”
   - pièges + limites + alternatives
   - checklist de sortie (signaux observables)
   - Références (liens primaires)
   - Changelog

## Contraintes d’écriture
- Zéro hype, zéro promesse magique.
- Dire ce qui est “À reproduire” si non testé.
- Préférer “vous/on”.
```

`./.github/skills/hugo-post/templates/post.md`

```md
---
title: ""
date: ""
description: ""
slug: ""
categories: []
series: ""
tags: []
draft: true
---

## TL;DR

- 
- 
- 
- 

<!--more-->

## Contexte

## Procédure

## Validation

## Pièges et limites

## Alternatives

## Checklist de sortie

- [ ] 
- [ ] 

## Références

- 

## Changelog

- YYYY-MM-DD : création
```

### Skill 2 — `ci-debug` : playbook de diagnostic CI

**Objectif** : imposer une routine (repro → logs → hypothèses → fix → vérif).

`./.github/skills/ci-debug/SKILL.md`

```md
---
name: ci-debug
description: Diagnostiquer un pipeline CI en échec (GitHub Actions, etc.). Utiliser quand on demande “pourquoi ça casse en CI” ou “corrige le build”.
---

# CI Debug — Playbook

## Process
1. Identifier le job/step qui casse + message d’erreur exact.
2. Classer l’échec :
   - dépendance/versions
   - cache
   - permissions
   - flakiness (tests)
3. Reproduire localement si possible (mêmes versions).
4. Proposer **1 fix** minimal + **1 check** (preuve).
5. Ajouter garde-fou (test, assertion, pin de version, doc).

## Sortie attendue
- Cause probable (avec indices)
- Correctif minimal
- Validation (commande/log attendu)
```

## Orchestration : Skills (le quoi) + Custom Agents (le qui) + Handoffs (le workflow)

Les skills ne remplacent pas les agents personnalisés. Ils les **alimentent**.[^vscode-custom-agents]

* Un **agent “Planification”** peut s’appuyer sur une skill “Architecture ADR”.
* Un **agent “Implémentation”** peut charger une skill “Migration DB”.
* Un **agent “Review”** peut déclencher une skill “Checklist sécurité”.

### Exemple de handoff (plan → implémentation)

Les custom agents sont des fichiers `.agent.md` dans `.github/agents/`.[^vscode-custom-agents]

`./.github/agents/planner.agent.md`

```md
---
name: Planner
description: Génère un plan d’implémentation par étapes validables.
tools: ['search', 'fetch']
handoffs:
  - label: Démarrer l’implémentation
    agent: implementer
    prompt: Implémente le plan ci-dessus. Commence par les tests, puis le code, puis la vérif.
    send: false
---
Tu produis un plan numéroté, avec critères de validation par étape.
```

`send: false` force un humain dans la boucle. `send: true` est tentant… et dangereux tant que votre chaîne n’est pas éprouvée.

## Gouvernance : traitez vos skills comme du code

Une skill, ce n’est pas un post-it. C’est un **artefact de prod** :

* **Versionnez** `.github/skills/` comme du code.
* **Code review** obligatoire : une skill peut inciter à exécuter des scripts.
* Évitez les descriptions vagues (“aide pour tout”) : vous tuez le déclenchement sémantique.
* Gardez les skills petites : 1 tâche = 1 skill. Sinon elles deviennent des “manuels” que personne ne charge bien.

## Pièges fréquents (et comment les éviter)

* **Description trop large** → déclenchement aléatoire.
  *Fix* : mentionner clairement “quand l’utiliser” + 3 cas d’usage typiques.
* **Skill monolithe** → instructions longues, rarement chargées en entier.
  *Fix* : découper en skills “playbook”.
* **Scripts non bornés** → risques de commandes destructrices.
  *Fix* : préconditions, commandes de vérif avant/après, rollback minimal, et jamais d’auto-approve par défaut.
* **Divergence outils** (VS Code vs CLI) → chemins personnels différents.
  *Fix* : documenter “où ça vit” par outil (et rester sur le standard + compat).

## Checklist de sortie

* [ ] `chat.useAgentSkills` activé (VS Code 1.108+)
* [ ] Au moins 1 skill en `.github/skills/<skill>/SKILL.md`
* [ ] `name` en kebab-case, `description` avec “quand l’utiliser”
* [ ] La skill contient une procédure + sorties attendues
* [ ] Ressources référencées via chemins relatifs (si utilisées)
* [ ] Revue sécurité effectuée (scripts, commandes, accès)
* [ ] (Optionnel) 2 agents custom + 1 handoff plan → implémentation

## Sources

[^vscode-agent-skills]: Microsoft — Use Agent Skills in VS Code: <https://code.visualstudio.com/docs/copilot/customization/agent-skills>.
[^vscode-release-1-108]: Microsoft — VS Code 1.108 release notes: <https://code.visualstudio.com/updates/v1_108>.
[^vscode-custom-agents]: Microsoft — Custom agents in VS Code: <https://code.visualstudio.com/docs/copilot/customization/custom-agents>.
[^github-agent-skills]: GitHub Docs — About Agent Skills: <https://docs.github.com/en/copilot/concepts/agents/about-agent-skills>.
[^standard-agent-skills]: Agent Skills — Overview: <https://agentskills.io/home>.
