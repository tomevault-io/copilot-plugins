## ychedgefund

> Core Atlas (YC Hedge Fund) coding conventions and architecture


# Atlas Conventions

## Tech Stack (STRICT)
- **Vanilla TypeScript** — NO React, Vue, Angular, Svelte
- **Vite** for bundling — NO Next.js, NO webpack
- **deck.gl + MapLibre** for globe — NO Mapbox (license)
- **Upstash Redis** for cache + auth + portfolio — NO traditional DB in MVP
- **Vercel Edge Functions** — all API in `/api/`, `runtime: 'edge'`

## File Ownership
- `api/` → api-agent
- `src/globe/`, `src/panels/`, `src/styles/` → frontend-agent
- `src/trading/` → trading-agent + risk-agent
- `src/intelligence/` → intelligence-agent
- `src/auth/` → frontend-agent (UI) + api-agent (endpoints)

## State & Events
- State: `src/lib/state.ts` — reactive store, `state.get()`, `state.update()`
- Events: `portfolio-updated`, `trading:performance`, `trading:signals`, `market-tick`, `auth:authenticated`
- Panels subscribe via `window.addEventListener('portfolio-updated', ...)`

## Auth Flow
- Auth required ONLY for paper trading (portfolio persistence)
- Dashboard is public
- `auth.isAuthenticated()` → check before server sync
- Portfolio: localStorage (instant) + Redis via PUT (debounced 5s)

## Reference
- WorldMonitor: https://github.com/koala73/worldmonitor (study, don't fork — AGPL)
- CLAUDE.md: project memory, file structure, build status

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KonstantinMB) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
