## personalwebsite

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

One Jekyll repository producing three static sites:

| Domain | Purpose | Theme | Config |
|--------|---------|-------|--------|
| gmacko.com | Build-in-public hub, startups, blog, resume | Warm editorial, burnt sienna | `_config.gmacko.yml` |
| grahammackie.com | Personal blog, social links | Warm editorial, slate blue | `_config.personal.yml` |
| gmac.io | Prototype & service dashboard | Dark mode, neon purple | `_config.gmac.yml` |

## Key Technologies

- **Jekyll**: Static site generator (Ruby-based)
- **SCSS/CSS Custom Properties**: Themeable design system via `:root` variables
- **Nix/ForgeGraph**: Deployment (planned, currently Docker/Nginx)
- **Markdown**: Content authoring for blog posts

## Building

Each site builds with Jekyll multi-config merge:

```bash
# gmacko.com
bundle exec jekyll build --config _config.yml,_config.gmacko.yml --destination _site_gmacko

# grahammackie.com
bundle exec jekyll build --config _config.yml,_config.personal.yml --destination _site_personal

# gmac.io
bundle exec jekyll build --config _config.yml,_config.gmac.yml --destination _site_gmac
```

For local development with live reload:
```bash
bundle exec jekyll serve --config _config.yml,_config.gmacko.yml --destination _site_gmacko
```

## Project Structure

```
_config.yml              # Shared base config (timezone, markdown, owner)
_config.gmacko.yml       # gmacko.com overrides (url, title, site_id, excludes)
_config.personal.yml     # grahammackie.com overrides
_config.gmac.yml         # gmac.io overrides

DESIGN.md                # Shared design system (typography, spacing, motion)
DESIGN.gmacko.md         # gmacko.com palette
DESIGN.personal.md       # grahammackie.com palette
DESIGN.gmac.md           # gmac.io palette (dark + neon)

_layouts/
  default.html           # Shared base layout (sidebar + content)
  landing.html           # gmacko.com homepage
  venture.html           # gmacko.com venture detail pages
  personal.html          # grahammackie.com sidebar + blog feed
  dashboard.html         # gmac.io standalone dark dashboard

_sass/
  components/_variables.scss   # CSS custom properties on :root + Sass backwards compat
  _theme-personal.scss         # Blue accent override for personal site
  _theme-dashboard.scss        # Dark theme override for dashboard
  pages/_personal.scss         # Personal sidebar layout styles
  pages/_dashboard.scss        # Dashboard grid/card styles

_data/
  ventures.yml           # Startup portfolio (gmacko.com)
  gmac/services.yml      # Live services (gmac.io)
  gmac/prototypes.yml    # Experimental prototypes (gmac.io)
  index/                 # Resume data (careers, education, skills)

pages/
  gmac/index.html        # gmac.io dashboard page
  personal/index.html    # grahammackie.com blog feed page
```

## Content Management

### Theming

All colors flow through CSS custom properties defined in `_variables.scss`. New code should use `var(--token)`, not `$sass-variable`. Key tokens: `--bg`, `--surface`, `--text`, `--text-strong`, `--muted`, `--accent`, `--accent-hover`, `--rule`, `--rule-heavy`.

### Content Filtering

Posts are filtered by `site_id` in templates using Liquid:
```liquid
{% assign site_posts = site.posts | where_exp: "p", "p.site contains site.site_id" %}
```

Posts need a `site` front matter field to appear on the correct site:
```yaml
site: gmacko    # or: personal, gmac, or [gmacko, personal] for cross-posting
```

### Site Configuration

Each site config sets `site_id` and `exclude` to prevent content leaking between sites. The base `_config.yml` holds shared settings. Site configs override via Jekyll's `--config` merge.

## Design System

Always read DESIGN.md before making any visual or UI decisions.
Per-site palettes are in DESIGN.gmacko.md, DESIGN.personal.md, DESIGN.gmac.md.
All font choices, colors, spacing, and aesthetic direction are defined there.
Do not deviate without explicit user approval.

## Deployment

Hosted on Cloudflare Pages — three projects (`gmacko`, `personal`, `gmac`), each pointed at this repo via git integration with its own build command and output directory:

- `gmacko` → gmacko.com — `--config _config.yml,_config.gmacko.yml --destination _site_gmacko`
- `personal` → grahammackie.com — `--config _config.yml,_config.personal.yml --destination _site_personal`
- `gmac` → gmac.io — `--config _config.yml,_config.gmac.yml --destination _site_gmac`

Deploys are manual via `./scripts/deploy-pages.sh <gmacko|personal|gmac|all>`, which builds with the matching `_config.*.yml` and runs `wrangler pages deploy` against the corresponding project. Requires `wrangler login` once.

---
> Source: [gmackie/personalWebsite](https://github.com/gmackie/personalWebsite) — distributed by [TomeVault](https://tomevault.io/claim/gmackie).
<!-- tomevault:4.0:copilot_instructions:2026-04-18 -->
