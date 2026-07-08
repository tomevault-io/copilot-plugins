## shaheers-mm

> A restaurant POS menu-management platform. Operators build menus (menus → categories → items), configure modifiers/options, set per-channel visibility and day/time availability, manage kitchen stations, and import/export everything via Excel. The repo bundles two deliverables: a Vite + React web app (`delightful-menu-craft/`) deployed to Vercel, and a Chrome extension (`extension/`) that scrapes DoorDash store menus and feeds them into the web app. Cloud storage, auth, and an AI-enhancement proxy run on Supabase.

# ShaheersMM (Menu Manager)

## Overview
A restaurant POS menu-management platform. Operators build menus (menus → categories → items), configure modifiers/options, set per-channel visibility and day/time availability, manage kitchen stations, and import/export everything via Excel. The repo bundles two deliverables: a Vite + React web app (`delightful-menu-craft/`) deployed to Vercel, and a Chrome extension (`extension/`) that scrapes DoorDash store menus and feeds them into the web app. Cloud storage, auth, and an AI-enhancement proxy run on Supabase.

## Tech Stack
- **Web app**: TypeScript, React 18, Vite 5 (`@vitejs/plugin-react-swc`), React Router 6.
- **UI**: shadcn-ui (Radix primitives), Tailwind CSS 3, `lucide-react`, `next-themes` (defaults to dark), `sonner` toasts.
- **State**: Zustand (with `persist` to localStorage) — no Redux/Context for data.
- **Data/server**: Supabase (`@supabase/supabase-js`) for Postgres + Auth + Edge Functions; `@tanstack/react-query` is wired up but most data flows through Zustand.
- **Excel**: `xlsx` (SheetJS) for import/export.
- **Forms/validation**: `react-hook-form` + `zod`.
- **Extension**: vanilla JS, Manifest V3 (no build step).
- **Deploy**: Vercel (root `vercel.json` builds the web app); a `Dockerfile`/`docker-compose.yml` exist for local dev only.

## Architecture
Three cooperating pieces:

1. **`delightful-menu-craft/` — the web app.** Single-page React app. All menu data lives in one Zustand store (`src/store/menuStore.ts`) persisted to `localStorage`, then synced to Supabase as one JSON-blob row per "workspace" (a menu build). `src/lib/workspaceSync.ts` handles autosave (~1.5s debounce), optimistic-concurrency version checks, and a tab-level edit lock (heartbeat + abandonment recovery) so only one editor mutates a workspace at a time. Auth (`src/contexts/AuthContext.tsx`) gates routes; roles are `admin` vs `member`.

2. **`extension/` — DoorDash scraper (Chrome MV3).** `content.js` runs on `doordash.com/store/*` and extracts menu data (RSC payload, with ld+json fallback). `mapper.js` normalizes it into the app's entity shape. `popup.js` drives the scrape→preview→import UX and stashes the result in `chrome.storage.local`. `bridge.js` runs on the deployed app domain, reads that stash, and `postMessage`s `DD_EXTENSION_IMPORT` into the page. The app receives it via `src/hooks/useExtensionImport.ts` and `src/lib/doorDashMapper.ts`.

3. **`delightful-menu-craft/supabase/` — backend.** `schema.sql` defines `workspaces`, `profiles` (roles), and `audit_log` with RLS + triggers. Edge Functions (Deno): `ai-enhance` (server-side Anthropic proxy), and user management (`create-user`, `set-password`, `remove-user`; `invite-user` is legacy/unused).

### Data model
Types live in `delightful-menu-craft/src/types/menu.ts` and mirror a multi-sheet Excel workbook: `Menu` → `Category` (nestable via `parentCategoryId`) → `Item`; `Modifier`/`ModifierOption` linked to items via `ItemModifier` join; `Station` (numeric id); `Tag`/`Allergen` via join tables. Visibility channels and grouping are centralized in `src/lib/visibility.ts` (`VISIBILITY_CHANNELS`); per-group day/time schedules are stored as `daySchedulesByGroup`.

