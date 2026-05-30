## hyperlocal

> - Monorepo (pnpm workspaces). Key paths:

# Repository Guidelines

## Project Structure & Module Organization
- Monorepo (pnpm workspaces). Key paths:
  - `apps/ui/` Next.js app (Data Health, Ingestor).
  - `packages/` libraries: `types/`, `core/`, `storage/`, `ingestor/`, `cli/`.
  - `configs/` runtime config: `default.config.yaml` (override via `HYPERLOCAL_CONFIG` or `configs/local.config.yaml`).
  - `scripts/` helper scripts (e.g., `scripts/binance_backfill.py`); lake under `storage_root` (default `./data/hyperliquid`).

See also: RUNBOOK.md for day‑to‑day operations.

## Build, Test, and Development Commands
- `pnpm start`: one‑command start (UI dev + auto‑start ingestor).
- `pnpm dev`: run package dev tasks in parallel (watchers).
- `pnpm --filter ui dev`: run the UI locally (recommended).
- `pnpm test`: run unit/integration tests.
- `pnpm typecheck`: TypeScript project references build check.
- `pnpm lint`: ESLint + Prettier.
- `pnpm run ci`: typecheck, lint, tests, and forbidden-symbols scan.
- `pnpm db:init` · `pnpm storage:seed` · `pnpm storage:verify`: storage ops.
- `pnpm --filter @hyperlocal/storage test` · `pnpm --filter @hyperlocal/ingestor test`: targeted tests.
- `pnpm binance:backfill` — Python historical backfill (klines). Use `-- --base-url https://api.binance.com` or `https://api.binance.us` and `--coins`, `--intervals`.

## Coding Style & Naming Conventions
- Language: TypeScript (strict). Node 20, pnpm 9.
- Formatting: Prettier; linting: ESLint; base config in `tsconfig.base.json`; EditorConfig enforced.
- Naming: types/interfaces `PascalCase`; variables/functions `camelCase`; constants `UPPER_SNAKE_CASE`; files `kebab-case.ts`.
- Do not introduce any trading/execution/signing code. This is signals-only; CI blocks forbidden terms.
- Schemas via `zod`; keep HL wire payloads and normalized types side-by-side for audit.

## Backfill Workflow & Data Integration
- Run Hyperliquid backfill first (UI Ingestor Start) to establish earliest HL boundaries.
- Run Python Binance backfill to extend older history only:
  - Pulls 1000‑bar windows backward from (earliest HL open − 1ms) per series.
  - Stops before earliest existing Binance row to avoid overlap/duplicates.
  - Writes Parquet with `src='binance'` (provenance preserved).
- The candles API merges sources with HL precedence per `open_time`, so Explorer shows a unified series (HL where available, otherwise Binance).
- Features (RSI/EWVol) are computed from HL bars; Explorer overlays align with bar opens.

## UI Behavior
- Explorer: auto full‑history hydration per interval + infinite scroll, HL‑over‑Binance dedupe, live tick indicator.
- Data Health: per‑source breakdown (HL vs Binance) with row counts and ranges, CSV export by coin/interval, gaps panel (lookback, Refresh, Fill Now), live tick indicator.
- Global nav: sticky header across pages for quick navigation.

## Testing Guidelines
- Framework: Vitest (deterministic unit/integration).
- Location: co-locate `*.test.ts` next to sources.
- Storage and Ingestor have runnable tests; prefer targeted filters when iterating.
- Run `pnpm test` and `pnpm run ci` before PRs.

## Commit & Pull Request Guidelines
- Commits: Conventional Commits style (e.g., `feat(ingestor): add heartbeat`).
- PRs must include: clear scope/summary, linked issue, screenshots for UI, reproduction steps, and checklist of `pnpm ci` passing.
- Small, focused PRs preferred; update docs/configs when contracts change.

## Security & Configuration Tips
- Local-only by design; no secrets required. Switch to testnet in `configs/local.config.yaml` (`ws.url`).
- Signals-only guardrail: `scripts/forbidden-check.cjs` blocks trading/execution symbols (CI).
- Auto‑init DuckDB/views on first API request; no manual init required.
- UI Data Health shows per-interval row breakdown; verify after ingestor Start (auto‑started by `pnpm start`).

## Gap Protection
- Startup: after Hyperliquid backfill, scan recent HL history for holes and fill.
- Mid‑session: on WS candle, detect missed intervals and backfill targeted ranges.
- Periodic: every 15 minutes, rescan recent data and fill any gaps.
- UI/API: Data Health gaps panel with Refresh/Fill Now actions.
- Binance Python backfill:
  - Preferred domain: `api.binance.com` (deeper history). If restricted, use `api.binance.us`.
  - Symbol mapping defaults to `{coin}USDT`. Override with `--symbol-map "BTC:BTCUSDT"` or config.
  - Rate safety: default sleep ~200ms between calls; raise if you see 429s.

---
> Source: [zekenewsom/hyperlocal](https://github.com/zekenewsom/hyperlocal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-30 -->
