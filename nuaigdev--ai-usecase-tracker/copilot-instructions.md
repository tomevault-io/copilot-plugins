## ai-usecase-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # dev server at localhost:4321
npm run build      # build → .vercel/output (SSR, @astrojs/vercel adapter)
npm run preview    # preview the build locally

npm run db:schema  # apply supabase/schema.sql over a direct Postgres connection
npm run create-admin
npm run migrate    # one-off: copy legacy Vercel Blob data into Supabase
```

The `db:*` scripts need `SUPABASE_DB_URL` in `.env.local` (Supabase → Settings →
Database → Connection string → "Session pooler"). Note `scripts/` is gitignored —
it is local-only operator tooling and is not part of a clean checkout.

## Architecture

**Stack:** Astro 4 (SSR via `@astrojs/vercel`, `prerender = false` on the data
pages) + React islands + Tailwind CSS 3, with **Supabase** (Postgres + Auth +
Storage) as the backend. Content is *not* in the repo — it lives in the database
and is read at request time by `src/lib/storage.ts`.

### Data model

Four tables (`supabase/schema.sql`): `trackers` → `categories` → `items`, plus
`profiles` for roles. A tracker's `type` decides how its items are shaped and
rendered; the per-type payload lives in the `items.data` **jsonb** column, so a
new tracker type usually needs *no* schema change.

| `trackers.type` | `items.data` payload | Public component |
| --- | --- | --- |
| `usecases` | `description`, `businessValue[]`, `techStack[]`, `limitations[]`, `complianceFlags[]`, `subCases[]`, `workflowImage?`, `link?` | `CardGrid.tsx` |
| `integrations` | `tag`, `url` | `IntegrationsView.tsx` |
| `automation` | `description`, `businessValue[]`, `techStack[]`, `steps[]` (a flow graph), `workflowImage?`, `link?` | `AutomationView.tsx` |
| `bi` | `description`, `kpis[]`, `screenshots[]` (`{url, caption?}`) | `BIView.tsx` |

`src/types.ts` is the source of truth for these shapes; `src/lib/storage.ts` maps
rows → typed objects (and tolerates hand-edited rows, e.g. a screenshot stored as
a bare URL string).

### Pages

- `/` (`src/pages/index.astro`) — renders whichever tracker `?tracker=<slug>`
  names, branching on `type` to the component in the table above.
- `/present` — full-screen slideshow. **Use-case trackers only**; the Present
  button is hidden for every other type.
- `/admin` — `AdminPanel.tsx`, a single island handling auth, tracker CRUD,
  category CRUD, item CRUD, and editor-user management.
- `/api/admin/users.ts` — the only server route; uses the service-role key to
  create/delete editor accounts after verifying the caller is an admin.

### Auth & RLS

- **admin** (`profiles.role = 'admin'`) — full CRUD on every tracker.
- **editor** (`profiles.role = 'editor'`, `tracker_id` set) — RLS confines writes
  to their one tracker.

Public reads need no auth. The admin panel writes with the logged-in user's JWT,
so RLS — not client-side code — is what actually enforces the above.

**Key handling:** `src/lib/supabase.ts` may only ever touch `PUBLIC_*` env vars
(it ships to the browser). The service-role key lives in `src/lib/supabaseAdmin.ts`,
which must only be imported from server-only routes.

### Image storage (BI screenshots, automation workflow images)

Two public Supabase Storage buckets: **`dashboards`** (BI screenshots, many per
item) and **`workflows`** (one workflow image per automation process). The admin
panel uploads **straight from the browser** with the user's JWT — never through an
API route, because a Vercel function caps the request body at 4.5 MB and these
PNGs routinely exceed it. Both buckets: reads public, writes require a signed-in
user (`supabase/migrations/0001_business_intelligence.sql`,
`0002_automation_workflow_image.sql`).

### Migrations

Post-schema changes live in `supabase/migrations/`, applied via the Supabase SQL
Editor. Each file is idempotent and safe to re-run. See `supabase/README.md`.

## Brand

- Blue accent `#069BDF` (exposed as `brand` in the Tailwind config), dark `#111111`.
- `public/logo.svg` (dark text, light bg) and `public/logo-white.svg` (present mode).

## Deployment

Vercel, from `main`. Node is pinned to **20.x** (`package.json` → `engines`), which
the `@astrojs/vercel` adapter requires to emit a valid runtime. `src/lib/nodeWebSocket.ts`
shims a global `WebSocket` so `supabase-js` can `createClient()` on Node 20 — it must
be imported *before* any `createClient()` call.

Environment variables (Vercel + `.env.local`): `PUBLIC_SUPABASE_URL`,
`PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`.

---
> Source: [nuaigdev/ai-usecase-tracker](https://github.com/nuaigdev/ai-usecase-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-20 -->
