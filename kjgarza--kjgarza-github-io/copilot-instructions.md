## kjgarza-github-io

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

This is an [Eleventy (11ty)](https://www.11ty.dev/) static site. Use **Bun** as the package manager and runtime.

```bash
bun run dev        # Start dev server with live reload (http://localhost:8080)
bun run build      # Build for production → _site/
bun run watch      # Watch and rebuild without serving
bun run clean      # Remove _site/ and node_modules/
bun run reinstall  # Clean + reinstall dependencies
```

Via `just` (requires [just](https://github.com/casey/just)):
```bash
just dev           # Same as bun run dev
just build         # Same as bun run build
just serve         # Serve _site/ with python3 http.server on :8080
```

No linter or test suite is configured.

## Architecture

### Tech stack
- **Eleventy 3.x** with **Nunjucks** as the primary template engine (`autoescape: false`)
- CSS is pre-built and served as a static asset from `src/_next/static/css/` (Tailwind output, not compiled during build)
- No JavaScript bundler — scripts are static files in `src/assets/`

### Directory layout

```
src/
  _data/          # Global data files (JS modules) — auto-injected into all templates
  _includes/
    layouts/      # Page layouts (base.njk, case-study.njk)
    components/   # Partials (nav.njk, footer.njk, work-card.njk, tool-card.njk)
  assets/         # Images, static CSS/JS — passthrough copied to _site/
  drawings/       # SVG decorations — passthrough copied to _site/
  _next/          # Pre-built CSS bundle — passthrough copied to _site/
  static/api/     # Static JSON API responses — copied to _site/api/
  work/           # Case study markdown files (one per project)
  *.njk           # Top-level pages: index, work, tools, publications, playground
_site/            # Build output (gitignored)
```

### Data layer (`src/_data/`)

All files export plain JS arrays/objects consumed as global template variables:

| File | Variable | Purpose |
|------|----------|---------|
| `site.js` | `site` | Author info, bio, GA code, schema.org |
| `work.js` | `work` | Portfolio items with metadata for the grid |
| `tools.js` | `tools` | Side-project tools shown on homepage + /tools |
| `publications.js` | `publications` | Academic publications list |
| `navigation.js` | `navigation` | Nav links array |

### Pages and routing

- **`/`** (`index.njk`) — Homepage: hero text, bio sections, featured work grid, tools preview
- **`/work`** (`work.njk`) — All work items from `work.js`
- **`/work/:slug/`** — Individual case studies from `src/work/*.md`, using `layout: layouts/case-study.njk`
- **`/tools`** (`tools.njk`) — All tools from `tools.js`
- **`/publications`** (`publications.njk`) — All publications from `publications.js`

### Adding a new case study

1. Add an entry to `src/_data/work.js` with `id`, `title`, `description`, `tags`, `status`, `link`, `featured`, and optional `image`/`gridClass` fields
2. Create `src/work/<slug>.md` with frontmatter: `layout: layouts/case-study.njk`, `title`, `description`, `company`, `tags`, `heroImage`, `permalink`, and optionally `passwordProtected: true`

### CSS / Styling

Tailwind utility classes are used throughout templates but the CSS is pre-compiled — **do not run a Tailwind build step**. The compiled bundle lives at `src/_next/static/css/b79f6ce842955dc5.css` and is referenced directly from `base.njk`. To change styles, edit the bundle or regenerate it externally.

### Static API passthrough

`src/static/api/` is copied verbatim to `_site/api/`, enabling static JSON endpoints (e.g. `/api/user.json`).

### Contact form

The "Get in touch" button uses [Tally](https://tally.so) embedded forms (`data-tally-open="3XjAKz"`). The Tally embed script is loaded in `base.njk`.

---
> Source: [kjgarza/kjgarza.github.io](https://github.com/kjgarza/kjgarza.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
