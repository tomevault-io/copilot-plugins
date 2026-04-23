## webring-ca

> A webring for Canadian builders (developers, designers, founders). Hosted on Cloudflare Workers with Hono and KV. Domain: webring.ca.

# CLAUDE.md — webring.ca

## Project Overview

A webring for Canadian builders (developers, designers, founders). Hosted on Cloudflare Workers with Hono and KV. Domain: webring.ca.

## Architecture

- **Hono** on Cloudflare Workers — all routing, JSX server-rendering, middleware
- **Cloudflare KV** — member list, health status, ring order. No database (D1).
- **Cloudflare Assets** — static files served from `public/` (configured in `wrangler.toml`)
- **htmx** — included in layout for future interactive features, NOT used in v1
- All data access goes through `src/data.ts` — never call KV directly from routes

## Key Conventions

- All HTML is server-rendered via Hono JSX (`.tsx` files). No client-side hydration.
- CSS is inline in the Layout component. No external CSS files, no Tailwind.
- Space Grotesk + Space Mono loaded via Google Fonts. No other font families.
- Light/dark mode via `prefers-color-scheme`.
- Static assets (images, SVGs) go in `public/` — never load from external CDNs.
- Member coordinates resolved via `src/utils/member-coords.ts` (city-name fallback) — never check `lat`/`lng` directly.
- Keep it simple. No over-engineering. This is a small community project.

## Before making changes, read:

- `docs/hono.md` — Hono reference for CF Workers, JSX, routing, middleware, bindings
- `docs/htmx.md` — htmx reference (for when we build v2 interactive features)
- `docs/deploy-checklist.md` — pre-deploy verification steps

## Data Flow

1. `members/*.json` -- each member has their own file (slug is the filename)
2. `scripts/seed.ts` reads all member files, enriches coordinates, syncs to KV
3. Routes read from KV via `src/data.ts`
4. Health check cron writes status to KV via `src/data.ts`
5. Shuffle cron writes ring order to KV via `src/data.ts`

## Map Pipeline

1. `scripts/build-map-assets.ts` fetches Natural Earth GeoJSON, simplifies geometry, and outputs:
   - `src/lib/canada-outline.json` / `canada-regions.json` — raw GeoJSON (build-time only, NOT bundled)
   - `src/lib/canada-map-data.json` — pre-computed SVG paths + projection params (~344K, bundled)
2. `src/lib/canada-map.ts` imports `canada-map-data.json` and re-creates the d3 projection for `projectToSvg()`
3. Run `npx tsx scripts/build-map-assets.ts` to regenerate after changing map parameters

**Do not import raw GeoJSON in Worker code.** The CF Workers bundle limit is 1MB compressed.

## KV Keys

- `members` — JSON array of Member objects
- `health:{slug}` — health status per member
- `ring-order` — shuffled array of slugs

## Do NOT

- Add D1 or any database
- Add React, Vue, Svelte, or any client-side framework
- Add Tailwind or any CSS framework
- Add htmx interactions (v2 feature — just keep the script tag in layout)
- Add admin dashboard, auth, or email notifications
- Import raw GeoJSON in Worker code (use pre-computed `canada-map-data.json`)
- Load assets from external CDNs — self-host in `public/`
- Check `member.lat`/`member.lng` directly — use `getMemberCoordinates()` from `src/utils/member-coords.ts`
- Over-engineer. If it can be done in 10 lines, do it in 10 lines.

## Design Context

### Users
Canadian builders — developers, designers, and founders with personal sites. Two equal jobs: (1) visitors discovering new Canadian creators by navigating the ring, and (2) builders evaluating whether to join. Both should feel this is a quality community worth being part of.

### Brand Personality
**Proud, modern, clean.** Canadian identity without being kitschy. Confident but not corporate. The site itself should feel like it was made by a builder — intentional, well-crafted, no filler.

### Aesthetic Direction
- Modern and clean with subtle warmth — not sterile
- Restrained use of Canadian identity (maple leaf as accent, not theme)
- Typography-driven hierarchy; let whitespace and type do the work
- No SaaS patterns: no hero sections, gradient CTAs, or sales copy
- No retro-nostalgic: no pixel art, marquees, or GeoCities callbacks
- Both light and dark mode, auto-switching via `prefers-color-scheme`, each designed intentionally

### Design Principles
1. **Substance over decoration** — Every element earns its place. No ornament for ornament's sake.
2. **Proud, not loud** — Canadian identity is present but understated. Confidence through craft, not volume.
3. **Member sites are the product** — The webring site is a frame, not the painting. Design serves discovery.
4. **Built by a builder** — The site should feel handmade and intentional, not generated or templated.
5. **Respect the visitor's time** — Fast to load, fast to understand, fast to navigate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stanleypangg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
