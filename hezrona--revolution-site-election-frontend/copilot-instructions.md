## revolution-site-election-frontend

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Start dev server on port 5173
npm run build      # Production build to dist/
npm run preview    # Serve the built dist/ on port 4173
npm run prerender  # Puppeteer static pre-render (requires preview server running first)
```

The full static-site workflow: `npm run build && npm run preview` (in one terminal), then `npm run prerender` (in another).

## Architecture

**Stack:** React 18 + Vite + React Router DOM v7. No test framework.

**Two backend connections:**
1. **Strapi CMS** (`src/api/strapi.js`) — content/copy fetched via `VITE_STRAPI_URL` + `VITE_STRAPI_TOKEN`. The `useHomeContent` hook normalizes the Strapi response (which uses French field names like `GrandTitre`, `SousTitre`) into consistent English-named objects consumed by components.
2. **Custom REST API** (`src/api/config.js`) — `VITE_API_URL` (defaults to `https://ufm-backend-production-3fd7.up.railway.app`). Used for forum threads/posts (`src/api/forum.js`), auth (`src/api/auth.js`), campaigns (`src/api/campaign.js`), and donations (`src/api/dons.js`).

**Environment variables** (create `.env.local`):
```
VITE_API_URL=...
VITE_STRAPI_URL=...
VITE_STRAPI_TOKEN=...
```

**Routing:** All routes are defined in `src/App.jsx`. `AppLayout` wraps all routes with `<Header>` and `<Footer>`, which receive content from the `useHomeContent` hook. Each route sets its own `<Helmet>` SEO tags inline in `App.jsx`.

**SSR/Prerender:** `main.jsx` uses `hydrateRoot` when the DOM already has children (prerendered HTML) and `createRoot` otherwise. `prerender.mjs` uses Puppeteer to visit each route via the preview server and writes static `index.html` files into `dist/`.

**Component structure:** Each feature has its own folder under `src/components/` with co-located `.css` files. Global styles live in `src/styles/` (`theme.css`, `global.css`, `sections.css`).

**Static data:** `src/data/candidate.js` and `src/components/forum/forumData.js` hold hardcoded fallback/seed data. `src/config/documents.js` stores document links.

---
> Source: [hezrona/revolution-site-election-frontend](https://github.com/hezrona/revolution-site-election-frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
