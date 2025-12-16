---
title: "Hugo : organiser ses posts et ses ressources (bundles, _index, archetypes)"
date: 2025-12-16T14:00:00+01:00
draft: true
description: "Une arborescence simple et robuste pour écrire des posts (leaf bundles), ranger les ressources, structurer les sections (_index.md) et standardiser le front matter (archetypes)."
tags: ["hugo", "blog", "bundles", "content"]
categories: ["Hugo"]
---

### TL;DR (si tu ne veux retenir qu’une chose)

* posts = `.../<slug>/index.md`
* sections = `_index.md`
* ressources du post = dans le dossier du post
* `static/` pour la copie brute, `assets/` pour le global “intelligent”
* `archetypes/` pour arrêter de répéter le front matter

---

Si ton blog Hugo commence à grossir, le chaos arrive vite : images perdues, PDFs introuvables, “où est-ce que j’ai mis ce schéma ?”, et des pages liste impossibles à piloter.

Ici je documente une convention **simple, maintenable et future-proof** pour organiser :
- des posts (texte),
- leurs ressources (images, fichiers, données),
- des sections et sous-sections (`_index.md`),
- et du front matter standardisé via `archetypes/`.

> Important : Hugo **n’impose pas** `content/post` ou `content/posts`. Choisis un nom et **reste cohérent**, surtout avec ton thème. Le nom du dossier devient ta section et influence templates/archetypes/URLs.

---

## 1) La règle d’or : 1 post = 1 dossier (leaf bundle)

Un **leaf bundle**, c’est un dossier contenant `index.md`. Tout ce qui est spécifique au post vit à côté.

Exemple de post :

```

content/posts/hugo/2025-12-16-arborescence-posts-ressources/
index.md
cover.jpg
images/
schema.png
screenshot-1.webp
files/
checklist.pdf

```

Pourquoi c’est mieux :
- tu peux déplacer/renommer le post sans casser 15 liens vers des images,
- tu gardes le post “portable” (tout est dans un seul répertoire),
- tu peux traiter les images proprement côté Hugo (quand ton thème/templates le permettent).

---

## 2) `index.md` vs `_index.md` : ne te trompe pas, sinon tu te tires dans le pied

### `index.md` (leaf bundle)
- = **une page finale**
- = **pas d’enfants**
- = le bon choix pour **un post**

### `_index.md` (branch bundle)
- = **une page liste / section**
- = peut contenir des enfants (posts, sous-sections)
- = le bon choix pour piloter une page type `/posts/` ou `/posts/hugo/`

---

## 3) Construire une arborescence claire (sections + sous-sections)

Je conseille une structure “thèmes” :

```

content/
posts/
_index.md
hugo/
_index.md
2025-12-16-arborescence-posts-ressources/
index.md
cover.jpg
images/
files/
jdr/
_index.md
2025-12-10-ma-campagne/
index.md

````

- `posts/` : ton blog
- `posts/hugo/` : une sous-section dédiée Hugo
- `posts/jdr/` : une sous-section JDR

---

## 4) Que mettre dans un `_index.md` (et pourquoi)

`_index.md` sert à piloter la page liste : titre, description, ordre, paramètres de thème, et parfois une intro affichée au-dessus des posts.

Exemple `content/posts/_index.md` :

```md
---
title: "Blog"
description: "Notes, retours d’expérience, docs et outils."
---
Ici je publie des notes courtes et des articles plus longs.
````

Exemple `content/posts/hugo/_index.md` :

```md
---
title: "Hugo"
description: "Conventions, pipeline, content modeling, thèmes, i18n."
weight: 10
---
Des posts concrets pour écrire et publier sans friction.
```

Ce que ça t’apporte :

* tu contrôles l’UX des pages liste,
* tu peux ordonner les sections (`weight`),
* tu ajoutes du contexte (intro) sans bricolage.

---

## 5) Quelles ressources mettre dans un post bundle ?

### Tu peux mettre :

* **images** (`cover.jpg`, `schema.png`, `*.webp`)
* **PDF/ZIP/etc.** (dans `files/`)
* **audio/vidéo** (`mp3`, `mp4`)
* **données** (`json`, `csv`) si tu les exploites dans un shortcode/template
* **annexes** (autres fichiers `.md`), si tu veux les inclure/consommer comme ressources

### Tu devrais mettre (minimum viable “propre”) :

* `cover.*` si ton thème supporte une image de couverture
* `images/` pour toutes les images inline
* `files/` pour tout ce qui est téléchargeable

---

## 6) Comment référencer les ressources depuis `index.md`

### Simple (100% suffisant pour démarrer)

Dans ton post :

```md
![Schéma du pipeline](images/schema.png)

[Télécharger la checklist](files/checklist.pdf)
```

Ça marche parce que les liens relatifs sont résolus depuis le bundle.

### Plus propre (si tu veux légendes/SEO)

Tu peux utiliser un shortcode `figure` (selon ton thème) :

```md
{{< figure src="images/schema.png" caption="Arborescence recommandée" >}}
```

---

## 7) `assets/` vs `static/` : où mettre quoi ?

C’est un choix d’architecture de ton site.

### `static/`

* copié tel quel dans le site généré
* pratique pour : favicon, robots, vérifs, fichiers à URL fixe

### `assets/`

* fait pour les “global resources” et les traitements (selon ton setup)
* pratique pour : ressources partagées, images globales, CSS/SCSS, JS, etc.

**Règle simple :**

* spécifique à un post → **dans le bundle**
* partagé partout → **assets/** (ou `static/` si tu veux juste copier tel quel)

---

## 8) Archetypes : arrêter de refaire le front matter à la main

Tu peux garder `archetypes/default.md` comme fallback, et créer un archetype dédié à tes posts.

Si ta section s’appelle `posts` :

* `archetypes/posts.md`

Si ta section s’appelle `post` :

* `archetypes/post.md`

Exemple d’archetype (YAML) :

```md
---
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
date: '{{ .Date }}'
draft: true
description: ""
tags: []
categories: ["Hugo"]
---
```

Résultat : chaque `hugo new content posts/.../index.md` sort directement avec un front matter cohérent.

---

## 9) i18n des `_index.md` (si ton blog est multilingue)

Deux stratégies propres (choisis-en une et reste dessus) :

### A) fichiers suffixés

```
content/posts/_index.fr.md
content/posts/_index.en.md
content/posts/hugo/_index.fr.md
content/posts/hugo/_index.en.md
```

### B) un `contentDir` par langue

```
content/fr/posts/_index.md
content/en/posts/_index.md
```

Dans les deux cas, l’objectif est le même : tes pages liste deviennent réellement traduites (titre + intro + description).

---

## 10) Mini-workflow (pipeline de création)

Quand tu standardises l’arbo + archetypes, créer un post Hugo devient un réflexe :

* Créer :

    * `.../posts/hugo/YYYY-MM-DD-mon-sujet/index.md`
* Ajouter ressources :

    * `cover.jpg`, `images/*`, `files/*`
* Publier :

    * passer `draft: false`
    * commit + push (ou déploiement, selon ton setup)

La valeur n’est pas “Hugo fait du Markdown”. La valeur c’est : **moins de friction** + **moins d’erreurs** + **une base qui tient à 200 posts**.

