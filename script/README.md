# README — `herdev` (CLI Hugo)

`herdev` est un petit script bash pour gérer ton blog Hugo depuis n’importe où : créer un post (bundle), lancer le serveur, publier proprement, afficher un post en stdout, et (nouveau) **créer un post à partir d’un fichier Markdown source en copiant aussi ses assets**.

---

## Pré-requis

* `hugo`
* `git`
* `iconv` (pour slugifier avec translit accents → ASCII)
* macOS : `open` (déjà présent)

---

## Installation rapide

1. Mets le script dans ton `$PATH`, par exemple :

```bash
mkdir -p ~/bin
cp herdev ~/bin/herdev
chmod +x ~/bin/herdev
```

2. Configure tes valeurs par défaut dans ton shell (`~/.zshrc` par ex.) :

```bash
export HERDEV_DIR="$HOME/Workspace/Perso/Blogs/herdev-blog"
export HERDEV_SECTION="posts"
export HERDEV_EDITOR="code -n"
```

Recharge ton shell (`source ~/.zshrc`) ou ouvre un nouveau terminal.

---

## Options globales (paramètres)

Ces options s’appliquent **à toutes** les commandes.

### `-C, --dir <path>` — choisir le repo blog

```bash
herdev -C ~/Workspace/Perso/Blogs/herdev-blog serve
```

### `-S, --section <name>` — choisir la section Hugo

```bash
herdev -S notes new "Une note rapide"
```

### `-E, --editor <cmd>` — forcer l’éditeur

```bash
herdev -E "code -n" new "Un post ouvert dans VS Code"
```

### `-h, --help` — aide

```bash
herdev --help
```

---

## Pipeline logique “habituel” (avec exemples concrets)

### 0) Afficher un post existant comme “template” (stdout)

```bash
herdev display example
```

Ou un post précis :

```bash
herdev display example content/posts/2025-12-16-mon-post/index.md
```

Redirection utile :

```bash
herdev display example > ./drafts/template.md
```

---

## 1) Créer un post

Tu as **trois workflows**.

### A) À partir d’un titre (classique)

Crée un bundle via `hugo new` et l’ouvre :

```bash
herdev new "Sealed Class : sérialisation des sous-types en Kotlin"
```

---

### B) À partir d’un fichier Markdown source (copie “md only”)

Crée un bundle et copie le fichier `.md` tel quel dans `index.md`.

✅ Quand l’utiliser : tu as déjà ton contenu rédigé dans un fichier (brouillon, export, template rempli, etc.) et tu veux l’importer.

Contraintes :

* le fichier source **doit contenir un front matter** (`---` ou `+++`) en première ligne
* `herdev` **n’ajoute aucun front matter** (il copie le fichier entier)
* le slug est dérivé du `title` du front matter

Exemples :

```bash
herdev new ./drafts/mon-post.md
```

Forme explicite :

```bash
herdev new --from ./drafts/mon-post.md
```

---

### C) À partir d’un fichier Markdown + assets (nouveau : `--with-assets`)

`--with-assets` sert à copier **les fichiers du même dossier** que le `.md` source dans le bundle Hugo, en plus de `index.md`.

✅ Quand l’utiliser : ton post a des **images, schémas, fichiers annexes** référencés dans le markdown (typiquement `![...](image.png)` ou `![...](./images/foo.png)`).

📌 Où l’utiliser : **uniquement** avec `new --from <source.md>` (ou `new <source.md>`).
Ça n’a aucun sens avec `new "Titre"`.

**Commandes :**

```bash
herdev new --from ./drafts/mon-post.md --with-assets
```

ou, forme courte :

```bash
herdev new ./drafts/mon-post.md --with-assets
```

**Ce qui est copié :**

* tout ce qui est dans le **même dossier** que `mon-post.md`
* y compris sous-dossiers (copie récursive)

**Ce qui est volontairement exclu :**

* le fichier `.md` source lui-même (déjà copié en `index.md`)
* les autres fichiers `*.md` (pour éviter de publier tes brouillons/notes par erreur)
* `.DS_Store`

> Conséquence pratique : mets ton `.md` et ses images dans un même dossier “draft”, par exemple `./drafts/mon-post/`, et ça se transfère proprement vers `content/<section>/.../`.

Exemple structuré recommandé :

```
drafts/
  mon-post/
    mon-post.md
    hero.png
    diagram.svg
    images/
      step1.png
      step2.png
```

Commande :

```bash
herdev new --from ./drafts/mon-post/mon-post.md --with-assets
```

---

## 2) Prévisualiser en local (drafts inclus)

```bash
herdev serve
```

---

## 3) Publier

Publier un post précis :

```bash
herdev publish content/posts/2025-12-16-mon-post/index.md
```

Publier sans chemin (prend le premier fichier modifié sous `content/`) :

```bash
herdev publish
```

> `publish` est “bundle-safe” : si tu publies un `.../index.md`, il ajoute automatiquement le **dossier du bundle** (donc images incluses).

---

## Variables d’environnement (par défaut)

* `HERDEV_DIR` : chemin du repo Hugo
* `HERDEV_SECTION` : section (ex: `posts`)
* `HERDEV_EDITOR` : commande d’éditeur (ex: `code -n`)

---
