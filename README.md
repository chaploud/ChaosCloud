# Chaos Cloud

Hugo + Blowfish theme, deployed to GitHub Pages.
Live: https://chaploud.github.io/ChaosCloud/

## Stack

- Hugo 0.161.1 extended
- Blowfish v2 (Hugo Module — pinned in `go.mod`)
- Deploy: GitHub Actions → GitHub Pages (Actions source, no `gh-pages` branch)

## Directory layout

| Path | Role |
|---|---|
| `config/_default/hugo.toml` | Site core (baseURL, title, taxonomies, outputs, sitemap) |
| `config/_default/languages.en.toml` | Language title, description, author block |
| `config/_default/params.toml` | Blowfish theme params (homepage, article, list, etc.) |
| `config/_default/menus.en.toml` | Header / footer menu entries |
| `config/_default/markup.toml` | Markdown / code highlight |
| `config/_default/module.toml` | Theme module import |
| `content/posts/<slug>/index.md` | Blog post (page bundle — drop images alongside) |
| `content/about.md` | Standalone fixed page (not in `/posts/`, not in RSS) |
| `static/` | Files served as-is at site root (favicons, CNAME, etc.) |
| `assets/` | Pipeline-processed assets (Hugo Pipes) |
| `layouts/` | Local template overrides (empty → using theme defaults) |
| `archetypes/default.md` | Front matter template for `hugo new content` |
| `.github/workflows/hugo.yaml` | Build + deploy workflow |
| `go.mod` / `go.sum` | Hugo Module deps (Blowfish version pinned here) |

## Taxonomies

**Tags only.** Categories are intentionally disabled.

## Add a new post

```sh
hugo new content posts/<slug>/index.md
# edit content/posts/<slug>/index.md
hugo server -D    # local preview with drafts
git add . && git commit -m "post: <title>" && git push
```

Push to `main` → workflow builds and deploys (~40 sec total).

### Post front matter

```toml
+++
date = '2026-05-25T23:17:13+09:00'
draft = false
title = 'Post Title'
summary = 'One-line summary for list view and meta description.'
tags = ['clojure', 'gc']
+++
```

Optional:
- `series = ['interpreter-series']`
- `featured_image = 'cover.jpg'` (alongside `index.md` in the bundle)
- `showHero = true` / `heroStyle = "big"` to override theme default per article

## Add a fixed page (e.g. `/about/`)

```sh
hugo new content about.md
```

Suppress article chrome in front matter:

```toml
+++
title = 'About'
showDate = false
showReadingTime = false
showAuthor = false
showWordCount = false
showTableOfContents = true
sharingLinks = []
+++
```

Add to header menu in `config/_default/menus.en.toml`:

```toml
[[main]]
  name = "About"
  pageRef = "about"
  weight = 40
```

## Drafts

- `draft = true` keeps the post out of the build.
- Prefer not pushing draft branches (this repo is public).

## Local development

```sh
hugo server          # http://localhost:1313/
hugo server -D       # include drafts
hugo --gc --minify   # production build to ./public
```

## Update Blowfish

```sh
hugo mod get -u github.com/nunocoracao/blowfish/v2
hugo mod tidy
```

When new theme params are added upstream, refresh defaults:
https://github.com/nunocoracao/blowfish/releases/latest/download/config-default.zip

## Customize appearance

- Color scheme: `colorScheme` in `params.toml` (e.g. `blowfish`, `avocado`, `fire`, `ocean`, `forest`, `princess`, `neon`, `bloody`, `terminal`, `marvel`, `noir`, `autumn`, `congo`, `slate`)
- Homepage layout: `[homepage].layout` in `params.toml` (`profile` / `page` / `hero` / `card` / `background` / `custom`)
- Custom CSS: `assets/css/custom.css`
- Per-section template override: `layouts/<kind>/<layout>.html` (only when really needed)
