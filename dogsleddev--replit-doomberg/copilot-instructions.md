## replit-doomberg

> Bloomberg-style multi-tenant finance terminal: natural-language queries build live

# Doomberg — Agentic Finance Terminal (rescued from Replit)

Bloomberg-style multi-tenant finance terminal: natural-language queries build live
dashboards (price charts, fundamentals, news, factor models, strategies). Rescued
from a dying Replit workspace; now lives on GitHub + Vercel.

- **Live:** https://doomberg.dogsled.dev
- **Repo (source of truth):** https://github.com/dogsleddev/Replit-Doomberg (private)
- **Hosting/redeploy/settings:** [DEPLOYMENT.md](DEPLOYMENT.md)
- **Session state, accounts, fresh-machine bootstrap:** [HANDOFF.md](HANDOFF.md)

## Repo map

- `artifacts/terminal/` — the SPA (React 19 + Vite + Tailwind 4 + Radix; wouter routing;
  TanStack Query). Built output: `dist/public`.
- `artifacts/api-server/app/` — **the real backend**: FastAPI (Python). Routes carry the
  `/api` prefix. Multi-tenant auth (`auth.py`, session cookie `at_session`, also accepted
  as `Authorization: Bearer`), orchestrator (`agent.py`, `harness.py`, `skills.py`),
  market data via yfinance/feeds (`sources/`), optional Postgres (`db.py`).
- `artifacts/api-server/src/` — dead Express scaffolding from the original template.
  Ignore it (replit.md's "Express 5" stack note is stale).
- `lib/` — TS workspace packages; `api-client-react` is Orval-generated from
  `lib/api-spec` (OpenAPI). Packages export raw TS (`./src/index.ts`) — Vite compiles.
- `api/index.py` — Vercel serverless shim: puts `artifacts/api-server` on `sys.path`,
  re-exports `app.main:app`. `/api/*` rewrites land here (see `vercel.json`).
- `scripts/smoke-api.sh` — canonical smoke flow (healthz → signup → orchestrate).

## Hard constraints (learned the expensive way)

- **You cannot build or run this locally on Windows.** `pnpm-workspace.yaml` strips all
  non-linux-x64 native binaries (esbuild/rollup/lightningcss/oxide). Don't `pnpm install`,
  don't try the dev server. Vercel's Linux builders do all builds.
- **Verify via deploys, not local runs.** Smoke-test the live URL (or a preview deploy):
  `curl https://doomberg.dogsled.dev/api/healthz` → 200; then signup + orchestrate per
  `scripts/smoke-api.sh` (translate to curl — bash script assumes a local server).
- **Vercel:** always pass `--scope team_i1Es1eTRb83TisgbHEU6gcA5` (hidden default team of
  the `dogsled` account; invisible to `vercel teams ls`). Deploy with
  `vercel deploy --prod --yes --archive=tgz`. The Vercel MCP connector is a *different*
  account (Jessica's) — useless here; use the CLI.
- **GitHub:** push as `dogsleddev` (see HANDOFF.md → Auth for per-machine specifics).
- Vercel CLI 54 quirks: `env add <name> preview` is broken non-interactively
  (`git_branch_required` loop — skip preview vars; build env is inlined in
  `buildCommand`); `vercel domains add <domain>` takes a single arg (project from link).

## Runtime env (Vercel project, Production + Development)

`PORT=5173`, `BASE_PATH=/` (vite config throws without them), `TERMINAL_SKIP_WORKER=1`
(no background ingestion loop on serverless), `REPLIT_DOMAINS=doomberg.dogsled.dev`
(CORS allow-list). Optional upgrades: `DATABASE_URL` (Postgres → real persistence;
without it stores are in-memory per instance), `AI_INTEGRATIONS_OPENAI_BASE_URL` +
`AI_INTEGRATIONS_OPENAI_API_KEY` (OpenAI-compatible endpoint → enables AI orchestration,
voice, transcribe; without them `aiAvailable:false` and the heuristic builder runs).

---
> Source: [dogsleddev/Replit-Doomberg](https://github.com/dogsleddev/Replit-Doomberg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
