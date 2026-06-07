## mugeshk97-github-io

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

Personal portfolio site at https://mugeshk97.github.io — a single static `index.html` with all CSS and SVG inlined. No build step, no JS framework, no test suite, no package manager. The repo is also a GitHub Pages site (the repo name `mugeshk97.github.io` makes it the user's root Pages site).

## Local development

Open `index.html` directly in a browser, or serve the directory:

```
python3 -m http.server 8000   # then visit http://localhost:8000
```

There is nothing to build, lint, or test. Changes are visible on reload.

## Deployment

`.github/workflows/deploy.yml` deploys on every push to `main` (and via `workflow_dispatch`). It rsyncs the repo (excluding `.git`, `.github`, `_site`, `CLAUDE.md`) into `_site/` and uploads it as the Pages artifact — meaning **any other file you add at the repo root ships to production.** Don't commit drafts, scratch files, or local configs to root; add them to the rsync exclude list in `deploy.yml`, put them under `.github/`, or `.gitignore` them.

Pages concurrency group is `pages` with `cancel-in-progress: false`, so consecutive pushes queue rather than cancel.

## Architecture & conventions

Everything lives in `index.html`:

- **Design tokens** are CSS custom properties in `:root` (`--bg`, `--surface`, `--card`, `--border`, `--ink`, `--muted`, `--accent`, `--accent2`) plus font shortcuts `--f` (Fraunces serif, used for display headings — often with `<em>` styled as italic accent), `--s` (Inter, body), `--m` (JetBrains Mono, used for small uppercase labels with letter-spacing). When adding UI, use these variables rather than hex literals so theme changes stay in one place.
- **Section structure** is a repeated pattern: `<section id="…">` with a `.s-head` (left: `.s-num` kicker + `<h2>` with italic `<em>` accent; right: descriptive paragraph) followed by a grid. Match this when adding new sections so spacing and rhythm stay consistent.
- **Project visuals are inline SVG**, not images — see the `.proj-visual` blocks in `#work`. Project rows alternate via `.proj-row.flip` which swaps the visual/info order. Keep visuals as inline SVG (no external assets) so the site stays single-file and ships without a CDN dependency.
- **Responsive breakpoint** is a single `@media(max-width:900px)` block at the bottom of the `<style>`. All grid collapses and padding overrides for mobile go there.
- **External resources** are only Google Fonts (Fraunces / Inter / JetBrains Mono via `fonts.googleapis.com`). Don't add JS bundles, build tooling, or other CDN dependencies without a strong reason — the value of this repo is that it's auditable in one file.

## Contact info baked into the page

Email (`mugeshk1171@gmail.com`), LinkedIn (`linkedin.com/in/mugeshk97`), and GitHub (`github.com/mugeshk97`) appear in multiple places (nav CTA, project links, contact cards, footer). When updating, grep for all occurrences — it's easy to miss one.

---
> Source: [mugeshk97/mugeshk97.github.io](https://github.com/mugeshk97/mugeshk97.github.io) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-07 -->
