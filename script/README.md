# README — `herdev` (CLI Hugo)

`herdev` est un petit script bash pour gérer ton blog Hugo depuis n’importe où : créer un post (bundle), lancer le serveur, publier proprement, et afficher un post en stdout (pratique pour copier/coller un template ou montrer un exemple).

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

Cas typique : tu bosses sur plusieurs blogs (ou ton blog n’est pas dans le dossier par défaut).

```bash
herdev -C ~/Workspace/Perso/Blogs/herdev-blog serve
```

### `-S, --section <name>` — choisir la section Hugo

Cas typique : tu as plusieurs sections (`posts`, `notes`, `jvm`, etc.) et tu veux que `new` crée au bon endroit.

```bash
herdev -S notes new "Une note rapide"
```

### `-E, --editor <cmd>` — forcer l’éditeur

Cas typique : tu veux ouvrir dans VS Code, ou dans un autre éditeur, ponctuellement.

```bash
herdev -E "code -n" new "Un post ouvert dans VS Code"
```

### `-h, --help` — aide

```bash
herdev --help
```

---

## Pipeline logique “habituel” (avec exemples concrets)

L’idée : tu pars d’un “template” que tu veux reproduire, tu crées ton post, tu le remplis, tu vérifies en local, puis tu publies.

### 0) Afficher un post existant comme “template” (stdout)

**Objectif :** récupérer un exemple de structure (front matter, sections, shortcodes, etc.) sans ouvrir l’éditeur.

* Prendre le post **le plus récent** de `content/<section>/` :

```bash
herdev display example
```

* Prendre un post précis (chemin relatif au repo) :

```bash
herdev display example content/posts/2025-12-16-mon-post/index.md
```

* Ou un chemin relatif à `content/<section>/` :

```bash
herdev display example 2025-12-16-mon-post/index.md
```

Tu peux ensuite copier/coller, ou mieux : rediriger vers un fichier temporaire pour bosser dessus :

```bash
herdev display example > /tmp/template.md
open /tmp/template.md
```

> Point important : ça “dump” le contenu tel quel. Si tu veux un vrai système de “new from template”, ça se fait, mais c’est une fonctionnalité distincte (et ça impose de choisir *où* vivent les templates).

---

### 1) Créer un nouveau post bundle

**Objectif :** générer `content/<section>/YYYY-MM-DD-slug/index.md` et l’ouvrir.

```bash
herdev new "Sealed Class : sérialisation des sous-types en Kotlin"
```

Avec section spécifique :

```bash
herdev -S jvm new "Kotlin: sealed class et polymorphisme"
```

Avec un repo explicite (si tu n’as pas exporté `HERDEV_DIR`) :

```bash
herdev -C ~/Workspace/Perso/Blogs/herdev-blog new "Mon nouveau post"
```

---

### 2) Éditer (dans ton éditeur)

Si tu as défini `HERDEV_EDITOR="code -n"`, le fichier s’ouvre automatiquement.

Sinon tu peux forcer à la commande :

```bash
herdev -E "code -n" new "Mon post"
```

---

### 3) Prévisualiser en local (drafts inclus)

**Objectif :** vérifier rendu, liens, images, shortcodes, etc.

```bash
herdev serve
```

Si tu as plusieurs repos :

```bash
herdev -C ~/Workspace/Perso/Blogs/herdev-blog serve
```

---

### 4) Publier

**Objectif :**

* met `draft: true` → `draft: false` si présent (YAML ou TOML)
* build local (`hugo --minify`) pour casser si ça ne compile pas
* commit + push

Publier en donnant le chemin :

```bash
herdev publish content/posts/2025-12-16-mon-post/index.md
```

Publier sans chemin : il prend le **premier fichier modifié** sous `content/` selon `git status`.

```bash
herdev publish
```

> Important : dans la version script que tu as, `publish` fait `git add "$target"` (donc il ajoute seulement le fichier).
> Si ton post est un bundle avec images, ça ne suffit pas. Dans ce cas, publie en ciblant le dossier (ou améliore le script pour `git add "$(dirname "$target")"`). C’est le comportement le plus logique pour Hugo bundles.

---

## Recettes pratiques (vraie vie)

### Pipeline “template → nouveau post”

1. Je récupère un exemple :

```bash
herdev display example content/posts/2025-11-01-mon-template/index.md > /tmp/template.md
```

2. Je crée un nouveau post :

```bash
herdev new "Mon nouveau post basé sur le template"
```

3. Je copie les sections utiles du template dans le nouveau `index.md` (dans l’éditeur)

4. Je preview :

```bash
herdev serve
```

5. Je publie :

```bash
herdev publish content/posts/2025-12-16-mon-nouveau-post/index.md
```

---

## Variables d’environnement (par défaut)

* `HERDEV_DIR` : chemin du repo Hugo
* `HERDEV_SECTION` : section (ex: `posts`)
* `HERDEV_EDITOR` : commande d’éditeur (ex: `code -n`)

Exemple réaliste :

```bash
export HERDEV_DIR="$HOME/Workspace/Perso/Blogs/herdev-blog"
export HERDEV_SECTION="posts"
export HERDEV_EDITOR="code -n"
```

---

## Frictions à anticiper (et ce que je recommande)

* Si tu utilises des **bundles avec images** : ton `publish` doit ajouter **le dossier du bundle**, pas juste `index.md`.
  Recommandation : remplacer dans le script :

```bash
git -C "$BLOG_DIR" add "$target"
```

par :

```bash
git -C "$BLOG_DIR" add "$(dirname "$target")"
```

Ça évite les “je publie le md mais j’oublie les images”.

* Ton mode “template” actuel est volontairement simple (affichage stdout). Si tu veux un vrai `new --from <template>`, on peut le faire proprement, mais il faudra décider :

  * où sont stockés les templates (`tools/templates/` ? `content/_templates/` ?),
  * comment on injecte automatiquement `title`, `date`, éventuellement `tags`.

Si tu veux, je te propose une extension minimale **sans complexifier** : `herdev new --from <path-to-template>` qui copie juste le contenu template dans le nouveau post en remplaçant `title` et `date`.
