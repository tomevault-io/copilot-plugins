## ac-wedding

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Catholic wedding planner for Aldrin & Christine's wedding (September 26, 2026). Full-stack self-hosted system: **Directus CMS** (backend/admin) + **Astro** (SSR frontend for guests). Deployed on Railway (Directus + PostgreSQL + Redis) and Vercel (Astro).

## Commands

```bash
# Docker (PostgreSQL + Redis + Directus)
pnpm directus:start    # Start all containers
pnpm directus:stop     # Stop containers
pnpm directus:logs     # Follow container logs
pnpm directus:cli      # Enter Directus container shell
pnpm db:exec           # Enter PostgreSQL shell

# Astro frontend
pnpm rsvp:dev          # Start Astro dev server on http://localhost:4321
pnpm rsvp:build        # Build Astro for production

# Seeding
pnpm seed                                  # Insert-or-skip (safe, default)
pnpm seed -- --fresh                       # Overwrite existing records (destructive)
pnpm seed -- --collection=persons,vendors  # Seed specific collections only
pnpm seed -- --fields                      # Apply field metadata
pnpm seed:settings / seed:budget / seed:checklist / seed:ceremony / seed:prep

# Emoji picker extension (use `pnpm run` for root scripts in this monorepo)
pnpm run emoji:build   # Build the extension
pnpm run emoji:dev     # Watch mode
pnpm run emoji:lint    # Lint src/
pnpm run emoji:format  # Format src/
```

There is no test runner or linter configured — validation happens at the Directus API level.

## First-Time Local Setup

1. `pnpm install`
2. Copy `.env.local.example` → `.env.local` and fill in secrets (SECRET, ADMIN_EMAIL, ADMIN_PASSWORD, SMTP credentials)
3. `pnpm directus:start`
4. `pnpm --filter directus bootstrap` (first time only)
5. Open http://localhost:8055 (Directus is already running from step 3)
6. Import schema: Directus UI → Settings → Data Model → Import → `packages/directus/schemas/snapshot.json`
7. Create admin access token in Directus UI → add to `.env.local` as `DIRECTUS_ADMIN_TOKEN`
8. `pnpm seed`
9. `pnpm rsvp:dev` → open http://localhost:4321?token=\<rsvp_token_from_guests_collection\>

## Architecture

### Stack
- **pnpm workspace** — `apps/*` and `packages/*`
- **Directus** (headless CMS + PostgreSQL 16/PostGIS + Redis) — admin panel, all data, automations
- **Astro 6 (SSR, Vercel adapter)** — public-facing RSVP and info pages
- **`@directus/sdk`** — used in both `apps/web` and `packages/seed`

### Key Directories
- `apps/web/src/lib/directus.ts` — Directus SDK client and all data fetchers for the frontend
- `apps/web/src/lib/types.ts` — TypeScript interfaces (generated from Directus schema)
- `apps/web/src/pages/api/` — Astro API endpoints: `.ics` calendar export, party lookup, photo upload
- `packages/seed/src/runner.js` — Generic seed orchestrator (~4k lines): reads JSON collections, builds a topological dependency graph, resolves `$ref` cross-references, upserts data
- `packages/seed/src/collections/*.json` — 17 seed files (persons, guests, vendors, entourage, budget, checklist, ceremonies, etc.)
- `packages/directus/schemas/snapshot.json` — Full Directus schema backup (import to restore schema)
- `.docker/compose.yaml` — Docker stack (PostgreSQL/PostGIS, Redis, Directus)
- `flows/*.json` — 7 Directus automation flows (importable via Directus UI)

### Data Flow: Guest RSVP
1. Guest receives email link: `https://wedding.aazucena.com?token=<uuid>`
2. Astro SSR page fetches guest by `rsvp_token` via public Directus API (only 5 fields exposed: `id`, `first_name`, `status`, `plus_ones_allowed`, `rsvp_token`)
3. RSVP form submits to **Flow 03 webhook**
4. Flow 03 updates the guest record and sends a confirmation email

### Directus Flows (7 Automations)
| # | Trigger | Purpose |
|---|---------|---------|
| 1 | Guest created | Generate unique `rsvp_token` |
| 2 | Manual bulk | Send HTML invitation emails |
| 3 | Webhook (RSVP page) | Update RSVP status, send confirmation |
| 4 | Scheduled | Mark non-responders as `no_response` |
| 5 | Vendor → booked | Allocate cost to budget category |
| 6 | Vendor → paid | Sync payment to budget |
| 7 | Entourage updated | Auto-invert procession → recessional order |

### Seed Runner `$ref` Resolution
Seed JSON files use `{ "$ref": "persons", "key": "aldrin" }` to reference other records. The runner builds a registry of inserted records and resolves references in dependency order before upserting. Self-referential/deferred relationships are handled in a second pass.

### Deployment
| Service | Platform | Domain |
|---------|----------|--------|
| Directus + DB | Railway | `planner.aazucena.com` |
| Astro wedding site | Vercel | `wedding.aazucena.com` |

See `RAILWAY.md` for Railway deployment steps. Astro env vars (`DIRECTUS_URL`, `DIRECTUS_TOKEN`) are server-only and set in `astro.config.mjs` / Vercel dashboard.

---
> Source: [aazucena/ac_wedding](https://github.com/aazucena/ac_wedding) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
