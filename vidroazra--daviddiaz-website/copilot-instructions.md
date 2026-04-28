## daviddiaz-website

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Personal website for David Díaz (GitHub: Vidroazra). A zero-dependency, single-file static site — no build tools, no frameworks, no package manager.

## Development

Open `index.html` directly in a browser. No build or install step required.

For live reload during development, any static file server works:

```bash
# Python (usually available)
python -m http.server 8080

# Node (if available)
npx serve .
```

## Architecture

Everything lives in `index.html` — CSS, HTML, and JS are all inline in a single file. This is intentional; keep it that way unless the complexity clearly warrants splitting.

**Section order in `index.html`:**
1. `<head>` — all CSS in a single `<style>` block, using CSS custom properties defined in `:root`
2. `<nav>` — sticky, fixed, with a JS-powered mobile hamburger
3. `<main>` — five `<section>` elements: `#hero`, `#about`, `#projects`, `#blog`, `#contact`
4. `<footer>`
5. `<script>` — only the mobile nav toggle (no dependencies)

**Design system** (CSS custom properties in `:root`):

| Variable | Value | Role |
|---|---|---|
| `--bg` | `#0f1117` | Page background |
| `--surface` | `#1a1d27` | Card backgrounds |
| `--border` | `#2a2d3d` | Card/divider borders |
| `--accent` | `#6c63ff` | Primary purple accent |
| `--text-primary` | `#f0f0f5` | Body text |
| `--text-secondary` | `#8b8fa8` | Muted/label text |

## Blog — The Observing Human

"The Observing Human" (`https://theobservinghuman.substack.com/`) is **not** an engineering or tech blog. It covers technology, science, literature, philosophy, photography, and broader reflections on life — anything connected to the act of paying attention to the world. When writing copy for the blog section, never reduce it to a tech newsletter. The tone is personal and essayistic, not informational.

## Roadmap

See `personal_web_page_plan.md` for the full 6-phase development plan. The next planned phases are:
- **Phase 2:** Replace placeholder project cards with real GitHub repos
- **Phase 3:** Substack RSS feed embed (latest posts from `https://theobservinghuman.substack.com/feed`)
- **Phase 4:** GitHub API for live repo activity (`api.github.com/users/Vidroazra/repos`)
- **Phase 6:** Deploy via GitHub Pages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vidroazra) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
