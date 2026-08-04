## vooi-signals-bot-example

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

The bot uses Typer; every command is `python -m bot <cmd>` (or `bot <cmd>` after `pip install -e .`).

| Need | Command |
|---|---|
| Start all 7 async tasks (foreground) | `python -m bot run` |
| Connectivity / DB / VOOI health | `python -m bot status` |
| Interactive Telethon login (one-time) | `python -m bot tg-login` |
| Add a channel | `python -m bot channels add @username` |
| Re-parse a stored message without trading | `python -m bot signal-dryrun <signal_id>` |
| Force a breakeven check on one position | `python -m bot simulate-breakeven --position-id <N>` |
| Export VOOI errors for triage | `python -m bot vooi-errors export out.json` |
| Apply DB migrations | `alembic upgrade head` (also runs automatically on `bot run`) |
| Unit tests (no external deps) | `pytest tests/unit/` |
| One test file or one test | `pytest tests/unit/test_tp_calculator.py::test_floor_applied` |
| Integration tests (needs local Postgres) | `pytest tests/integration/` |

`pyproject.toml` sets `asyncio_mode = "auto"`, so async tests do **not** need `@pytest.mark.asyncio`.

### One-off operator scripts

These exist under `scripts/` and are not part of `bot run`. Run with the bot stopped:

- `python -m scripts.reset_tp [--apply]` — recompute TP on every open position and replace the live trigger. Dry-run by default.
- `python -m scripts.reset_sl [--apply]` — same for SL (use after changing `DEFAULT_SL_PCT` or flipping `USE_SIGNAL_SL`).
- `python -m scripts.mae_winners` — pull 1m Binance klines for every `closed_tp` position and report Maximum Adverse Excursion. Useful when tuning SL distance from empirical data.

## Architecture

A single Python process runs **seven concurrent asyncio tasks** plus a one-shot `startup_cleanup`. They share a Postgres DB and an `httpx`-based VOOI client.

```
ingester → parser → router → place_entry  ──(SSE order frame, status=filled)──>  post_fill_placer
                                                                                 │
                                                                                 ▼
                                                                       places SL + TP triggers
                                                                                 │
                                       sse_listener.on_market_price_frame
                                            └─▶ evaluate_breakeven_trigger  (in-memory index)
                                                     └─▶ asyncio.create_task → move_sl_to_breakeven
                                       breakeven_supervisor (10s) — heartbeat + reconcile + stale-SSE rescue
                                       sl_safety_check      (30s)
                                       tp_safety_watchdog   (30s)   re-places missing TPs
                                       lighter_sl_watchdog  (30s)   re-places dropped SLs
                                       reconciler           (60s)   DB ↔ exchange diff
```

Tasks are registered in `bot/cli.py:_run_all_tasks`. Breakeven detection is **SSE-driven, not polled**: every `marketPrice` frame triggers `evaluate_breakeven_trigger`, which looks up the in-memory `trigger_index` (keyed by `(exchange, symbol)`) and schedules a BE move via `asyncio.create_task` if the threshold is crossed. Per-trigger `asyncio.Lock` + `fired` flag guarantee idempotency under torrents of ticks. The supervisor's only jobs are (a) update `tp_breakeven_watcher_last_tick` so `sl_safety_check` doesn't flag `ERROR_WATCHER_HUNG`, (b) reconcile `trigger_index` against the DB every 10s (self-heals missed register/unregister calls), and (c) batch-quote REST for positions whose SSE feed has gone stale — the only place breakeven logic ever hits REST.

### The signal funnel — what filters out what

Reading `signals.is_signal` and `signals.skip_reason` is the fastest way to debug "why didn't my signal trade":

```
message  →  parsed signal (is_signal=true)  →  router gates  →  entry order  →  fill  →  post_fill (SL+TP)
              else: stored as noise           else: skip_reason   else: rejected  else: open_pending_tp_sl
```

Common `skip_reason`s: `symbol_not_found`, `rate_limit_global`, `rate_limit_per_channel`, `already_in_position`, `symbol_active_on_all_available_exchanges`, `duplicate_signal`, `no_entry_price`, `leverage_set_failed`. Market-orders are **forbidden** in v1 — empty `entry_prices` → `no_entry_price`.

### TP and SL math — read this before touching `tp_calculator.py`

**All `*_PCT` settings in the trading section are denominated as % of MARGIN (collateral), not price.** The bot converts to price by dividing by leverage. This applies to:

- `DEFAULT_SL_PCT`           (e.g. 5 → at lev=5, SL is 1% from entry → loses 5% of margin if hit)
- `MIN_PROFIT_PCT_OF_COLLATERAL` (required net profit)
- `TP_OVERHEAD_FLOOR_PCT`    (floor on cost-overhead allowance)
- `BREAKEVEN_TRIGGER_PCT`    (when to move SL to breakeven)

So the TP formula in `compute_tp_price` is:

