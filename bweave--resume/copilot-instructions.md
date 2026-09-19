## resume

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — build then serve `dist/` at http://localhost:8888/resume.html, watching `resume.md` and `styles/resume.css` for auto-rebuild (HTML + PDF).
- `npm run build` — build both HTML and PDF into `dist/`.
- `npm run build:html` / `npm run build:pdf` — build one format. `build:pdf` invokes `build:html` first.

There is no test suite or linter wired into npm scripts (an `eslint.config.js` exists but is not run via npm).

## Architecture

This is a static resume pipeline: `resume.md` + `styles/resume.css` → `dist/resume.html`, `dist/index.html`, `dist/resume.pdf`.

- **`scripts/build-html.js`** — the source of truth for output structure. Uses `markdown-it` to render `resume.md`, then applies regex post-processing to convert specific Markdown shapes into structured HTML the CSS depends on:
  - `### Company | Location` → `<h3 class="company-header">` with `.company` / `.location` spans.
  - `#### Role, Date` → `<h4 class="role-header">` with `.role` / `.date` spans.
  - Each `h4.role-header` plus the following `<ul>` is wrapped in `<div class="role-section">`.
  - YAML-ish frontmatter (`---` block with `title:`) is stripped and used for `<title>`.
  - CSS is inlined into a `<style>` tag (single self-contained HTML file). Both `dist/resume.html` and `dist/index.html` are written (the latter for GitHub Pages).
  - **If you change the Markdown conventions for company/role headings, update both the regexes here and the matching selectors in `styles/resume.css`.**

- **`scripts/build-pdf.js`** — runs `build-html.js`, then loads `dist/resume.html` into headless Puppeteer and prints A4 with 0.5in margins to `dist/resume.pdf`.

- **`scripts/serve.js`** — initial full build, then `fs.watchFile` on `resume.md` and `styles/resume.css` triggers a rebuild of both HTML and PDF on every change; `npx http-server dist` serves the output.

- **`.github/workflows/build-resume.yml`** — on push to `main` (or PR) touching `resume.md`, `styles/**`, or `scripts/**`: builds both formats, uploads artifacts (30-day retention), deploys `dist/` to the `gh-pages` branch (live at https://blog.bweave.dev/resume/), and creates a dated GitHub release.

## Conventions

- Output is committed-ignored; only `resume.md`, `styles/`, and `scripts/` are source. The deployed site lives on the `gh-pages` branch.
- Keep the HTML self-contained (CSS inlined) — the PDF generator and GitHub Pages both rely on a single-file artifact.

---
> Source: [bweave/resume](https://github.com/bweave/resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-18 -->
