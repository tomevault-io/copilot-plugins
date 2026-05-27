## test

> This document is written for an AI coding agent (Claude Sonnet 4.6) working on this

# AGENTS.md — Polymarket BTC Backtesting Tool

This document is written for an AI coding agent (Claude Sonnet 4.6) working on this
codebase. Read this before touching any file.

---

## Project Purpose

Backtest trading strategies on Polymarket BTC "Up or Down" 5-minute and 15-minute
prediction markets using real historical CLOB order-book data.

The tool does NOT trade live. It replays recorded market episodes, feeds each
~100 ms snapshot to a strategy, simulates realistic order fills against the real
CLOB event stream, and records every trade in full detail for analysis via a
Streamlit UI.

---

## Data Source

**Dataset**: `trentmkelly/polymarket_crypto_derivatives` (HuggingFace)
**Coverage**: 2026-02-21 → 2026-03-24  
**Size**: ~32.3 GB  
**Symbols**: BTC, ETH, SOL, XRP — but we filter to BTC only.  
**Intervals**: 5-minute and 15-minute markets.

### Episode Directory Layout

Each market episode is one directory on HuggingFace:

```
btc5m_market<id>_<window_start_unix_s>_all/
  steps.parquet        # one row per ~100ms decision snapshot
  events.parquet       # CLOB events attached to the following step
  book_levels.parquet  # full orderbook depth per step
```

### steps.parquet Schema

| Column | Type | Description |
|---|---|---|
| step_index | int | monotonically increasing within episode |
| ts | int | Unix milliseconds |
| progress | float | fraction of market lifetime elapsed (0 → 1) |
| dt_s | float | seconds since previous step |
| hour | int | UTC hour |
| chainlink_price | float | BTC/USD from Chainlink (this is the resolution oracle) |
| binance_price | float | BTC spot from Binance |
| up_best_bid | float | CLOB best bid for UP token (0–1) |
| up_best_ask | float | CLOB best ask for UP token (0–1) |
| up_mid | float | (best_bid + best_ask) / 2 for UP |
| up_spread | float | best_ask - best_bid for UP |
| up_bid_size_total | float | total USD on bid side for UP |
| up_ask_size_total | float | total USD on ask side for UP |
| up_imbalance | float | (bid_size - ask_size) / (bid_size + ask_size) for UP |
| down_best_bid | float | same fields for DOWN token |
| down_best_ask | float | |
| down_mid | float | |
| down_spread | float | |
| down_bid_size_total | float | |
| down_ask_size_total | float | |
| down_imbalance | float | |

### events.parquet Schema

| Column | Type | Description |
|---|---|---|
| following_step_index | int | these events occurred before this step |
| event_index | int | order within step |
| event_type | int | 1=trade, 2=price/book update, 3=tick size change |
| ts | int | Unix milliseconds |
| is_down | bool | True = event is for DOWN token, False = UP |
| is_sell | bool | True = sell order |
| is_sell_side | bool | True = from sell side perspective |
| price | float | Polymarket contract price (0–1) |
| size | float | CLOB token size |
| old_tick_size | float | (type 3 only) |
| new_tick_size | float | (type 3 only) |

### book_levels.parquet Schema

| Column | Type | Description |
|---|---|---|
| step_index | int | matches steps.parquet step_index |
| outcome | int | 0=UP, 1=DOWN |
| side | int | 0=bid, 1=ask |
| level_index | int | 0=best, 1=second-best, ... |
| price | float | price level (0–1) |
| size | float | total size at this price level |

---

## Architecture Overview

```
HuggingFace Dataset
        │
        ▼
data/downloader.py     ── downloads & caches parquet files locally
        │
        ▼
data/loader.py         ── DuckDB queries → assembles MarketEpisode objects
        │
data/filter.py         ── filters episodes list by symbol/interval/date
        │
        ▼
engine/backtester.py   ── main loop
    for each episode:
        strategy.reset()
        for each step (chronological):
            book = build BookSnapshot from step row
            events = events for this step
            order = strategy.on_step(step, book, events, position)
            if order and position is None:
                fill = fill_simulator.try_fill(order, step, future_events)
                if fill: create ActivePosition
        on resolution:
            close position → create TradeRecord
        │
        ▼
analytics/run_result.py  ── collects TradeRecords → BacktestResult → saves JSON
        │
        ▼
ui/app.py  (Streamlit)   ── loads run JSONs → 3-page UI
```

---

## Data Layer (`data/`)

### `models.py`

Defines all dataclasses used across the system. DO NOT add business logic here.

- `Step`: one row from steps.parquet, all fields as Python floats/ints
- `BookSnapshot`: wraps UP and DOWN `SideSnapshot` objects (convenient access)
- `SideSnapshot`: best_bid, best_ask, mid, spread, imbalance, bid_size_total, ask_size_total
- `CLOBEvent`: one row from events.parquet
- `BookLevel`: one row from book_levels.parquet
- `MarketEpisode`: market_id, interval, window_start_ts, steps[], events[], book_levels[], resolution

### `downloader.py`