## Key Files & Entry Points
- `delightful-menu-craft/src/main.tsx` — React entry.
- `delightful-menu-craft/src/App.tsx` — routing + providers; routes `/login`, `/set-password`, `/workspaces`, `/team` (admin-only), `/` (builder, requires auth + loaded workspace).
- `delightful-menu-craft/src/pages/Index.tsx` — main builder shell (left icon nav / main content / right detail panel).
- `delightful-menu-craft/src/store/menuStore.ts` — single Zustand store (UI + data state), localStorage key `menu-manager-storage`, versioned migrations.
- `delightful-menu-craft/src/lib/workspaceSync.ts` — Supabase sync, autosave, edit-lock logic.
- `delightful-menu-craft/src/lib/visibility.ts` — channel definitions (single source of truth).
- `delightful-menu-craft/src/lib/excelParser.ts` / `excelExporter.ts` — Excel import/export.
- `delightful-menu-craft/src/lib/aiEnhance.ts` + `src/hooks/useAiEnhance.ts` — AI station/name enhancement (calls the `ai-enhance` edge function).
- `delightful-menu-craft/src/lib/doorDashMapper.ts` + `src/hooks/useExtensionImport.ts` — receive extension imports.
- `delightful-menu-craft/src/types/menu.ts` — all entity types.
- `delightful-menu-craft/supabase/schema.sql` + `supabase/README.md` — DB schema and full backend setup guide.
- `extension/manifest.json`, `content.js`, `mapper.js`, `popup.js`, `bridge.js` — the Chrome extension.
- `vercel.json` (root) — production build config; `delightful-menu-craft/CLAUDE.md` — deeper app-internals notes.

## Build / Run / Test
All web-app commands run from `delightful-menu-craft/` (scripts in its `package.json`):
```bash
cd delightful-menu-craft
npm install          # or: npm i
npm run dev          # Vite dev server on http://127.0.0.1:3000
npm run build        # production build (also serves as the type-check — clean output = no TS errors)
npm run build:dev    # development-mode build
npm run lint         # ESLint
npm run preview      # preview the production build
```
There is **no test suite** — use `npm run build` to verify TypeScript correctness.

Docker (local dev only; runs the dev server on port 8080):
```bash
cd delightful-menu-craft && docker compose up
```

Production build (Vercel, from repo root, per `vercel.json`):
`cd delightful-menu-craft && npm install && npm run build` → output `delightful-menu-craft/dist`.

Supabase Edge Functions (deploy from `delightful-menu-craft/`, requires Supabase CLI):
```bash
supabase secrets set ANTHROPIC_API_KEY=sk-ant-...
supabase functions deploy ai-enhance
supabase functions deploy create-user
supabase functions deploy set-password
supabase functions deploy remove-user
```

The Chrome extension has **no build step** — load `extension/` unpacked via `chrome://extensions` (Developer mode → Load unpacked).

## Conventions & Gotchas
- **Env vars** (`delightful-menu-craft/.env.local`, template in `.env.example`): only `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`. The anon key is safe to ship (RLS enforces access). **Never** put the `service_role` key or the Anthropic key in client code — the Anthropic key lives server-side as a Supabase function secret.
- **Single store**: both UI state (active tab, selected/editing IDs) and data state are in one Zustand store. It has a version counter with explicit migrations — bump the version and add a migration when a schema change needs backfilling.
- **Workspace = one JSON row**: each menu build is a single row in the `workspaces` table; the client autosaves ~1.5s after edits with an optimistic-concurrency version check. A tab-level lock (heartbeat ~8s, abandonment after 90s) prevents concurrent edits.
- **Path alias**: `@/` → `delightful-menu-craft/src/` (see `vite.config.ts` and `tsconfig`).
- **Visibility channels** are defined only in `src/lib/visibility.ts` — edit there when adding/renaming a channel; UI labels and parse helpers derive from it.
- **Auth is invite-only**: sign-up is disabled in Supabase; admins create accounts via the in-app **Team** screen (email-free, no SMTP). Roles (`admin`/`member`) live in `profiles` and are not client-writable.
- **Dev URL is `127.0.0.1:3000`** (not `localhost`); the extension's host permissions and `bridge.js` target `127.0.0.1:3000`, `localhost:3000`, and `https://shaheers-mm.vercel.app`. To point the extension at local dev, change `MENU_MANAGER_URL` in `extension/popup.js`.
- **DoorDash scraping is brittle**: `content.js` depends on DoorDash's RSC/ld+json structure and may break when their page changes; full menu must be scrolled into view before scraping.
- `lovable-tagger` runs only in dev mode (the project was scaffolded with Lovable).
- The web app's own `delightful-menu-craft/CLAUDE.md` has more detailed internals (store layout, dropdown/scheduling UI patterns, AI flow); consult it before changing those areas.
- Sample data workbooks (`business-*-menu-data*.xlsx`) live in `delightful-menu-craft/` for testing Excel import.

---
> Source: [ShaheerAIO/shaheers-mm](https://github.com/ShaheerAIO/shaheers-mm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
