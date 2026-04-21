## threestoriesdesign

> Single-page lead generation website for **Daughters & Sons Contracting**, a general contracting / renovation & remodeling company based in Brookline, MA, serving the Greater Boston area. Built with Eleventy (11ty) v3 and deployed to Netlify.

# CLAUDE.md — Daughters & Sons Contracting

## Project Overview
Single-page lead generation website for **Daughters & Sons Contracting**, a general contracting / renovation & remodeling company based in Brookline, MA, serving the Greater Boston area. Built with Eleventy (11ty) v3 and deployed to Netlify.

## Commands

```bash
npm start        # Dev server with live reload
npm run build    # Production build → _site/
```

## Architecture

```
src/
├── _data/site.js        # Global data (e.g., current year)
├── assets/css/main.css  # All styles (~1150 lines, CSS custom properties)
├── assets/js/main.js    # Vanilla JS (~241 lines, IIFE pattern)
└── index.njk            # Single Nunjucks template (~648 lines)
_site/                   # Build output (do not edit)
.eleventy.js             # Eleventy config (input: src, output: _site)
netlify.toml             # Build + redirect config
```

## Key Conventions

### CSS
- CSS custom properties (variables) for all colors, spacing, shadows — defined at `:root`
- Mobile-first responsive: breakpoints at 1024px, 768px, 480px
- Color scheme: navy `#1e3a5f` (primary), orange `#f97316` (accent/CTA)
- Fluid typography with `clamp()`

### JavaScript
- Vanilla JS only — no frameworks, ES5-compatible for broad support
- IIFE pattern for scope encapsulation
- Intersection Observer for scroll-triggered animations
- Form submission via `fetch()` to Netlify Forms, with native form fallback

### Templates
- Nunjucks (`.njk`) with front matter for page metadata
- Single-page layout: sections use anchor IDs (`#services`, `#about`, `#contact`)
- Inline SVG icons throughout

## Content
Keep all copy focused on **renovation, remodeling, and general contracting** — not handyman/repairs.

Services: kitchen remodeling, bathroom remodeling, home additions, basement finishing, full home renovation, flooring, carpentry/millwork, decks, windows/doors, painting.

Contact: build@daughtersandsons.com · (617) 555-0100 (placeholder) · Licensed & insured in Massachusetts

Service area: Brookline, Boston, Newton, Cambridge, Somerville, Watertown, Belmont, Arlington, Chestnut Hill, Jamaica Plain, Needham, Wellesley, Lexington, Weston, Medford

## Forms
Two forms (hero + contact), both processed by Netlify Forms:
- `hero-quote` (hero section) and `contact-quote` (contact section)
- Identified by `name` attribute on `<form>` elements
- Client-side validation only (Netlify handles backend validation)
- Success state: hidden div revealed after successful `fetch()` submission

## Deployment
- Platform: Netlify
- All traffic redirects to `/index.html` (SPA-style, configured in `netlify.toml`)
- No backend — Netlify Forms handles all form data

## What to Avoid
- Do not add npm dependencies without good reason — the zero-runtime-dependency approach is intentional
- Do not edit files in `_site/` — it is fully generated on build
- Do not introduce a JS framework; vanilla JS is the deliberate choice here
- Do not use handyman/repair language — this is a GC/remodeling company

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pjuxio) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
