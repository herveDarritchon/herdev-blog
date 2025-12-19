---
name: write-herdev-post
description: Rédige un post HerDev durable (doc + procédure), à partir d’un brief.
argument-hint: "topic=...; goal=...; persona=...; context=...; env=...; constraint=...; format=...; target_file=..."
agent: "herdev-blog-writer"
tools: ["search", "web/fetch", "web/githubRepo", "search/usages"]
---

Vous rédigez un post pour **HerDev Blog** à partir du brief ci-dessous.

## Brief (fourni par l’utilisateur)
- Sujet / problème à résoudre : ${input:topic:Quel problème précis ce post doit résoudre ?}
- Objectif (résultat attendu) : ${input:goal:Quel résultat concret le lecteur doit obtenir ?}
- Persona : ${input:persona:Dev ? Tech lead ? Autre ?}
- Contexte projet : ${input:context:Taille d’équipe, contraintes, environnement, etc.}
- OS / shell / versions : ${input:env:OS, shell, versions des outils/techs concernés}
- Contrainte clé : ${input:constraint:Ex: pas de dépendances, compat, perf, sécurité, délai…}
- Format souhaité : ${input:format:FICHE | TUTO | GUIDE | CADRE | REX (ou "auto")}
- Fichier cible (optionnel) : ${input:target_file:Chemin du fichier Markdown à créer/mettre à jour (sinon laisser vide)}

## Sources (exigence)
- N’utilisez **que des sources officielles / primaires** pour les “best practices” (documentation éditeur, repo officiel, standards).
- Si une info est incertaine ou dépendante d’une version/OS/config : balisez-la ([Vérifié doc] / [À vérifier] / [Hypothèse]) et donnez une méthode pour trancher.
- Si besoin d’info à jour : utilisez #tool:search puis #tool:fetch. Évitez les blogs et posts non officiels sauf absence totale de source primaire.

## Références internes à appliquer (ne pas dupliquer, juste suivre)
- Invariants : [copilot-instructions.md](../copilot-instructions.md)
- Règles blog (formats/DONE/meta/maillage) : [herdev-blog.instructions.md](../instructions/herdev-blog.instructions.md)
- DOD : [DOD.md](../../resources/DOD.md) (si le lien diffère dans le repo, l’ajuster)
- ANNEXE : [ANNEXE-project-instructions.md](../../resources/ANNEXE-project-instructions.md) (si présent)

Règle de priorité : en cas de contradiction, appliquez la règle la plus spécifique au périmètre du fichier édité.

## Production attendue
1) Si `format=auto`, choisissez le format adapté et respectez son contrat (FICHE/TUTO/GUIDE/CADRE/REX).
2) Produisez un **post complet en Markdown**, prêt à coller dans le fichier cible :
   - TL;DR
   - `<!--more-->`
   - Corps structuré selon le format
   - Validation(s) reproductibles / critères observables
   - Pièges + limites + ≥1 alternative + quand la choisir
   - Checklist de sortie
   - Refs (liens vers sources officielles)
   - Changelog (même minimal)
3) Si des infos essentielles manquent, posez **1–3 questions max** avant d’écrire.

## Sortie
- Si `target_file` est fourni : commencez par un en-tête `# Fichier: <path>` puis le contenu complet.
- Sinon : retournez le contenu complet du post (Markdown), sans blabla.