Downloads episodes from HuggingFace Hub into a local `./cache/` directory.
Supports partial download (skip already-cached episodes).
Exposes `list_btc_episodes()` → list of episode directory names.
Exposes `ensure_episode_cached(episode_name)` → local path.

### `loader.py`

Uses DuckDB to read parquet files without loading everything into RAM.
Exposes `load_episode(local_path) -> MarketEpisode`.
Reads steps, events, and book_levels lazily.

### `filter.py`

Exposes `filter_episodes(episodes, symbol, interval, date_from, date_to)`.
Episode names follow `<symbol><interval>m_market<id>_<ts>_all` pattern — parse these.

---

## Engine Layer (`engine/`)

### `order.py`

```python
@dataclass
class Order:
    side: str           # "UP" or "DOWN"
    order_type: str     # "LIMIT" or "MARKET"
    price: float        # for LIMIT: the limit price (0–1); ignored for MARKET
    size_usd: float     # how many USD to spend (converted to tokens at fill price)
```

### `position.py`

```python
@dataclass
class ActivePosition:
    side: str           # "UP" or "DOWN"
    fill_price: float
    tokens: float
    cost_usd: float
    fee_usd: float
    entry_ts: int       # Unix ms
    entry_step_index: int
    entry_progress: float
    order_type: str     # "LIMIT" or "MARKET"
    is_maker: bool
    # signals recorded at entry (for TradeRecord):
    chainlink_price_at_entry: float
    binance_price_at_entry: float
    mid_at_entry: float
    imbalance_at_entry: float
    spread_at_entry: float
```

### `fees.py`

```python
TAKER_FEE_RATE = 0.07  # crypto market rate

def calc_taker_fee(shares: float, price: float) -> float:
    return round(shares * TAKER_FEE_RATE * price * (1 - price), 6)

MAKER_FEE = 0.0
```

### `fill_simulator.py`

`try_fill(order, step, future_events) -> FillResult | None`

**LIMIT order logic:**
- For BUY UP: look through `future_events` (event_type=1, is_down=False) for any trade at `price <= order.price`
- For BUY DOWN: look through (event_type=1, is_down=True) for `price <= order.price`
- First matching event = fill. Timestamp from the event. Role = MAKER, fee = 0.
- IMPORTANT: `future_events` are only events from steps AFTER the current step (no look-ahead).

**MARKET order logic:**
- BUY UP: fill at `step.up_best_ask` immediately (current step). Role = TAKER.
- BUY DOWN: fill at `step.down_best_ask`. Role = TAKER.
- Apply `calc_taker_fee`.

### `backtester.py`

Main loop pseudocode:
```
results = []
for episode in filtered_episodes:
    strategy.reset()
    position = None
    pending_order = None
    future_events_buffer = []

    for i, step in enumerate(episode.steps):
        step_events = events_for_step(episode, step.step_index)
        book = make_book_snapshot(step)

        # Try to fill a pending limit order against this step's events
        if pending_order and position is None:
            fill = try_fill_limit(pending_order, step_events)
            if fill:
                position = make_position(fill, pending_order, step_context)
                pending_order = None

        # Ask strategy for a new order (only if no position)
        if position is None and pending_order is None:
            order = strategy.on_step(step, book, step_events, None)
            if order:
                if order.order_type == "MARKET":
                    fill = try_fill_market(order, step)
                    position = make_position(fill, order, step_context)
                else:
                    pending_order = order  # will try next steps

        else:
            # Pass current state to strategy even with open position
            strategy.on_step(step, book, step_events, position)

    # Resolve at end of episode
    if position:
        trade = resolve_position(position, episode.resolution, episode)
        results.append(trade)

return BacktestResult(strategy=strategy, trades=results)
```

---

## Analytics Layer (`analytics/`)

### `trade_log.py` — `TradeRecord`

All fields listed in the plan. Fully serializable to JSON via `dataclasses.asdict`.

### `metrics.py` — `calc_metrics(trades: list[TradeRecord]) -> AggregateMetrics`

Compute all aggregate metrics. See plan for full list.

Max drawdown formula:
```python
cumulative = np.cumsum([t.pnl_usd for t in trades])
peak = np.maximum.accumulate(cumulative)
drawdown = peak - cumulative
max_drawdown = float(drawdown.max()) if len(drawdown) > 0 else 0.0
```

Sharpe ratio (per-trade):
```python
pnls = np.array([t.pnl_usd for t in trades])
sharpe = float(pnls.mean() / pnls.std()) if pnls.std() > 0 else 0.0
```

### `run_result.py` — `BacktestResult`

Saved to `runs/<strategy_name>_<timestamp>.json`.
Contains `strategy_name`, `strategy_description`, `run_ts`, `trades` (list), `metrics` (dict).

---

## Strategy Layer (`strategies/`)

### How to Add a New Strategy

1. Create `strategies/my_strategy.py`
2. Extend `Strategy` from `strategies/base.py`
3. Set `name` and `description` class attributes
4. Implement `on_step(step, book, events, position) -> Order | None`
5. Optionally override `reset()` to clear internal state
6. Import and add to `strategies/__init__.py` `ALL_STRATEGIES` list

