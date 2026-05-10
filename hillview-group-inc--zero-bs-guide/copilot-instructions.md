## zero-bs-guide

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Build the static site (outputs to dist/)
npm run build

# Clean build output
npm run clean

# Install API dependencies
cd api && npm install
```

There are no tests. There is no dev server — build and open `dist/index.html` in a browser, or use the VS Code Live Server extension on port 5501.

## Architecture

This is a **custom static site generator** built with plain Node.js — no framework. It pre-renders HTML and deploys to Azure Static Web Apps with a serverless Azure Functions API backend.

### Build pipeline (`scripts/build-static.js`)

The build script:
1. Reads content from `data/articles.json` and `data/faqs.json`
2. Loads HTML templates from `views/` (using `{{PLACEHOLDER}}` substitution)
3. Generates individual article and FAQ pages from `views/article-template.html` and `views/faq-template.html`
4. Generates `robots.txt` and `sitemap.xml` (requires `BASE_URL` env var)
5. Copies `public/` assets (CSS, JS, images) to `dist/`

All output lands in `dist/`, which is the Azure Static Web Apps output location.

### API (`api/`)

A single Azure Function at `api/contact/` handles the contact form:
- Validates and sanitizes input server-side
- Sends email via `nodemailer`
- Requires env vars: `SMTP_HOST`, `SMTP_PORT`, `SMTP_SECURE`, `SMTP_USER`, `SMTP_PASS`

### Environment variables

Copy `.env.example` to `.env` for local development. Required vars:
- `BASE_URL` — used for sitemap.xml generation
- SMTP vars above for the contact API

### Routing and headers

`staticwebapp.config.json` defines all routing rules, security headers (CSP, X-Frame-Options, etc.), cache-control for static assets, and the 404 fallback to `/404.html`.

### Deployment

GitHub Actions (`.github/workflows/azure-static-web-apps.yml`) deploys automatically on push. Azure Static Web Apps serves `dist/` as the frontend and `api/` as the serverless backend.

---
> Source: [Hillview-Group-Inc/zero-bs-guide](https://github.com/Hillview-Group-Inc/zero-bs-guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
