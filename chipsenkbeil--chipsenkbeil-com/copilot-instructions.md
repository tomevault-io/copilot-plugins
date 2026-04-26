## chipsenkbeil-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Personal website for Chip Senkbeil (chipsenkbeil.com), built with Hugo using the `hugo-coder` theme. Content is written in Org-mode format. The site is a pure static site with no JavaScript.

## Build Commands

- `make serve` — Run local dev server with draft support
- `make build` — Production build
- `make clean` — Remove generated output in `public/`
- `make all` — Clean + build (default target)
- `make push` — Clean, build, and deploy to gh-pages branch
- `make new-post TITLE="..." DESCRIPTION="..." CATEGORIES="..." TAGS="..."` — Create new blog post
- `make new-note TITLE="..." DESCRIPTION="..." CATEGORIES="..." TAGS="..."` — Create new note
- `make help` — Show all available targets

Hugo version: 0.147.5 (extended). CI uses the same version via GitHub Actions.

## Content Structure

All content uses **Org-mode** (`.org`) format with this front matter pattern:

```org
#+TITLE: Post Title
#+SLUG: post-slug
#+DESCRIPTION: Short description
#+DATE: YYYY-MM-DD
#+CATEGORIES[]: cat1 cat2
#+TAGS[]: tag1 tag2
```

Content files are sequentially numbered (`001-`, `002-`, etc.):
- `content/posts/` — Blog posts (long-form articles)
- `content/notes/` — Quick reference notes
- `content/about.org` and `content/miscellaneous.org` — Standalone pages

## Architecture

- **config.toml** — Hugo site configuration (theme, taxonomies, social links, menus)
- **themes/hugo-coder/** — Theme (git submodule from luizdepra/hugo-coder)
- **layouts/partials/footer.html** — Only theme override: custom footer with copyright and PGP key link
- **static/** — Static assets (images, favicons, PGP keys, CNAME)
- **public/** — Build output (git submodule pointing to gh-pages branch)
- **scripts/** — Utility scripts (md2org conversion tooling from a past migration)

## Deployment

- GitHub Actions (`.github/workflows/deploy.yml`) builds on push to `master` and deploys to gh-pages
- Local deploy via `make push` force-pushes built output to the `gh-pages` branch
- Custom domain configured via CNAME file (`chipsenkbeil.com` and `www.chipsenkbeil.com`)

## Key Configuration Choices

- Goldmark renderer has `unsafe = true` to allow raw HTML in content
- `canonifyurls = true` (legacy option, noted as needing future replacement in config)
- Color scheme is `auto` (system preference), JS is disabled (`disableDefaultJsScripts = true`)
- Taxonomies: categories, series, tags, authors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/chipsenkbeil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
