## highlowticker-tui

> Free tier supports **Coinbase** (crypto, free) and **Yahoo Finance** (equity polling).

# HighlowTicker Free — Developer Notes

## What's included

Free tier supports **Coinbase** (crypto, free) and **Yahoo Finance** (equity polling).
Equity real-time streaming (Schwab, Alpaca, Tastytrade, IBKR, etc.) requires Pro.
See https://highlowtick.com to upgrade.

## Architecture

```
app.py                         # Textual TUI entry point
core/
  app_config.py                # Config loader (~/.highlowticker/config.toml)
  high_low_ticker.py           # Per-symbol high/low detection engine
  license.py                   # RSA license validation
  provider_loader.py           # Instantiates providers from config + env
config/
  highlight.json               # Thresholds + colors (user-editable at runtime)
providers/
  coinbase_provider.py         # Coinbase Advanced Trade WebSocket (free)
  yahoo_provider.py            # Yahoo Finance REST polling (free)
  _subscription.py             # wall_clock_counts rate windows
  _volume.py                   # Volume spike detection
tickers/
  tickers.json                 # Default symbol list
  spy_tickers.json / dow / qqq # Index constituent lists
```

## Data flow

```
Provider WebSocket/REST → EquityTick / CryptoTick
  → HighLowTicker.process_stock()
  → asyncio.Queue  {"type": "HIGHLOW_UPDATE", "data": state}
  → app._data_loop()
  → app._apply_highlow_update()
  → app._refresh_ui()
```

## Performance rules — do not break these

- **Widget refs are cached** in `on_mount` as `self._w_*`. Never call `query_one` inside
  `_refresh_ui` — it walks the DOM every call.
- **`await asyncio.sleep(0)`** after `_refresh_ui()` in `_data_loop` is intentional.
  Yields the event loop so Textual can flush renders and handle keyboard input.
- **Tables only rebuild when dirty** (`_highs_dirty` / `_lows_dirty` flags). Don't
  remove this gate — in a quiet market the tables should do zero work per tick.
- **`add_columns` must only run in `on_mount`**. `DataTable.clear()` keeps columns.
- **`MAX_TABLE_ROWS = 50`**. A terminal shows ~25 rows per table at most.

## Highlight logic (priority order)

1. `flash_high` / `flash_low` — index 0 (most recent entry)
2. `week52_high` / `week52_low` — symbol at 52-week extreme
3. `yellow` — count == 1 (first occurrence this session)
4. `orange` — symbol in ≥ `consecutiveCount+1` contiguous rows
5. `purple` — pct change delta > `significantPercentChange`
6. `default`

Thresholds are in `config/highlight.json`, hot-reloadable via `s` → Settings.

## Rate bars

`highCounts` / `lowCounts` count timestamps in 30s / 1m / 5m / 20m windows.
Timestamps are pruned to 1200s — do not reduce this or the 20m bar silently shortens.

---
> Source: [jach8/highlowticker-tui](https://github.com/jach8/highlowticker-tui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
