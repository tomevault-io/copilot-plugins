## uurc-web

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

UU Remote Web (`uurc-web`): a self-hosted browser controller for UU Remote devices (SMS login, credential import, device listing, WebRTC remote video/audio/input/clipboard). Two deployment targets share the `shared/` protocol code and must stay at signaling parity:

- **Cloudflare Worker + Durable Object** (`cloudflare/`) — the recommended/primary target.
- **Node/Express gateway** (`backend/`, shipped via Docker) — must keep signaling logic in sync with Cloudflare.

## Structure — npm workspaces monorepo (Node 22, npm)

Workspaces: `shared`, `backend`, `frontend`. **`cloudflare/` is NOT a workspace** — it is bundled/tested from the repo root via wrangler + its own vitest config.

- `shared/` (`@uurc/shared`) — protocol codecs, validation, room/session models, shared types. Exposes ~30 subpath exports mapping to `./dist/*.js`.
- `backend/` (`@uurc/backend`) — Express gateway. `src/{routes,services,middleware,types}`.
- `frontend/` (`@uurc/frontend`) — React 19 + react-router 7 + Vite 6 SPA.
- `cloudflare/` — Worker entry `src/index.ts`, `signalSession.ts` (`RemoteSignalSession` Durable Object, SQLite-backed), own `tests/`.

## Commands

- Install: `npm ci`
- Dev (shared+backend+frontend): `npm run dev` — backend on :8787, Vite on :5173 (proxies `/api`→8787)
- Cloudflare dev: `npm run dev:cloudflare` (`wrangler dev`)
- Build all: `npm run build` — builds `shared` → `backend` → `frontend` **in order**
- Lint / fix: `npm run lint` / `npm run lint:fix`
- Format / check: `npm run format` / `npm run format:check`
- Frontend typecheck: `npm run typecheck -w frontend` (no root typecheck script; CI runs this explicitly)
- Test all: `npm test`
- Test one workspace file: `cd <shared|backend|frontend> && npx vitest run <file>` (or `-t "<name>"`)
- Cloudflare tests use their own config: `npx vitest run --config cloudflare/vitest.config.ts`

**`shared` must be built before backend/frontend/dev** — they import `@uurc/shared/*` → `./dist/*.js`. `npm run dev` and `npm run build` already chain this; a stale `shared/dist` breaks the backend, so rebuild `shared` after editing it.

## Code style

- Prettier: **printWidth 120**, 2-space, semicolons, **double quotes**, `trailingComma: "all"`.
- TS is `NodeNext` + ESM: relative imports in `backend`/`shared` need explicit **`.js` extensions** (e.g. `import { createApp } from "./app.js"`).
- ESLint: unused vars are an **error**, but `^_`-prefixed args/vars/caught-errors are intentionally allowed. Test files relax `no-explicit-any` / `no-non-null-assertion`.
- Source/config comments are often in Chinese; keep that convention. Commits: **Conventional Commits in English** (`type(scope): summary`).

## Testing

- Vitest per workspace (frontend: jsdom + `@testing-library`; others: node; backend also uses supertest).
- **Backend vitest has `retry: 2`** — deliberate mitigation for supertest ephemeral-port flakiness; real regressions still fail deterministically.
- Tests must use **synthetic/redacted data** — never real UU credentials, SMS codes, device IDs, or IPs.
- Backend gateway changes → add backend tests; Worker signaling changes → add cloudflare tests; browser behavior → focused frontend test files (not one large integration test).

## Env vars (no `.env.example` exists; read directly in code)

- Backend (`backend/src/config.ts`): `HOST` (`0.0.0.0`), `PORT` (`8787`), `ENABLE_DIAGNOSTICS` (on unless `"false"`), `ENABLE_WISP` (**off** unless `"true"` — optional WASM curl transport).
- `UURC_BACKEND_LOG_PATH` (default `/tmp/uurc-web-backend.log`, 5MB cap).
- Frontend: `VITE_REMOTE_CURSOR_LOCAL_RENDERING` (on unless `"false"`).

## Gotchas

- **shared export ↔ vitest alias drift:** `backend/vitest.config.ts` (and cloudflare) hardcode a manually-maintained subset of `@uurc/shared` alias paths. Adding a new `shared` subpath export may require updating those alias lists. Frontend instead auto-derives aliases via `frontend/sharedSourceAliases.ts` (remaps to `shared/src/*.ts` source for dev/build/test).
- **Docker** runs `read_only: true` with all caps dropped and `no-new-privileges`, so the log file needs the `/tmp` tmpfs mount.
- **Docker publish** (`docker-publish.yml`) only runs on GitHub release and is gated `if: github.repository == 'iola1999/uurc-web'`. Cloudflare deploy is manual (`npm run deploy:cloudflare`), not in CI.

## Before considering a change done

Run the CONTRIBUTING pre-PR sequence (also enforced by CI):

```
npm run format:check
npm run lint
npm run typecheck -w frontend
npm test
npm run build
npm run check:cloudflare
```

- Keep **Cloudflare and Node gateways at signaling parity** — change both sides and add tests for both.
- Keep bilingual docs in sync: update **both** `README.md` and `README.zh-CN.md` (and `CONTRIBUTING` variants) for user-facing changes.

---
> Source: [iola1999/uurc-web](https://github.com/iola1999/uurc-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-27 -->
