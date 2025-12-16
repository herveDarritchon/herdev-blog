---
title: "Hugo: Organizing posts and resources (bundles, _index, archetypes)"
date: 2025-12-16T14:00:00+01:00
draft: true
description: "A simple, scalable content tree for blog posts (leaf bundles), page resources, section pages (_index.md), and consistent front matter via archetypes."
tags: ["hugo", "blog", "bundles", "content"]
categories: ["Hugo"]
---

### TL;DR (si tu ne veux retenir qu’une chose)

* posts = `.../<slug>/index.md`
* sections = `_index.md`
* post resources = inside the bundle
* `static/` for raw copy, `assets/` for global “smart” resources
* `archetypes/` to stop rewriting front matter

When a Hugo site grows, content chaos arrives fast: images scattered everywhere, PDFs you can’t find again, broken links after a rename, and section pages you can’t control.

This post documents a convention that’s **simple, maintainable, and future-proof** for organizing:
- posts (Markdown),
- post resources (images, PDFs, data),
- sections and sub-sections (`_index.md`),
- and standardized front matter via `archetypes/`.

> Important: Hugo **does not require** `content/post` (singular) or `content/posts` (plural). Pick one and **stay consistent**, especially with your theme. The folder name becomes your section and can influence templates, archetypes, and URLs.

---

## 1) The golden rule: 1 post = 1 folder (leaf bundle)

A **leaf bundle** is a folder containing `index.md`. Everything specific to that post lives next to it.

Example:

```

content/posts/hugo/2025-12-16-posts-and-resources-structure/
index.md
cover.jpg
images/
diagram.png
screenshot-1.webp
files/
checklist.pdf

```

Why it’s better:
- you can move/rename the post without breaking 15 image links,
- the post is “portable” (one folder contains everything),
- you can leverage Hugo’s resource pipeline later (depending on theme/templates).

---

## 2) `index.md` vs `_index.md`: get this right or you’ll regret it

### `index.md` (leaf bundle)
- a **final page**
- **no children**
- perfect for **a blog post**

### `_index.md` (branch bundle)
- a **section/list page**
- can contain children (posts, sub-sections)
- perfect for controlling pages like `/posts/` or `/posts/hugo/`

---

## 3) A clean content tree (sections + sub-sections)

I recommend organizing by themes:

```

content/
posts/
_index.md
hugo/
_index.md
2025-12-16-posts-and-resources-structure/
index.md
cover.jpg
images/
files/
ttrpg/
_index.md
2025-12-10-my-campaign/
index.md

````

- `posts/`: your blog section
- `posts/hugo/`: a Hugo-focused sub-section
- `posts/ttrpg/`: another sub-section

---

## 4) What goes into `_index.md` (and why)

`_index.md` controls the section/list page: title, description, ordering, theme params, and often an intro displayed above the list (theme-dependent).

Example `content/posts/_index.md`:

```md
---
title: "Blog"
description: "Notes, write-ups, docs, and tools."
---
Short intro text for the blog section.
````

Example `content/posts/hugo/_index.md`:

```md
---
title: "Hugo"
description: "Conventions, workflows, content modeling, themes, i18n."
weight: 10
---
Practical Hugo notes, focused on building a frictionless writing pipeline.
```

What this gives you:

* control over list pages UX,
* ordering via `weight`,
* context/intros without hacks.

---

## 5) What resources can (and should) live in a post bundle?

### You *can* put:

* **images** (`cover.jpg`, `diagram.png`, `*.webp`)
* **downloads** (`pdf`, `zip`, etc.) under `files/`
* **audio/video** (`mp3`, `mp4`)
* **data** (`json`, `csv`) if consumed by a shortcode/template
* **extra Markdown files** as resources (appendices, fragments)

### You *should* put (minimum “clean” setup):

* `cover.*` if your theme supports a cover/featured image
* `images/` for inline images
* `files/` for downloadable assets

---

## 6) How to reference resources from `index.md`

### Simple (enough to start)

In your post:

```md
![Pipeline diagram](images/diagram.png)

[Download the checklist](files/checklist.pdf)
```

Relative links work nicely because everything is inside the bundle.

### Cleaner (captions / layout control)

Many themes support the `figure` shortcode:

```md
{{< figure src="images/diagram.png" caption="Recommended content structure" >}}
```

---

## 7) `assets/` vs `static/`: where to put global files

This is an architecture decision for your site.

### `static/`

* copied as-is into the generated site
* great for: favicon, robots, verification files, fixed URLs

### `assets/`

* meant for “global resources” and processing (depending on your setup)
* great for: shared resources, images reused across pages, CSS/SCSS, JS, etc.

**Rule of thumb:**

* post-specific → **inside the post bundle**
* shared across the site → **assets/** (or `static/` if you just want a raw copy)

---

## 8) Archetypes: stop rewriting front matter by hand

Keep `archetypes/default.md` as your fallback, and add a dedicated archetype for your post section.

If your section is `posts`:

* create `archetypes/posts.md`

If your section is `post`:

* create `archetypes/post.md`

Example archetype (YAML):

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

Result: every `hugo new content posts/.../index.md` starts consistent.

---

## 9) i18n for `_index.md` (if your site is multilingual)

Two clean approaches (pick one and stick to it):

### A) Filename suffixes

```
content/posts/_index.fr.md
content/posts/_index.en.md
content/posts/hugo/_index.fr.md
content/posts/hugo/_index.en.md
```

### B) One `contentDir` per language

```
content/fr/posts/_index.md
content/en/posts/_index.md
```

In both cases your section pages become properly translated (title + intro + description).

---

## 10) Mini workflow (creation pipeline)

Once you standardize the tree + archetypes, creating a Hugo post becomes muscle memory:

* Create:

  * `.../posts/hugo/YYYY-MM-DD-topic/index.md`
* Add resources:

  * `cover.jpg`, `images/*`, `files/*`
* Publish:

  * set `draft: false`
  * commit + push (or deploy, depending on your setup)

The real value isn’t “Hugo turns Markdown into HTML”.
The value is: **less friction**, **fewer mistakes**, and **a structure that still works after 200 posts**.
