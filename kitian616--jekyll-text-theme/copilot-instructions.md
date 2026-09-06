## jekyll-text-theme

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build the production site
npm run build

# Serve locally with live reload (default)
npm run serve

# Serve with tracing output
npm run default

# Serve docs/demo site (dev, beta, production variants)
npm run dev          # docs/_config.dev.yml
npm run demo-dev     # docs/_config.yml (dev env)
npm run demo-prod    # docs/_config.yml (production env)

# Lint JS in _includes/**/*.js
npm run eslint
npm run eslint-fix

# Lint SCSS in _sass/**/*.scss
npm run stylelint
npm run stylelint-fix

# Build & publish gem (release flow only)
npm run gem-build
npm run gem-push

# Docker-based development
npm run docker-dev:default
npm run docker-dev:dev
npm run docker-dev:demo-dev

# Docker production build + serve
npm run docker-prod:build
npm run docker-prod:serve
```

**CI**: Travis builds the docs site via `JEKYLL_ENV=production bundle exec jekyll build --config ./docs/_config.yml` and deploys to `gh-pages` branch.

**Commit style**: Conventional Commits via commitlint (`feat:`, `fix:`, `docs:`, `refactor:`, `release:`, etc.).

## Project Architecture

### Skin System
6 built-in skins (`default`, `dark`, `forest`, `ocean`, `chocolate`, `orange`) + 5 highlight themes. A skin is a SCSS file at `_sass/skins/_<name>.scss` that defines CSS custom properties (colors, fonts, borders). Selected via `text_skin` in `_config.yml`. The main entry point `assets/css/main.scss` dynamically `@import`s the active skin.

### Layout Inheritance
Layouts in `_layouts/` form a chain: `none` ← `base` ← `page` ← `article` / `home` / `landing` / `articles` / `archive` / `404`. `base.html` is the root — it sets up the HTML shell, analytics, head, and core JS utilities. Page-level layouts extend it and add content wrappers.

### Include System
`_includes/` is organized by concern:
- **article/** — header, footer, info, list components
- **aside/** — sidebar content (TOC, affix)
- **scripts/** — vanilla JS (no framework): lib/ (third-party), utils/, components/ (search, lightbox, sidebar)
- **head/**, **footer/**, **sidebar/** — structural includes
- **comments-providers/**, **analytics-providers/**, **search-providers/**, **sharing-providers/**, **pageview-providers/** — pluggable third-party integrations

### JavaScript Architecture
Vanilla JS (no build step, no framework). Scripts are included directly via Liquid `{%- include scripts/...js -%}` in `base.html`. The JS is split into:
- **lib/** — isolated utilities (lazyload, throttle, affix, TOC, swiper, modal, gallery, scroll-to)
- **components/** — stateful UI (search, lightbox, sidebar)
- **utils/** — helpers (image loading, general utils)
- Page-level scripts: `article.js`, `page.js`, `home.js`, `archieve.js`

No bundler — all JS is served raw from `_includes/scripts/`.

### SCSS Organization
`_sass/` is structured as:
- **skins/** — color/font themes (each skin is a standalone variable file)
- **common/** — shared variables, functions, classes, reset, print styles, reusable components (button, card, modal, gallery, etc.)
- **components/** — page-level component styles (header, footer, search, lightbox, etc.)
- **layout/** — page layout styles (base, page, article, home, archive, etc.)
- **additional/** — optional utility styles (alerts, tags, photo frames)
- **animate/** — keyframe animations (fade-in variants)
- **custom.scss** — user overrides placeholder

### Content & Data
- `_data/` — YAML files for authors, licenses, locale (i18n), navigation, and theme variables
- `_posts/` — sample/blog posts directory
- `docs/` — standalone documentation site (has its own `_config.yml`, posts, sample pages, and shared `_includes` from root via Travis `before_script`)
- `test/` — minimal test site (used for smoke testing the theme)

### Building a Release
1. Update version in `jekyll-text-theme.gemspec`, `package.json`, and `_includes/scripts/variables.html`
2. Update `CHANGELOG.md`
3. Run `npm run gem-build && npm run gem-push`
4. Tag and push with `release: vx.x.x` commit message

---
> Source: [kitian616/jekyll-TeXt-theme](https://github.com/kitian616/jekyll-TeXt-theme) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-06 -->