### Rules for Strategy Authors

- **Never** read data beyond the current `step` object. That is look-ahead bias.
- `position is not None` means you already have an open bet. Return `None` in that case.
- To enter UP: `Order(side="UP", order_type="MARKET", price=0.0, size_usd=10.0)`
- To enter with a limit: `Order(side="UP", order_type="LIMIT", price=0.45, size_usd=10.0)`
- Limit orders that are not filled by resolution expire with no effect.
- `reset()` is called before EVERY new episode — reset all running state (counters, buffers, etc).

### Available Signals Reference

```
step.chainlink_price    BTC/USD from Chainlink (resolution oracle)
step.binance_price      BTC spot from Binance
step.progress           fraction of market lifetime elapsed [0, 1]
step.ts                 Unix milliseconds

book.up.mid             CLOB mid price for UP token [0, 1]
book.up.best_bid        best bid for UP
book.up.best_ask        best ask for UP
book.up.spread          best_ask - best_bid for UP
book.up.imbalance       (bid_size - ask_size) / total — positive = buy pressure
book.up.bid_size_total  total USD on UP bid side
book.up.ask_size_total  total USD on UP ask side

book.down.*             same fields for DOWN token

events                  list[CLOBEvent] since previous step
  .event_type           1=trade  2=price/book change  3=tick size change
  .price                contract price of the event
  .size                 token size
  .is_down              True=DOWN token event
  .is_sell              True=sell side

position                ActivePosition | None — current open position
  .side, .fill_price, .tokens, .entry_progress, .entry_ts, ...
```

---

## UI Layer (`ui/`)

### Entry Point

```bash
streamlit run ui/app.py
```

### How Run Results Are Loaded

`ui/app.py` scans `runs/*.json`, loads all `BacktestResult` objects, and stores them
in `st.session_state["runs"]`. All pages read from that shared state.

### Page 1 — Dashboard (`01_dashboard.py`)

- Metrics comparison table (all strategies × all metrics)
- Overlaid equity curves (plotly line chart, one trace per strategy)
- Win rate bar chart
- Max drawdown comparison bar chart

### Page 2 — Strategy Detail (`02_strategy_detail.py`)

- Select a strategy from sidebar
- Full metrics card (`st.metric` grid)
- Equity curve with trade entry markers (plotly scatter overlay on line chart)
- PnL distribution histogram
- Entry price distribution histogram
- Progress-at-entry scatter plot

### Page 3 — Trade Inspector (`03_trade_inspector.py`)

- Filterable/sortable trade log table (`st.dataframe` with column filters)
- Click any row → expand panel showing:
  - Full 100ms CLOB mid price chart for the episode (UP and DOWN lines, plotly)
  - BTC Chainlink price on second y-axis
  - Vertical line + annotation at entry_ts
  - Orderbook depth heatmap at entry step (book_levels.parquet data)
  - Full event log table for that episode

### Adding a New Chart or Metric

1. Add the metric to `AggregateMetrics` in `analytics/metrics.py`
2. Compute it in `calc_metrics()`
3. It is automatically included in the JSON and loaded by the UI
4. Reference it in the relevant page via `run.metrics["your_metric"]`

---

## Conventions

| Item | Convention |
|---|---|
| Timestamps | Always Unix **milliseconds** internally |
| Prices | Always `float` in `[0.0, 1.0]` (Polymarket contract price) |
| USD amounts | `float`, rounded to 6 decimal places |
| Side | Always string literal `"UP"` or `"DOWN"` |
| Order type | Always string literal `"LIMIT"` or `"MARKET"` |
| Episode IDs | Raw directory name from HuggingFace, e.g. `btc5m_market12345_1778662200_all` |
| Cache dir | `./cache/` relative to project root (git-ignored) |
| Run files | `./runs/<strategy_name>_<ISO_timestamp>.json` |

---

## Common Pitfalls

1. **Look-ahead bias**: `future_events` in fill_simulator are only events with
   `following_step_index > current_step_index`. Never peek at step N+k data in a strategy.

2. **Episode reset**: `strategy.reset()` is called before every episode.
   If your strategy has internal state (e.g., a price history buffer), clear it in `reset()`.

3. **Maker vs taker at placement**: A LIMIT order that is placed at or above the
   current `best_ask` immediately crosses the spread — it should be treated as TAKER,
   not MAKER. The fill simulator must detect this case.

4. **Pending order expiry**: If a LIMIT order is pending at resolution (never filled),
   it expires silently. No trade is recorded.

5. **Resolution value**: UP token = $1.00 if market resolved "Up", else $0.00.
   DOWN token = $1.00 if market resolved "Down", else $0.00.

6. **Fee symmetry**: `fee = C × 0.07 × p × (1 - p)`. At p=0.50 this is 1.75 USD
   per 100 shares. At p=0.20 it is 1.12 USD. NEVER use a flat percentage.

7. **Size in USD vs tokens**: `Order.size_usd` is the USD budget.
   `tokens = size_usd / fill_price`. Track both in ActivePosition.

---
> Source: [tonio0/test](https://github.com/tonio0/test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
