## troy-resume

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static resume portfolio website for a physician executive, deployed at www.troymd.com. Pure HTML5/CSS3/vanilla JavaScript — no frameworks, no bundler, no package.json, zero frontend dependencies.

## Development Commands

**Local dev server** (required for JSON fetch to work):
```bash
cd frontend
python -m http.server 8000
# or: npx serve
```

**CSS minification** (run after any CSS changes):
```bash
cd frontend
node minify-css.js
```
This generates `.min.css` files. Production uses `css/styles.min.css`; development uses `css/styles.css`.

There are no tests, no linter, and no build step beyond optional CSS minification.

## Deployment

GitHub Actions (`.github/workflows/deploy-github-pages.yml`) auto-deploys `frontend/` to GitHub Pages on push to `main`. No build step — the workflow uploads `frontend/` directly using `actions/upload-pages-artifact@v4` and `actions/deploy-pages@v4`.

Custom domain `www.troymd.com` is configured via `frontend/CNAME` and GoDaddy DNS. DNS setup details are in `tasks/transfer-domain.md`.

## Pending owner actions

The migration from Azure to GitHub Pages is in progress (PRD: `tasks/prd-azure-to-github-pages.md`). Local file changes are complete. The following require manual action by the repo owner:

- **GitHub**: Rename repo from `cloud-resume-azure` to `troy-resume`, update local git remote URL
- **GitHub**: Enable Pages in repo Settings > Pages, set source to "GitHub Actions"
- **GitHub**: Remove `AZURE_STATIC_WEB_APPS_API_TOKEN` secret from repo settings
- **GoDaddy DNS**: Update A records and `www` CNAME to point to GitHub Pages IPs (see `tasks/transfer-domain.md`)
- **GitHub**: Configure custom domain `www.troymd.com` in Pages settings, enable "Enforce HTTPS"
- **Azure Portal**: Delete Azure Static Web Apps resource and `sybertresume` storage account

## Architecture

### Data-driven rendering

`frontend/index.html` is a minimal semantic HTML shell. On `DOMContentLoaded`, the `ResumeManager` class (`frontend/js/resume.js`) fetches six JSON files from `frontend/data/` in parallel via `Promise.all`, then renders each resume section into the DOM using template literals.

Data files: `profile.json`, `roles.json`, `credentials.json`, `publications.json`, `volunteer.json`, `technology.json`.

### Key classes

- **ResumeManager** (`frontend/js/resume.js`, ~816 lines) — data loading, section rendering (profile, technology, work experience, credentials, publications, volunteer), Gantt chart timeline, category filtering, modal system, hash-based navigation, scroll-based active nav highlighting.
- **Chatbot** (`frontend/js/chatbot.js`, ~179 lines) — FAQ chatbot with keyword matching against `frontend/data/faq.json`. Toggle UI, typing indicator, template question buttons.

### CSS structure

`frontend/css/styles.css` imports 8 modular files: `variables.css` (custom properties/colors), `base.css`, `header.css`, `sections.css`, `timeline.css`, `gantt.css`, `chatbot.css`, `responsive.css`. Work experience categories use color coding: administrative (blue), clinical (green), academic (purple) — defined as CSS variables.

### Gantt chart

The timeline visualization spans 2000-2025. Position/width calculations are documented in `docs/gantt-logic.md`. Gantt metadata (`gantt_position`, `gantt_width`) lives in `roles.json` entries.

## Conventions

- All source files kept under 500 lines.
- JSON keys use snake_case; JavaScript uses camelCase.
- No external fonts — system font stack only.
- Semantic HTML5 elements (`header`, `nav`, `main`, `section`) with ARIA labels on interactive elements.
- Hash-based navigation (`#summary`, `#technology`, `#work`, etc.) — no router library.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HacksterT) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