```
required_gain (price) = MIN_PROFIT_PCT_OF_COLLATERAL / 100 / L
floor         (price) = TP_OVERHEAD_FLOOR_PCT       / 100 / L
cost_overhead (price) = max((quote_fees*2 + slippage + funding) / 10000, floor)
total                 = required_gain + cost_overhead
TP_long  = entry × (1 + total)
TP_short = entry × (1 − total)
```

`USE_SIGNAL_SL=false` (default) means the bot **ignores** any `stop_loss` from the signal and always derives SL from `DEFAULT_SL_PCT`. Setting it to `true` honors the signal SL when present; the bot still falls back to `DEFAULT_SL_PCT` if the signal omitted one.

**Caveat at lev≥7**: BE-SL uses a fixed ~8 bps buffer (in price). If `BREAKEVEN_TRIGGER_PCT / leverage < 0.08%`, the BE move places SL above the current price → instant trigger and near-zero close. Raise `BREAKEVEN_TRIGGER_PCT` when raising leverage above 6.

### Order placement — what the bot sends to VOOI

- **Entries are limits only.** No bracket fields (`stopLoss`/`takeProfit`) inline; Aster rejects them and the bot would lose accuracy on the TP calc.
- **TP and SL are placed after fill** by `post_fill_placer`, as separate `reduceOnly=true` trigger orders. The SSE `order` frame with `status='filled'` is the trigger; reconciler is the fallback if SSE is missed.
- **No broker/builder field is sent.** VOOI assigns the broker server-side from the API key; `quote.feesBps` already includes the server-side fee. Do not re-add `broker.{id,feeBps}` to any order body — that's a regression.

### Per-exchange quirks worth knowing

- **Lighter** returns `clientOrderId: null` in `/exchange/open-orders`, which breaks the standard lookup path. `place_trigger_with_verification` falls back to attribute matching (`asset/side/triggerPrice/size/type`). Search for `lighter silent-NAKED` in code/logs for context.
- **Lighter** occasionally drops live SL trigger orders for opaque reasons (TTL / funding). `lighter_sl_watchdog_task` detects this and re-places. Rate-limit: 3 attempts/hour/position.
- **Aster** does not support bracket fields in the entry POST — never send `stopLoss`/`takeProfit` with the entry. The post-fill flow handles all three exchanges uniformly.
- **Aster** intermittently returns `401 Invalid credentials` on *mutating* calls (POST/DELETE) even while reads on the same key succeed — an upstream session glitch, not a dead key. Two consequences are handled defensively: (a) `cancel_expired_limit_orders` never marks an order `expired` on an unconfirmed cancel, so a `401`'d cancel does not desync a still-live order; (b) aster can also lag `/exchange/positions` behind `/exchange/trades` by a long time, so `reconciler.detect_entry_fills_from_history` cross-checks `/exchange/trades` to recover fills that `/positions` + SSE both missed. For triage, `ASTER_DEBUG_LOG_PATH` captures full per-request NDJSON (headers + bodies + `fly-request-id`) for every aster call.
- **Hyperliquid** requires a `0x`-prefixed 32-hex-char `clientOrderId`. `make_client_order_id()` returns the right format per exchange — always use it.

### TP retry on transient VOOI failures

`tp_safety_watchdog_task` (new) handles the case where VOOI returns 503 during the post-fill TP POST and the inline 3-retry exhausts. Every 30s it scans `status='open'` positions with no live TP and re-places. It **does not** skip positions where BE-SL has already fired — BE-SL only protects the downside, but without TP the position has no programmed profit-take and would have to be closed by hand. Rate-limit: 6 attempts/hour/position; on exhaustion → `ERROR_NO_TP`.

The companion `sl_safety_check` now emits `ERROR_NO_TP` as well as `ERROR_NAKED_POSITION` so missing TPs surface in alerts.

### Key DB invariants

- `positions.entry_order_id` is **NOT NULL** — a position row exists iff its entry filled.
- `positions.sl_order_id` may be NULL briefly between fill detection and post-fill placement. If still NULL after `status='open'` for >30s, `sl_safety_check` fires `ERROR_NAKED_POSITION`.
- `positions.sl_strategy = 'fixed'` means SL came from the signal; `'computed_margin_pct'` means it was derived from `DEFAULT_SL_PCT`.
- `orders.vooi_order_id` is the exchange-side ID — recovered via `clientOrderId` lookup or attribute match when VOOI's POST returns `{status:"ok"}` without an ID.

### Adding a new bot command

1. Add a `@app.command()` (or sub-typer) in `bot/cli.py`.
2. If it touches the DB, use `async with session_scope():`.
3. If it talks to VOOI, use `get_vooi_client()` (singleton with retry/redaction).
4. CLI commands run via `asyncio.run(...)` from inside the sync Typer handler — match the existing pattern.

### Reference docs

- `docs/tech-spec.md` is the source of truth for business logic and the VOOI API contract. The TP/SL formula derivations live in §8.6.
- `README.md` covers deployment paths (Docker / native / VPS) and the configuration table for `.env`.

---
> Source: [vooi-app/vooi-signals-bot-example](https://github.com/vooi-app/vooi-signals-bot-example) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
