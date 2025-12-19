# HerDev Blog — Instructions pour AI Agents

## Architecture du projet

**Stack technique** : Hugo (Extended) + fork maintenu du thème PaperCSS.

```
content/                    # Contenu Markdown bilingue (fr/en)
├── posts/                  # Posts organisés en leaf bundles (1 dossier = 1 post)
│   ├── hugo/              # Sous-sections thématiques
│   ├── architecture/
│   ├── jvm/
│   └── ide/
config/_default/config.toml # Config Hugo (multilangue, permalinks, taxonomies)
themes/papercss-hugo-theme/ # Thème (submodule Git)
script/herdev               # CLI bash pour créer/publier posts
resources/                  # Instructions éditoriales (CORE, DOD, ANNEXE)
public/                     # Site généré (ne pas éditer manuellement)
```

**Point clé** : chaque post est un **leaf bundle** (`content/posts/<categorie>/<slug>/index.md`) avec ses ressources (
images, PDFs) dans le même dossier. Ne jamais mettre de contenu directement sous `public/`.

## Workflows critiques

### Créer un nouveau post

```bash
# Via le CLI herdev (recommandé)
./script/herdev new "Titre du post"

# Ou via Hugo directement
hugo new posts/categorie/mon-slug/index.md
```

Le CLI `herdev` :

- slugifie le titre (accents → ASCII, minuscules)
- crée un leaf bundle avec la date (YYYY-MM-DD-slug/)
- ouvre le fichier dans l'éditeur configuré (`$HERDEV_EDITOR`)
- supporte `--with-assets` pour copier des fichiers sources

### Dev local

```bash
# Serveur dev avec drafts
hugo server -D

# Avec bind réseau (test mobile)
hugo server -D --bind 0.0.0.0 --baseURL http://192.168.1.10:1313/

# Sans fast render (si problèmes CSS/templates)
hugo server -D --disableFastRender
```

### Build production

```bash
hugo --minify              # Génère dans public/
rm -rf public resources    # Clean rebuild si nécessaire
```

### Publier un post

```bash
./script/herdev publish [path]  # Met draft=false, build, commit, push
```

## Conventions éditoriales strictes

Workflow d’écriture efficace (Copilot) :

     1) produire un outline court avec `/outline-herdev-post` (sections + promesse + critères observables) et corriger le scope si nécessaire ;
     2) générer le draft complet avec `/draft-herdev-post` (DONE + meta) ; 
     3) lancer le durcissement anti‑invention via 
        - le handoff **Durcir (anti‑invention)** 
        - ou `/harden-herdev-post` pour obtenir la liste des affirmations non triviales balisées `[Vérifié doc]` / `[À vérifier]` / `[Hypothèse]` ; 
     4) réécrire uniquement les sections signalées 
        - via le handoff **Réécrire les sections à risque** ;
     5) puis (optionnel) traduire 
        - via le handoff **Traduire en anglais (.en.md)** pour produire `post.en.md` dans le même dossier :

Ce blog suit un **modèle "base de connaissance"** (pas un journal). Instructions complètes
dans [resources/CORE-project-instructions.md](../resources/CORE-project-instructions.md)
et [resources/DOD.md](../resources/DOD.md).

**Règles non-négociables lors de la génération de contenu** :

- **Vérifiabilité absolue** : ne jamais inventer flags, options, sorties de commandes, comportements par défaut
- **Marquage obligatoire** : `[Vérifié doc]` (lien) ou `[À vérifier]` (commande) ou `[Hypothèse]` (méthode)
- **Pas de "testé sur"** sauf si l'utilisateur fournit versions/logs/outils/date explicites
- **Voix neutre** (vous/on), jamais "je/nous" sauf REX avec données terrain fournies

**Structure d'un post complet** :

1. Front matter : `title`, `date`, `draft`, `description`, `tags`, `categories`, `series` (optionnel)
2. TL;DR (1 bullet point essentiel)
3. `<!--more-->` (séparateur summary/contenu)
4. Contexte + intention + persona + contraintes
5. Étapes numérotées avec checkpoints
6. Alternatives + trade-offs + limites
7. Validation (checklist de sortie)
8. Références (doc primaire obligatoire)
9. Changelog (pour mises à jour)

**Tags autorisés** : chercher dans `config/_default/config.toml` ou regarder les posts existants. Préférer réutiliser
avant de créer.

## Patterns spécifiques

### Front matter

Utiliser les archetypes dans `archetypes/posts.md` comme base. Les champs Git (`lastmod`) sont gérés automatiquement via
`enableGitInfo = true`.

### Permalinks

Config : `permalinks.posts = "/:contentbasename/"` → l'URL est générée depuis le nom du dossier, **pas** depuis `title`.

### Multilinguisme

- Fichier FR : `index.md` ou `fichier.md`
- Fichier EN : `index.en.md` ou `fichier.en.md`
- La langue par défaut (`defaultContentLanguage = "fr"`) n'utilise pas de préfixe dans l'URL

### Ressources locales dans un post

Référencer directement : `![Description](image.webp)` ou `[PDF](fichier.pdf)` (chemin relatif depuis `index.md`).

### Taxonomies actives

`tags`, `categories`, `series`, `tutorials` (définies dans `config.toml`).

## Dépendances externes

- **Thème** : [papercss-hugo-theme](https://github.com/hervedarritchon/papercss-hugo-theme) (submodule)
    - Mise à jour : `git submodule update --remote --merge themes/papercss-hugo-theme`
- **Hugo Extended** requis (pour les pipelines SCSS)
- **Variables d'environnement** pour `herdev` : `HERDEV_DIR`, `HERDEV_SECTION`, `HERDEV_EDITOR`

## Éviter ces erreurs fréquentes

- Ne **jamais** éditer directement dans `public/` (écrasé à chaque build)
- Ne pas oublier `<!--more-->` sinon tout le contenu apparaît dans les listes
- Respecter la casse des taxonomies (`tags:` pas `Tags:`)
- Ne pas créer de `content/posts/mon-post.md` seul → toujours utiliser un leaf bundle avec `index.md`
- Vérifier que les images sont bien dans le dossier du post (pas dans `static/` sauf ressources globales)
