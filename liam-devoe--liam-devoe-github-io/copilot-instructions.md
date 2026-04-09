## liam-devoe-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Pelican-based personal blog hosted on GitHub Pages at tybug.dev.

## Build & Serve

```bash
pip install -r requirements.txt       # Install Python dependencies
pelican --listen                       # Build and serve at http://127.0.0.1:8000/
pelican content -s publishconf.py --fatal=errors  # Production build (used by CI)
```

Deployment is via GitHub Actions on push to master.

## Architecture

- **`content/articles/`** — Blog posts in Markdown with Pelican metadata header (Title, Date, Tags, Slug, etc.).
- **`content/thoughts/`** — Short-form posts. Files must be numbered (`1.md`, `2.md`, etc.). Only `Date` metadata is required. Rendered at `/thoughts/<id>/` via `thought.html`, with an index at `/thoughts/` via `thoughts.html`.
- **`content/pages/`** — Static pages (e.g., `about.md`, `contracting.html`).
- **`content/extra/`** — Files copied as-is to output (CNAME, PDF).
- **`theme/templates/`** — Jinja2 templates: `base.html` (shell with persistent header), `article.html` (post), `articles.html` (full article listing at `/articles/`), `index.html` (home — recent articles + recent thoughts), `tag.html` (tag listing), `page.html` (static pages), `thought.html`/`thoughts.html` (thoughts).
- **`theme/static/css/`** — SCSS source files (`theme.scss` imports `_rouge.scss` for syntax highlighting), compiled to CSS at build time by the webassets plugin. Color palette is defined as SCSS variables at the top of `theme.scss`.
- **`theme/static/js/mathjax/`** — Vendored MathJax for LaTeX math rendering.
- **`plugins/`** — Custom Pelican plugins:
  - `hidden_articles.py` — Articles with `Hidden: true` are accessible at their URL but excluded from the index and tag listings.
  - `redirect_generator.py` — Generates HTML redirect pages from the `REDIRECTS` dict in `pelicanconf.py` (maps old date-based URLs to current slugs).
  - `sidenotes.py` — Transforms Markdown footnotes (`[^1]`) into margin sidenotes at build time. Footnote references get a `<span class="sidenote">` injected after them; the bottom footnote section is removed.
  - `thoughts.py` — Separates thought files from regular articles, assigns them `/thoughts/<id>/` URLs, and generates the thoughts index page. Puts `thoughts` in the Jinja context.
- **`pelicanconf.py`** — Dev config. **`publishconf.py`** — Production config (sets SITEURL).

## Site Structure

- **Persistent header** on every page: site name (left) + nav links `articles | thoughts | about` (right). Active nav is set by JS in `base.html` matching `location.pathname`.
- **Homepage** (`index.html`): shows first 5 articles + last 3 thoughts. Generated as a `DIRECT_TEMPLATES` entry.
- **Articles listing** (`articles.html`): full article list at `/articles/`. Also a `DIRECT_TEMPLATES` entry.
- **Individual articles** at `/<slug>/`. On wide screens (≥62em), dates appear in the left margin (CSS Grid) and footnotes appear as sidenotes in the right margin (float with negative margin).

## Article Metadata

Always use `---` fences for front matter in Markdown files.

```
---
Title: Post Title
Date: 2024-01-15
Tags: python, math
Slug: post-slug
---
```

Optional: `Hidden: true` (accessible but unlisted), `Status: draft` (not published).

## Key Conventions

- MathJax is enabled globally (vendored, not CDN): use `$...$` for inline math, `$$...$$` for display math (via pymdownx.arithmatex).
- Python-Markdown with footnotes, fenced_code, codehilite, tables extensions. Footnotes are converted to sidenotes by the `sidenotes` plugin.
- Font: Source Serif 4 via Google Fonts. SCSS color variables: `$color-body`, `$color-secondary`, `$color-link`, etc. at the top of `theme.scss`.
- No JavaScript frameworks — plain HTML/CSS with minimal JS (MathJax + active nav highlight).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Liam-DeVoe)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Liam-DeVoe)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
