## forven

> Forven is a local-first algorithmic trading operations framework. It acts as an autonomous workspace for quantitative trading: strategy creation, backtesting, deployment, and risk management.

# Forven - Agent Instructions

## Project Overview

Forven is a local-first algorithmic trading operations framework. It acts as an autonomous workspace for quantitative trading: strategy creation, backtesting, deployment, and risk management.

- **Backend**: Python 3.11+ / FastAPI - serves on `http://127.0.0.1:8003`
- **Frontend**: SvelteKit 2 (Svelte 5) + TailwindCSS + Vite - serves on `http://127.0.0.1:5173`
- **Database**: SQLite via `forven/db.py`
- **Backtesting**: Built-in bar-by-bar engine with vectorized signal generation
- **Vector Store**: ChromaDB
- **Exchange**: CCXT / Hyperliquid integration under `forven/exchange/`

---

## Repository Layout

```text
forven/                    # Python backend package
  api.py                   # FastAPI app, lifespan, router registration
  api_core.py              # Shared startup, compatibility, and legacy helpers
  control_plane/           # Operator-facing control-plane logic
  api_domains/             # API-facing domain modules and compatibility helpers
  routers/                 # FastAPI routers (one file per domain)
    agents.py              #   /api/agents
    analytics.py           #   /api/dashboard/*, /api/stats, scanner analytics
    approvals.py           #   /api/approvals
    auth.py                #   /api/auth/providers/*
    backtesting.py         #   /api/backtesting/*
    data.py                #   /api/data/* and dataset routes
    jobs.py                #   /api/jobs
    legacy.py              #   /api/forven/* compatibility routes
    lifecycle.py           #   /api/lifecycle/*
    memory.py              #   /api/memory/*
    notifications.py       #   /api/notifications/*
    ops.py                 #   /api/system/*, /api/logs, scheduler, resets
    paper.py               #   /api/paper/*
    quant_factory.py       #   /api/quant-factory
    robustness.py          #   /api/robustness/*
    simulation.py          #   /api/simulation/*
    status.py              #   /, /api/health, dashboard and status routes
    strategies.py          #   /api/strategies and results routes
    system.py              #   /api/settings/*, brain chat, system helpers
    tasks.py               #   /api/tasks and pipeline task audit routes
    trading.py             #   /api/trades/*
    verdict.py             #   /verdict/*
    webhooks.py            #   /api/webhooks/*
    websockets.py          #   /api/ws/live and /ws/live
  strategies/
    base.py                # BaseStrategy interface - all strategies extend this
    backtest.py            # Backtest engine, run_backtest()
    optimizer.py           # Grid search and optimization helpers
    fitness.py             # Fitness scoring functions
    registry.py            # Strategy discovery and loading
    sentiment.py           # Sentiment-based signal helpers
    builtin/               # Shipped strategies
    custom/                # User-created strategies (gitignored)
  cli.py                   # Click CLI (`python -m forven ...`)
  config.py                # Global configuration loader
  data.py                  # Market data download and ingestion
  db.py                    # SQLite schema and session helpers
  policy.py                # Pipeline stages and gate criteria
  scanner.py               # Market screener / scanner logic
  scheduler.py             # Cron-style task scheduling
  simulation.py            # Core simulation engine

frontend/                  # SvelteKit frontend
  src/
    routes/
      +page.svelte         #   /
      agents/              #   /agents
      ai-dropzone/         #   /ai-dropzone
      approval/            #   /approval
      data/                #   /data
      lab/                 #   /lab
        strategy/[id]/     #   /lab/strategy/:id
      memory/              #   /memory
      ops/                 #   /ops
      risk/                #   /risk
      runs/                #   /runs
      settings/            #   /settings
      tasks/               #   /tasks
      trades/              #   /trades
    lib/
      api/                 # Typed API client modules
      stores/              # Svelte writable stores
      components/          # Reusable Svelte components

tests/                     # pytest suite
docs/                      # project documentation
templates/workspace/       # agent workspace file templates
```

---

## Key Conventions

### Backend (Python)

- **Import style**: Always use absolute imports - `from forven.module import X`, never relative.
- **Router pattern**: Keep FastAPI endpoints thin and delegate business logic to focused modules.
- **Pipeline stages**: `researching -> backtesting -> paper -> deployed -> retired` (see `forven/policy.py`).
- **Type hints**: All function signatures should have type hints.
- **Linter**: Ruff.
- **Tests**: pytest under `tests/`.
- **Async**: FastAPI endpoints are async where appropriate; heavy compute can be offloaded.

