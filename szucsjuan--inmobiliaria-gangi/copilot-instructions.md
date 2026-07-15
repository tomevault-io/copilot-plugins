## inmobiliaria-gangi

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

Real estate listing site for "Inmobiliaria Gangi". Two independent, decoupled projects in one repo:

- `backend/` — Express API that proxies a third-party listings provider (Argencasas) and sends contact-form emails via Gmail/Nodemailer.
- `front/` — Static multi-page site (plain HTML/CSS/vanilla JS, no bundler, no framework, no `package.json`) that consumes the backend API.

There is no monorepo tooling tying the two together; they are run and deployed independently.

## Commands

### Backend (`backend/`)
```
npm install         # install dependencies
node index.js        # run the API (reads backend/.env, defaults to PORT=3000)
```
There is no real test script (`npm test` is a placeholder that exits 1) and no lint config. `backend/test/testEmail.js` is a manual, standalone script for smoke-testing the Nodemailer setup — run it directly with `node backend/test/testEmail.js`, it is not part of an automated suite.

Required env vars (see `backend/.env.example`): `API_KEY` (Argencasas API key), `PORT`, `EMAIL_USER`, `EMAIL_PASS` (Gmail address + app password used to send contact-form mail).

### Frontend (`front/`)
No build step and no `package.json`. It's served as static files — open `front/src/*.html` directly or serve the `front/src` directory with any static file server. All internal links/asset paths are root-relative (e.g. `/assets/...`, `/js/...`), so the site must be served from `front/src` as the web root, not opened via `file://` or from a nested path.

## Architecture

### Backend
- `index.js` — single entry point. Sets up Express, CORS, a global rate limiter (100 req/15min on `/api`), and:
  - A generic reverse-proxy factory (`argencasasProxy`) that forwards to `https://api.argencasas.com/{endpoint}?api_key=...` and re-exposes it under `/api/props`, `/api/zones`, `/api/operations-status`, `/api/types`, `/api/varios`. This is why the frontend never talks to Argencasas directly — the API key stays server-side.
  - Mounts `routes/emailRoutes.js` under `/api`.
- `routes/emailRoutes.js` — `POST /api/send-email`, with a stricter rate limit (5 req/15min), manual field validation (required fields, email regex, length caps), delegating actual sending to `helpers/emailHelper.js`.
- `helpers/emailHelper.js` — wraps Nodemailer with Gmail service auth from `EMAIL_USER`/`EMAIL_PASS`.

### Frontend
No SPA router — each `.html` page is a standalone entry point that pulls in shared JS via `<script>` tags, in a fixed order: `config.js` first (defines `API_BASE_URL` and `fetchListingData()`), then page logic, then `navbar.js`/`footer.js` (inject header/footer markup via `insertAdjacentHTML` on `DOMContentLoaded`) and `hamburger-nav.js` (mobile nav toggle) last.

- `js/config.js` — single source of truth for the API base URL and `fetchListingData({ includeVarios })`, which fan-outs parallel fetches to the backend's proxy endpoints (`props`, `zones`, `types`, `operations-status`, optionally `varios`) and returns them keyed by name. All pages that need listing data call this instead of hitting `fetch` directly.
- Domain model: raw records from Argencasas (`props`) reference other endpoints by numeric id — `property.zona` → `zones`, `property.tipo` → `types`, `property.operacion` → `operations`, `property.varios[]` → `varios`. Every page that renders a property looks up these ids against the corresponding `*Data.records` array to get display names. There's no client-side caching/store beyond page-local state (e.g. `propertyListState` in `filter.js`).
- Page/script pairing:
  - `home.html` + `js/home.js` — landing page, shows first 4 properties as cards.
  - `property-list.html` + `js/filter.js` + `js/property-card.js` — listing page with zone/type/mode dropdown filters and price sort; filters read initial state from URL query params (`?location=&tipo=`).
  - `property.html` + `js/property.js` — single property detail page (id via `?id=`), renders a Swiper.js gallery (loaded from CDN), Google Maps embed, and a contact form that posts to `/api/send-email`.
  - `contact.html` + `js/contact-form.js` — standalone contact form, same `/api/send-email` contract as the one embedded in `property.js`. Note: this duplicates the submit-handler logic in `property.js` rather than sharing it.
  - `property-card.html` — appears to be an isolated dev/preview harness for the card markup (contains commented-out sample markup), not linked from the main nav.
- CSS is one shared `style.css` plus a page-specific stylesheet per page (`home.css`, `property.css`, `property-list.css`, `property-card.css`, `contact.css`).

## Notes for making changes

- Any change to the shape of proxied Argencasas data (field names like `nro`, `valor`, `calle`, `zona`, `tipo`, `operacion`, `varios`) affects `home.js`, `filter.js`, `property-card.js`, and `property.js` simultaneously since they all consume the same records independently.
- The contact form exists in two places (`contact.html`/`contact-form.js` and inline inside `property.js`'s generated markup) — keep both in sync if changing the `/api/send-email` request contract or validation.
- `front/src/js/property.js` has a hardcoded Google Maps Embed API key inline (client-side, restrict it by HTTP referrer in Google Cloud Console rather than treating it as secret).

---
> Source: [SzucsJuan/inmobiliaria_gangi](https://github.com/SzucsJuan/inmobiliaria_gangi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
