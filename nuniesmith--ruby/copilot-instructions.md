## ruby

> > **Repo:** `github.com/nuniesmith/ruby`

# ruby — Claude Code Project Instructions

> **Repo:** `github.com/nuniesmith/ruby`
> The Python trading system for FKS. Built into the `nuniesmith/fks:ruby`
> Docker image by the **fks** orchestration repo (via `git clone` at
> `${RUBY_REF:-main}`, or by copying `src/ruby/` from its tree). Infrastructure
> (Docker, compose, nginx, Prometheus, CI/CD) lives in **fks**, not here.

## What this is

Market-data ingestion + signal generation + CNN model training + execution
routing + paper trading, exposed as FastAPI REST/SSE APIs. In production it
runs as several supervised processes in one container (data API on :8000 is
the primary; engine + factory + web run alongside). Janus consumes the data
API as the **single source of truth** for market data.

## Stack

| | |
|--|--|
| Python | 3.13+ (code uses PEP 701 f-strings — will not parse on ≤3.11) |
| Web | FastAPI + uvicorn (+ HTMX/Jinja2 on the legacy web app) |
| ML | PyTorch (CPU + CUDA), scikit-learn, numpy, pandas |
| Data | Postgres (psycopg), Redis, QuestDB (PG wire) |
| Lint / types | ruff, mypy |
| Tests | pytest, pytest-asyncio (~5,300 test functions across 76 files) |

## Repository layout

```
ruby/
├── pyproject.toml         # deps + ruff/mypy/pytest config (build: hatchling)
├── sql/                   # Postgres migrations 001–008 (ruby_db schema)
├── tests/                 # pytest suite (mirrors src/)
└── src/                   # application code — TOP-LEVEL packages live here
    ├── main.py            # futures supervisor entry (python -m src.main)
    ├── core/              # asset registry, schema/DDL, config, logging_config
    ├── analysis/          # ICT, CVD, confluence, regime, sentiment, ML inference
    ├── indicators/        # EMA, RSI, VWAP, ATR, Bollinger, volume profile, patterns
    ├── trading/           # ORB/breakout, swing detector, daily bias, multi-TP
    ├── integrations/      # Kraken, Rithmic, Massive, TradingView, Pine, Grok
    ├── model/ · ml/       # ensemble / deep / statistical models + CNN pipeline
    ├── data/              # data factory: gaps, backfill, chain, news, datasets
    ├── services/          # execution gate, risk engine, redis store, data API
    ├── workers/           # per-asset async workers
    └── web/               # legacy HTMX app (API-only split pending — see TODO)
```

## Imports & how to run

- **Code imports its packages as top-level** — `from core.logging_config import
  get_logger`, `from services.risk_engine import ...`. So **`src/` must be on
  `PYTHONPATH`**. The container sets `PYTHONPATH=/app/src`; locally use `src/`.
  A handful of modules use the `src.`-prefixed form, so `pytest` puts both
  `src` and `.` on the path (see `[tool.pytest.ini_options]`).

```bash
# Install (editable, with dev tooling)
python3.13 -m venv .venv && source .venv/bin/activate
pip install -e ".[dev]"        # add [test] for the full ML test deps

# Tests (pytest config wires PYTHONPATH automatically)
pytest                          # all
pytest -k test_orb -v           # one
pytest --cov=src                # with coverage

# Lint / format / typecheck
ruff check src/
ruff format src/
mypy src/

# Run the data service locally
PYTHONPATH=src uvicorn services.data.main:app --host 0.0.0.0 --port 8000

# Build/run the full stack: from the fks repo
#   ./run.sh all   |   docker compose up -d ruby   |   docker compose logs -f ruby
```

## Code conventions

- **Logging:** always the project logger; never `print()` / `logging.basicConfig()`:
  ```python
  from core.logging_config import get_logger   # (or utils.logging_config)
  logger = get_logger("my.module.name")
  ```
- **Type hints** on public functions; `from __future__ import annotations` at module top.
- **Async:** FastAPI endpoints are `async def`; CPU-bound work goes in `asyncio.to_thread()`.
- **Config:** env vars via `os.getenv()` with defaults — never hardcode. `.env` is gitignored.
- **Tests** mirror the source tree: `src/foo/bar.py` ↔ `tests/test_bar.py`.

## Critical invariants

- **No autonomous execution.** Every order routes through
  `services.execution_gate`, which requires explicit operator confirmation.
  `EXECUTION_MODE=paper_trading` is the default — never default to `live`.
- **Ruby is the data source of truth.** Janus pulls bars from
  `http://fks_ruby:8000` (`PYTHON_DATA_SERVICE_URL`). Don't add a second data path.
- **SQL schema is owned here** (`sql/001–008`); fks's Postgres image applies it.

## Known issue — runtime module paths (in-flight layout migration)

The code is **`src/`-rooted with top-level packages** (run as
`services.data.main`, `src.main`). However, some artifacts still reference a
planned `lib/` + `entrypoints/` layout that has **not** landed:

- fks `infrastructure/docker/services/ruby/supervisord.conf` launches
  `entrypoints.data.main`, `lib.services.engine.main`, `entrypoints.factory`,
  `lib.services.web.main` — none of which exist here.
- Several module docstrings say `lib.services.*` / `PYTHONPATH=src` for modules
  that are actually importable as `services.*`.
- There is **no `services/engine/` entry module** yet (referenced by supervisord).

Before the fks stack can start the ruby container, the supervisord
commands + Dockerfile `APP_SRC`/`PYTHONPATH` must be aligned with this layout
(or the code moved to `lib/`+`entrypoints/`). Pick one direction and apply it
consistently across both repos.

---
> Source: [nuniesmith/ruby](https://github.com/nuniesmith/ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
