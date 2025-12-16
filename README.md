# HerDev Blog

Personal blog built with **Hugo** and my maintained fork of the **PaperCSS Hugo Theme**.

- Live: https://herdev.hervedarritchon.fr/
- Tech: Hugo (Extended recommended) + PaperCSS theme

---

## Requirements

- **Hugo** (Extended recommended)
    - Check version: `hugo version`
- Git (if using the theme as a submodule)

---

## Quick start (local dev)

```bash
# from the repo root
hugo server -D
````

Useful flags:

```bash
# include drafts + future posts
hugo server -D -F

# bind on local network (test on phone)
hugo server -D --bind 0.0.0.0 --baseURL http://192.168.1.10:1313/

# sometimes helps when tweaking assets/templates
hugo server -D --disableFastRender
```

---

## Build

```bash
# production build (outputs to ./public)
hugo --minify
```

If you want a clean rebuild:

```bash
rm -rf public resources
hugo --minify
```

---

## Create a new post

```bash
hugo new post/my-new-article.md
```

If you use sections other than `post/`, adapt accordingly.

---

## Theme

This blog uses a PaperCSS Hugo theme fork (maintained).

### Install the theme (submodule)

```bash
git submodule add https://github.com/<YOU>/papercss-hugo-theme.git themes/papercss-hugo-theme
git submodule update --init --recursive
```

In your Hugo config:

```yaml
theme: papercss-hugo-theme
```

### Update the theme (submodule)

```bash
git submodule update --remote --merge themes/papercss-hugo-theme
git add themes/papercss-hugo-theme
git commit -m "chore: bump papercss-hugo-theme"
```

---

## Project structure (high level)

Typical layout:

```
.
├── content/                 # blog content (posts, pages, translations…)
├── static/                  # static files copied as-is to the site root
├── assets/                  # processed assets (Hugo Pipes: css/js/images…)
├── themes/papercss-hugo-theme/
└── config.yaml              # (or hugo.yaml / config.toml / config/_default/)
```

---

## Design customization

### Custom CSS

The theme expects customization in:

* `themes/papercss-hugo-theme/assets/css/custom.css`

This is where you override PaperCSS (sizes, spacing, layout width, headings, etc.).

### Fonts

Fonts are served from `static/fonts/...` and declared in:

* `themes/papercss-hugo-theme/assets/css/fonts.css`

In the theme, ensure `head.html` loads it **before** PaperCSS/custom CSS.

---

## Logo in the navbar

Put your logo here:

* `static/logo.png` (or `static/img/logo.png` depending on your choice)

Expose it via config so the theme stays reusable:

```yaml
params:
  navLogo: "/logo.png"     # path under /static
  navLogoAlt: "HerDev"
  navLogoHeight: 32        # optional, px
```

The theme should render the image **left of the title** when `params.navLogo` is set.

---

## Favicons

Place the generated favicon files under `static/` (site root), e.g.:

```
static/
├── apple-touch-icon.png
├── favicon-16x16.png
├── favicon-32x32.png
├── site.webmanifest
└── safari-pinned-tab.svg
```

The theme’s `head.html` links to these.

---

## Footer (“Powered by …”)

Enable/disable via config:

```yaml
params:
  hideFooter: false
```

Expected behavior:

* `hideFooter: false` → footer shows: “Powered by Hugo and PaperCSS” (with links)
* `hideFooter: true` → footer hidden

---

## i18n (multilingual)

Enable languages in your config (example):

```yaml
defaultContentLanguage: en
languages:
  en:
    languageName: English
    weight: 1
  fr:
    languageName: Français
    weight: 2
```

Content patterns you can use:

* Translated content with filename suffix:

    * `content/post/my-article.en.md`
    * `content/post/my-article.fr.md`

Or language folders:

* `content/en/post/...`
* `content/fr/post/...`

The theme includes a minimal language switcher in the navbar when more than one language is configured.

---

## Useful commands (cheat sheet)

```bash
# serve locally
hugo server -D

# build
hugo --minify

# clean build
rm -rf public resources && hugo --minify

# theme submodule update
git submodule update --remote --merge themes/papercss-hugo-theme
```

---

## Licensing

### Content (posts & media)
All blog posts in `content/` and original media in `static/` (unless stated otherwise) are licensed under
**Creative Commons BY-NC-ND 4.0**. See `LICENSE`.

### Code (theme, templates, scripts)
All code in this repository (Hugo layouts, theme customizations, scripts, config) is licensed under the **MIT License**.
See `LICENSE_CODE`.

> Note: The upstream PaperCSS Hugo theme keeps its own license inside `themes/`.

