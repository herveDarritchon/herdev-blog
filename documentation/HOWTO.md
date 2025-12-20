---
title: "How-to — Inclure du code de herdev-labs dans herdev-blog (shortcodes Hugo)"
description: "Guide pratique pour utiliser les shortcodes codefile/codelines/codesnip/labzip et garder les posts synchronisés avec le code du repo herdev-labs."
---

# How-to — Inclure du code de `herdev-labs` dans `herdev-blog`

Ce guide explique comment intégrer **du code réel** (source de vérité) provenant du submodule `external/herdev-labs` dans tes articles Hugo, via des **shortcodes**.

Objectifs :
- éviter le copier-coller (et la divergence code ↔ article)
- rendre les posts **reproductibles**
- garder une convention stable et scalable

---

## Pré-requis

### 1) Submodule `herdev-labs` présent
Dans `herdev-blog`, tu dois avoir :

- `external/herdev-labs/` (submodule)

Vérifier :

```bash
git submodule status
ls external/herdev-labs
````

### 2) Shortcodes installés

Dans `herdev-blog/layouts/shortcodes/` :

* `codefile.html`
* `codelines.html`
* `codesnip.html`
* (optionnel) `labzip.html`

---

## Convention de chemins

Dans les posts, **toujours** utiliser des chemins relatifs au repo blog, typiquement :

* `external/herdev-labs/<lang>/<lab>/...`

Exemple :

* `external/herdev-labs/kotlin/jackson-sealed-class/src/test/kotlin/scenario02_property/Scenario02_PropertyTest.kt`

⚠️ Important : un chemin utilisé dans un post devient une **API**.
Renommer/déplacer un fichier casse l’article.

---

## Shortcode `codefile`

### À quoi ça sert

Afficher **un fichier entier** (pratique pour des petits fichiers, ou pour un README de lab).

### Signature

* `path` (obligatoire) : chemin vers le fichier
* `lang` (optionnel) : langage pour la coloration (`kotlin`, `java`, `json`, `bash`, `yaml`, …)

### Exemple

```md
{{< codefile
  path="external/herdev-labs/kotlin/jackson-sealed-class/README.md"
  lang="markdown"
>}}
```

### Bon usage

✅ fichiers courts (≤ ~120 lignes)
❌ éviter sur des tests entiers longs : préfère `codelines` ou `codesnip`.

---

## Shortcode `codelines`

### À quoi ça sert

Afficher un **extrait** basé sur des numéros de lignes.

### Signature

* `path` (obligatoire)
* `lang` (optionnel)
* `from` (optionnel, défaut `1`)
* `to` (optionnel, défaut très grand)

### Exemple

```md
{{< codelines
  path="external/herdev-labs/kotlin/jackson-sealed-class/src/test/kotlin/scenario01_notyping/Scenario01_NoTypingTest.kt"
  lang="kotlin"
  from="1"
  to="80"
>}}
```

### Bon usage

✅ extrait stable (haut de fichier, bloc qui ne bouge pas)
❌ fragile si tu refacto souvent (les lignes changent) → préfère `codesnip`.

---

## Shortcode `codesnip` (recommandé)

### À quoi ça sert

Afficher un extrait **délimité par des marqueurs** dans le code.
C’est la méthode la plus robuste sur la durée.

### Comment marquer un extrait dans le code

Dans ton fichier Kotlin (ou autre), ajoute :

```kotlin
// snippet:begin existing-property
... code ...
// snippet:end existing-property
```

> Le nom (`existing-property`) doit être unique **dans le fichier** et stable.

### Signature

* `path` (obligatoire)
* `lang` (optionnel)
* `name` (obligatoire) : nom du snippet (la clé après `snippet:begin`)

### Exemple

```md
{{< codesnip
  path="external/herdev-labs/kotlin/jackson-sealed-class/src/test/kotlin/scenario03_existingprop/Scenario03_ExistingPropTest.kt"
  lang="kotlin"
  name="existing-property"
>}}
```

### Bon usage

✅ idéal pour des posts maintenables (tu refacto sans casser l’article)
✅ parfait pour montrer “le morceau qui compte”
⚠️ discipline : ne supprime pas les marqueurs sans mettre à jour les posts.

---

## Shortcode `labzip` (optionnel)

### À quoi ça sert

Générer un lien “Download lab ZIP” vers un asset GitHub Release.

### Signature

* `user` (obligatoire) : pseudo GitHub
* `repo` (optionnel, défaut `herdev-labs`)
* `tag` (obligatoire) : tag de release (`lab-<slug>-vX.Y.Z`)
* `zip` (obligatoire) : nom du zip attaché à la release

### Exemple

```md
{{< labzip
  user="herveDarritchon"
  tag="lab-kotlin-jackson-sealed-class-v1.0.0"
  zip="kotlin-jackson-sealed-class-v1.0.0.zip"
>}}
```

### Bon usage

✅ dans chaque post : proposer un zip versionné = “preuve reproductible”
✅ évite le copier-coller d’URL d’assets

---

## Recettes d’utilisation dans un post (patterns)

### Pattern A — “Snippet + explication”

Le plus fréquent :

```md
Voici la stratégie `@JsonTypeInfo` avec un discriminant en propriété.

