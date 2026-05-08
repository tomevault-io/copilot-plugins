## polymarket-weather-quant

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## ⚠️ READ FIRST IF YOU'RE TRADING

If this session involves placing orders against a Polymarket wallet, read in this order:

1. **`AGENT_ONBOARDING.md`** — the entry point. Sequenced reading list + quick-start.
2. **`PLAYBOOK.md`** — THE rules. 15+ lessons from real P&L. Violations cost money.
3. **`WEATHER_QUANT_README.md`** — setup + architecture.
4. **`weather/cities.py::STATION_OFFSET_C`** — apply before computing cushion.

## Commands

Dependency management uses [UV](https://docs.astral.sh/uv/):

```bash
uv sync                     # install deps
uv sync --extra dev         # + jupyter/notebook/ipykernel
uv run python update_all.py # full pipeline (markets → goldsky → process)
```

Run a single pipeline stage:

```bash
uv run python -c "from update_utils.update_markets import update_markets; update_markets()"
uv run python -c "from update_utils.update_goldsky  import update_goldsky;  update_goldsky()"
uv run python -c "from update_utils.process_live    import process_live;    process_live()"
```

Fast backfill of the Goldsky stage (splits `last_synced → now` across workers, writes per-worker segment CSVs, then merges):

```bash
uv run python parallel_sync.py --workers 5
uv run python parallel_sync.py --workers 2 --end-ts 1767000000   # bounded test run
```

There is no test suite or linter configured.

## Architecture

Three-stage pipeline orchestrated by `update_all.py`. Each stage is **idempotent and resumable** — re-running picks up where the last run stopped. This is the central design property; new code must preserve it.

1. **`update_utils/update_markets.py`** — pages the Polymarket markets API in chronological order into `markets.csv`. Resumes by counting existing rows to compute the next offset.
2. **`update_utils/update_goldsky.py`** — queries the Goldsky `orderbook-subgraph` for `orderFilledEvents` into `goldsky/orderFilled.csv`. Resumes from the last timestamp in the CSV. Must paginate *within* a single timestamp when many events share it (the "sticky" cursor pattern — see `parallel_sync.py` for the full implementation, which `update_goldsky.py` mirrors).
3. **`update_utils/process_live.py`** — joins raw order events to markets and writes structured trades to `processed/trades.csv`. Resumes by finding the last processed `transactionHash`. When an event references a token not in `markets.csv`, it fetches that market on demand and appends it to `missing_markets.csv`.

`poly_utils/utils.py` exposes `get_markets()` (merges `markets.csv` + `missing_markets.csv`) and `PLATFORM_WALLETS` — import from `poly_utils`, not the submodule.

### Trade semantics (load-bearing)

- Each order event pairs USDC (assetId `"0"`) with one outcome token; map the non-USDC side to `token1`/`token2` on the market.
- Raw amounts are 10^6-scaled — divide before use.
- **Taker direction**: BUY if taker pays USDC, SELL if taker receives USDC. **Maker direction**: opposite. Price is always USDC per outcome token.
- When filtering trades for a specific user, filter on the `maker` column — Polymarket emits contract-level events from the maker's perspective, so maker-filtered rows are the complete set of that user's trades (not a subset).

### Parallel sync cursor logic

`parallel_sync.py` divides the time range into N segments and runs `sync_segment` workers concurrently. The non-obvious part is the cursor state machine inside each worker: on a full batch (`n >= BATCH_SIZE`) it decides between going "sticky" (paginate by `id_gt` within a fixed timestamp) vs. advancing `timestamp_gt` to the last *complete* timestamp in the batch. `STICKY_THRESHOLD` gates this to avoid tiny follow-up queries. Segments are written to `goldsky/parallel_segments/segment_<id>.csv` and appended to the main CSV in worker-id order (= timestamp order) during merge. `goldsky/cursor_state.json` is the authoritative resume point; the CSV tail is the fallback.

## Data locations

- `markets.csv`, `missing_markets.csv` — market metadata (tracked)
- `goldsky/orderFilled.csv` — raw events (auto-generated, large; first-time users should download the snapshot linked in `README.md` rather than backfilling from scratch)
- `processed/trades.csv` — joined trades (auto-generated)
- `goldsky/cursor_state.json`, `goldsky/parallel_segments/`, `parallel_logs/` — parallel sync state

---
> Source: [kevinbadi/polymarket-weather-quant](https://github.com/kevinbadi/polymarket-weather-quant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
