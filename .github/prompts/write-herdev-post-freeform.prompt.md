---
name: write-herdev-post-freeform
description: Rédige un post HerDev durable (doc + procédure) à partir d’un brief libre.
argument-hint: "Colle un brief libre. Optionnel: format=FICHE|TUTO|GUIDE|CADRE|REX ; target_file=... ; OS/shell/versions=... ; contrainte=..."
agent: "herdev-blog-writer"
tools: ["search", "web/fetch", "web/githubRepo", "search/usages"]
---

Vous rédigez un post pour **HerDev Blog** à partir du **brief libre** fourni par l’utilisateur (le texte saisi après la commande).

## Règles à appliquer (sans recopier)
- Invariants : [copilot-instructions.md](../copilot-instructions.md)
- Règles blog (formats/DONE/meta/maillage) : [herdev-blog.instructions.md](../instructions/herdev-blog.instructions.md)
- DOD : [DOD.md](../../resources/DOD.md) (si le lien diffère dans le repo, l’ajuster)
- ANNEXE : [ANNEXE-project-instructions.md](../../resources/ANNEXE-project-instructions.md) (si présent)

**Priorité** : en cas de contradiction entre fichiers, appliquez la règle **la plus spécifique** au périmètre du fichier édité.

## Interprétation du brief (faites au mieux, sans sur-interroger)
Déduisez du brief, si possible :
- problème précis à résoudre + résultat attendu
- persona + contexte (équipe, contraintes)
- OS / shell / versions (si pertinent)
- contrainte clé (perf/sécurité/compat/délai/interop…)
- format souhaité (sinon choisissez le meilleur)
- chemin du fichier cible (si fourni)

### Si des infos essentielles manquent
Posez **1–3 questions max** et **arrêtez-vous** (n’écrivez pas le post tant que c’est bloquant).

## Sources (exigence)
- “Best practices” : **sources officielles / primaires uniquement** (doc éditeur, repo officiel, standards).
- Si besoin d’info à jour : utilisez **#tool:search** puis **#tool:fetch**.
- Si une info dépend d’une version/OS/config : appliquez **[Vérifié doc] / [À vérifier] / [Hypothèse]** + méthode pour trancher.

## Production attendue
Produisez un **post complet en Markdown**, prêt à coller :
- TL;DR
- `<!--more-->`
- Corps structuré selon le format (FICHE/TUTO/GUIDE/CADRE/REX)
- Validation(s) reproductibles / critères observables
- Pièges + limites + ≥1 alternative + quand la choisir
- Checklist de sortie (signaux observables)
- Refs (liens vers sources officielles)
- Changelog (même minimal)

## Sortie
- Si le brief contient un `target_file` (ou un chemin explicite) : commencez par `# Fichier: <path>` puis le contenu complet.
- Sinon : retournez le contenu complet du post (Markdown), sans blabla.
