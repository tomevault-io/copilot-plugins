## sideimpactor

> - Use `bun` for all Node.js dependency and script operations.

# AGENTS Guide

## Package Manager

- Use `bun` for all Node.js dependency and script operations.

## Project Layout

- Core npm package source: `dependencies/webmuxd/src/`
- High-level iMobileDevice interactions: `dependencies/webmuxd/src/core/imobiledevice-client.ts`
- Frontend React app: `frontend/src/`
- Frontend entry: `frontend/src/main.tsx` → `App.tsx`
- Frontend components: `frontend/src/components/`
- Frontend business logic: `frontend/src/lib/` (storage, pair-record, account-session) + `frontend/src/flows/` (pair, login, sign, install)
- Cloudflare Workers backend: `backend/`
- WASM packages: `wasm/openssl/`, `wasm/libcurl-wasm/`, `wasm/zsign-wasm/`

## Key Rule: Avoid Logic Duplication

- Do not re-implement usbmux/lockdown/AFC/InstProxy protocol logic in `frontend`.
- Frontend must consume workspace package exports from `webmuxd` via the vite alias.
- If behavior changes are needed, modify `dependencies/webmuxd/` first, then wire it in frontend.

## Build & Validate

- WASM dist (always run before frontend): `bun run build:wasm:dist`
- Dev server: `bun run dev`
- Frontend build: `bun run build:frontend`
- Root lint: `bun run lint`
- Root test: `bun run test`
- Frontend tests: `bun run test:frontend`

## Change Style

- Keep changes minimal, focused, and consistent with existing style.
- Prefer removing dead code over keeping legacy paths.

---
> Source: [lbr77/SideImpactor](https://github.com/lbr77/SideImpactor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