{{< codesnip
  path="external/herdev-labs/kotlin/jackson-sealed-class/src/test/kotlin/scenario02_property/Scenario02_PropertyTest.kt"
  lang="kotlin"
  name="type-info-property"
>}}
```

### Pattern B — “Extrait court + note de test”

Pour relier directement au “preuve par test” :

````md
**Preuve** : le test suivant passe.

{{< codelines
  path="external/herdev-labs/kotlin/jackson-sealed-class/src/test/kotlin/scenario02_property/Scenario02_PropertyTest.kt"
  lang="kotlin"
  from="1"
  to="60"
>}}

Pour vérifier :
```bash
cd kotlin/jackson-sealed-class
./gradlew test --tests "*Scenario02*"
````

````

### Pattern C — “Télécharger et exécuter sans Git”
Au début ou à la fin du post :

```md
### Tester sur ta machine (sans Git)

{{< labzip
  user="herveDarritchon"
  tag="lab-kotlin-jackson-sealed-class-v1.0.0"
  zip="kotlin-jackson-sealed-class-v1.0.0.zip"
>}}
````

---

## Recommandations éditoriales (pour rester scalable)

### 1) Préférer `codesnip`

* `codelines` = rapide, mais fragile
* `codesnip` = stable, maintenable

### 2) Garder les extraits courts

Un extrait doit éclairer **un point précis**.
Au-delà, tu perds le lecteur.

### 3) Une section “Preuve / Vérifier”

Dans chaque post technique, tu ajoutes un bloc standard :

* Lab : `kotlin/jackson-sealed-class`
* Scénario : `scenario02_property`
* Commande : `./gradlew test --tests "*Scenario02*"`
* (optionnel) ZIP release + version

### 4) Ne pas casser l’API des chemins

Si tu dois déplacer/renommer des fichiers cités dans les posts :

* fais-le en même temps que la mise à jour des articles
* idéalement bump la version du lab (SemVer)

---

## Dépannage

### Hugo échoue avec “fichier introuvable”

* vérifie que le submodule est bien init :

  ```bash
  git submodule update --init --recursive
  ```
* vérifie le chemin dans le shortcode
* vérifie que le fichier existe dans `external/herdev-labs/...`

### “codesnip: begin introuvable”

* le marker `snippet:begin <name>` n’est pas présent
* typo dans `name`
* markers supprimés lors d’un refactor

### Le rendu n’affiche pas la coloration attendue

* change `lang` (`kotlin`, `json`, `bash`, `yaml`, `toml`, etc.)
* vérifie la config Hugo highlight :

  ```toml
  [markup]
    [markup.highlight]
      noClasses = false
      lineNos = true
  ```

---

## Cheat sheet (copier-coller)

### Fichier entier

```md
{{< codefile path="external/herdev-labs/..." lang="kotlin" >}}
```

### Lignes 10–40

```md
{{< codelines path="external/herdev-labs/..." lang="kotlin" from="10" to="40" >}}
```

### Snippet nommé

```md
{{< codesnip path="external/herdev-labs/..." lang="kotlin" name="my-snippet" >}}
```

### ZIP release

```md
{{< labzip user="herveDarritchon" tag="lab-<slug>-vX.Y.Z" zip="<slug>-vX.Y.Z.zip" >}}
```