### Frontend (SvelteKit / TypeScript)

- **API calls**: Route backend communication through `frontend/src/lib/api/`.
- **State**: Shared stores live in `frontend/src/lib/stores/`.
- **Styling**: TailwindCSS utility classes.
- **Components**: Reusable UI belongs in `frontend/src/lib/components/`.

---

## Running the Project

```powershell
# Full stack (recommended on Windows)
powershell -ExecutionPolicy Bypass -File .\start_all.ps1

# Full stack (macOS/Linux)
bash start_all.sh

# Backend only
python -m uvicorn --app-dir . forven.api:app --host 127.0.0.1 --port 8003 --reload

# Frontend only
cd frontend
npm run dev

# CLI
python -m forven --help

# Tests
python -m pytest tests -q

# Linting
python -m ruff check forven tests
```

Important:

- `python -m forven` launches the CLI, not the API server.
- `start_all.ps1` is the most complete bootstrap path on Windows and can auto-create `.venv` plus install missing dependencies.

---

## Important Patterns To Follow

1. **Adding a new backend endpoint**
   - Create or edit a router in `forven/routers/`
   - Add business logic in a focused backend module
   - Register the router in `forven/api.py` if it is new
   - Add a corresponding API wrapper in `frontend/src/lib/api/`

2. **Adding a new strategy**
   - Extend `BaseStrategy` from `forven/strategies/base.py`
   - Place it in `forven/strategies/builtin/` or `forven/strategies/custom/`
   - Register it through `forven/strategies/registry.py`

3. **Adding a frontend route**
   - Create `frontend/src/routes/<name>/`
   - Add `+page.svelte` and optional loader files
   - Add typed API client functions if the route needs new backend data

---

## Do NOT

- Commit `.env`, `*.db`, auth tokens, or files in `.forven_home/`
- Modify `forven/exchange/` without explicit instruction
- Use relative imports in backend code
- Put business logic directly in router files
- Use raw `fetch()` in Svelte components when a typed API client belongs in `frontend/src/lib/api/`
- Install new Python dependencies without updating `pyproject.toml`

---

## Driving Forven programmatically (no MCP) — `forven.agent`

The Forven MCP server is only a thin **stdio wrapper** over the backend REST API
on `:8003` (the same API the frontend uses). When you can't use MCP — Codex, the
Tauri app, a sidecar, CI, or when MCP drops — use the **zero-dependency HTTP
harness** instead. It does everything MCP does.

**Shell (Claude Code / Codex):** every command prints JSON to stdout.
```bash
python -m forven.agent health
python -m forven.agent context --out .tmp/ctx.json     # datasets, template, param families (large)
python -m forven.agent list --status paper
python -m forven.agent gate-report S02545              # why a strategy is/isn't promotable
# write a strategy .py to forven/strategies/custom/, then one-shot the genuine pipeline:
python -m forven.agent enqueue --file /abs/path/strat.py --dataset BTC/USDT-1h
python -m forven.agent wait-paper --strategies S02545,S02604 --timeout 1800
```
Also installed as the `forven-agent` console script. Full command list + the gate
reality (quick_screen / cost_stress / deflated-Sharpe) are in `forven/agent/README.md`.

**Python (sidecars/embedding):**
```python
from forven.agent import ForvenAgentClient
fc = ForvenAgentClient()                       # http://127.0.0.1:8003, env-overridable
verdict = fc.enqueue_candidate("/abs/strat.py", "BTC/USDT-1h")   # register→backtest→screen→promote (force=false)
```

**In-app / Tauri / browser (TypeScript):** use `frontend/src/lib/api/agent.ts`
(`ForvenAgent`), which reuses the app's `fetchApi` (auth + base discovery):
```ts
import ForvenAgent from '$lib/api/agent';
const v = await ForvenAgent.enqueueCandidate('/abs/strat.py', 'BTC/USDT-1h');
```

Rules: never pass `force=true` to skip a gate; set `compatible_regimes =
["trending","volatile","range_bound"]` on custom strategies; no `stop_loss_pct`
in `default_params`. Auth (only if `:8003` is exposed beyond localhost): set
`FORVEN_API_KEY` / `FORVEN_OPERATOR_KEY`.

---
> Source: [judder659/Forven](https://github.com/judder659/Forven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-23 -->
