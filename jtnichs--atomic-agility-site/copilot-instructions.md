## atomic-agility-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Static marketing site for Atomic Agility — pure HTML5, CSS3, and vanilla JavaScript. No build process, no frameworks, no package manager.

## Local Development

Preview the site locally using Python's built-in HTTP server (required to avoid CORS issues with relative paths):

```bash
python -m http.server 8000
# Then open http://localhost:8000
```

## Deployment

Hosted on GitHub Pages with a custom domain (`CNAME` file). Pushing to the main branch deploys automatically.

## Architecture

**Routing**: Folder-based clean URLs. Each page lives in its own directory as `index.html`.
- `/` → `index.html`
- `/about/` → `about/index.html`
- `/services/` → `services/index.html`
- `/resources/` → `resources/index.html`
- `/contact/` → `contact/index.html`

**Styling**: Single `styles.css` file shared across all pages. Uses Flexbox and CSS Grid. Responsive breakpoints at 1100px and 700px.

**JavaScript**: Single `assets/site.js` (IIFE pattern) handles mobile menu toggle, keyboard accessibility (ESC to close), and scroll-based header styling. Loaded at the bottom of every page's `<body>`.

**Forms**: Contact form POSTs to Formspree (`https://formspree.io/f/mykdwknk`). Includes honeypot spam protection via a hidden `_gotcha` field.

## Conventions

**Page template**: Every page shares the same header (`.brand`, `.nav-desktop`, hamburger button) and footer structure. When adding or editing nav links, update all pages.

**Color palette** (defined in `styles.css`):
- Background: radial gradient `#0e2a4a` → `#061a2f` → `#041322`
- Body text: `#e0f2fe`
- Accent / CTA: `#38bdf8` (cyan)
- Secondary text: `#94a3b8`

**CSS class conventions**:
- `.section` — full-width content sections (padding: 4rem 2rem)
- `.card` — glassmorphic content cards
- `.metrics-grid` — 4-col grid, collapses to 2 then 1 on mobile
- `.quote-grid` — 3-col testimonial grid
- `.cta-button` — primary cyan call-to-action buttons

---
> Source: [jtnichs/atomic-agility-site](https://github.com/jtnichs/atomic-agility-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
