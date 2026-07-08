## pnl-monitor

> Tkinter desktop app that monitors intraday P&L across a UBS brokerage and UBS 401k account. Holdings come from `claudedev_shared`; prices come from Yahoo Finance via `yfinance`. Charts redraw on demand or on a 60 s auto-update loop.

# PnL-Monitor

## Overview
Tkinter desktop app that monitors intraday P&L across a UBS brokerage and UBS 401k account. Holdings come from `claudedev_shared`; prices come from Yahoo Finance via `yfinance`. Charts redraw on demand or on a 60 s auto-update loop.

## Environment
- **Conda env:** `PnL-Monitor` — interpreter `C:\Users\wamfo\anaconda3\envs\PnL-Monitor\python.exe`. Never use system Python.
- **Deps:** `environment.yml` (Python 3.13). pip pins are `>=X.Y,<X+1.0`: `yfinance`, `pandas`, `matplotlib`, `squarify`, `sv-ttk`, `claudedev_shared`, `pytest`.
- **Launch:** `main.py` (configures logging + `sv_ttk` light theme + `PnLApp`). `Launch PnL Monitor.bat` uses `pythonw.exe` for console-less double-click launch. User docs in `MANUAL.md`.
- **GitHub:** https://github.com/marange63/PnL-Monitor (branch `main`).

## Data Pipeline
- `ubs_live_price_holdings()` + `ubs_401k_holdings()` return DataFrames with `DESCRIPTION`, `SYMBOL`, `SOD VALUE`, `Ticker Alias`, `Tag`, `Source`.
- Prices via `yf.Ticker(t).fast_info` — use `last_price` and `regular_market_previous_close` (**not** `previous_close`, which includes after-hours).
- `load_and_compute()` concatenates both sources, fetches prices in parallel (`ThreadPoolExecutor(max_workers=5)`, one retry with 0.5 s backoff, failed tickers yield `(None, None, None)`), then fills `Last Price`, `Last Close`, `% Move On Day`, `PnL`.
- **Split auto-detect** (`_detect_split_ratio`): if `last_price / last_close < 0.35`, divide `last_close` by `round(1/ratio)`. Logs `detected N:1 split` INFO line — check the log before chasing "too good" prices.

## DataFrame Schema
Reference columns through `constants.Col` — never string literals.

| `Col.` attr | Column | Notes |
|---|---|---|
| `DESCRIPTION` | `DESCRIPTION` | Security full name |
| `SYMBOL` | `SYMBOL` | Brokerage symbol |
| `SOD_VALUE` | `SOD VALUE` | Start-of-day USD |
| `TICKER` | `Ticker Alias` | Yahoo ticker |
| `SOURCE` | `Source` | `"UBS"` or `"401K"` |
| `TAG` | `Tag` | Sector/category |
| `LAST_PRICE` | `Last Price` | from yfinance |
| `LAST_CLOSE` | `Last Close` | split-adjusted if detected |
| `PCT_MOVE` | `% Move On Day` | **decimal**, e.g. `0.0179` = 1.79% |
| `PNL` | `PnL` | `SOD VALUE * PCT_MOVE` |

## Modules
| File | Role |
|---|---|
| `main.py` | Entry point: logging, theme, `PnLApp` |
| `app.py` | `PnLApp` — wires control strip, panes, and `RunLoop`; owns toggle vars; persists custom tickers to `custom_tickers.json` (`MAX_CUSTOM_TICKERS = 4`) |
| `run_loop.py` | `RunLoop` — worker thread + auto-update timer + countdown tick; calls `on_result(RunResult)` on main thread |
| `data.py` | `load_and_compute`, `get_price_data`, `validate_ticker`, `get_default_drawdowns`, `get_drawdowns`, `get_intraday_prices`, `HighCache`; `DEFAULT_TICKERS`, `INTRADAY_TICKERS` |
| `charts.py` | `draw_scatter`/`draw_bar`/`draw_treemap` + their `build_*_df` companions; `dollar_fmt`, `pct_fmt` |
| `constants.py` | `Col`, color palette, button colors, sizing/timing constants |
| `drawdown_table.py` | `DrawdownTable` — ticker/6W/ATH table (+ optional Today column via `show_today`); editable variant adds × delete labels, Add row, async validation, height-sync to sibling |
| `bar_chart_pane.py` | `ScrollableBarChart` — debounced resize, mousewheel scroll, hover; used for both ticker bar and tag bar |
| `intraday_panel.py` | `IntradayChartGrid` — row of % return vs prev-close thumbnails, shared y-range, segment-colored `LineCollection` |
| `scatter_pane.py` | `ScatterPane` — SOD vs PnL/Return, hover + log-X toggle |
| `treemap_pane.py` | `TreemapPane` — \|PnL\|-sized tiles, sign-colored, 150 ms debounced resize |
| `tooltip.py` | `Tooltip` — singleton yellow floater shared by every pane |

## GUI Layout

### Control strip (row 0)
Col 0 **ETF % from Highs** (SPY/QQQ/IWM/EEM) · Col 1 **Custom % from Highs** (up to 4 user tickers with × delete + Add row that only appears when count < 4; shows extra **Today** column) · Col 2 buttons + status + PnL summary · Col 3 **Intraday** thumbnails (SPY/QQQ/SMH, 1-min `period="1d"`).

Buttons: **Run** · **Auto Update** (60 s loop; label becomes `Stop (Ns)` with live countdown) · **Log X** · **Group Tickers** · **Return %** · **Sort A–Z** · **Export CSV**. PnL summary shows UBS / 401K / Total, green/red by sign.

Intraday y-axis is **% return vs prev close** with a shared y-range across all three charts. Each chart is a `LineCollection` whose segments are colored by sign of midpoint; dotted gray 0% line; title color tracks sign of latest value.

ETF and Custom tables share a single `HighCache` (highs cached per calendar day; `bump`ed up if `last_price` overshoots). Custom pane's height is pinned to ETF's via `grid_propagate(False)` so they stay aligned.

### Keyboard
`F5` Run · `Ctrl+A` toggle Auto Update · `Ctrl+S` Export CSV

### Panes (row 1, `ttk.PanedWindow`, horizontal)
Order and initial weights: **treemap (7) | scatter (9) | ticker bar (6) | tag bar (7)**.

- **Treemap** — tiles sized by `abs(PnL)`, flat green/red by sign of `% move`; label hidden when `min(dx,dy) ≤ fig_w * 0.06`; y-axis inverted (squarify origin is upper-left).
- **Scatter** — X=SOD, Y=PnL or Return; UBS blue, 401K orange, multi-source purple (`#9467bd`).
- **Ticker bar** — `ScrollableBarChart`, row 0.28 in, min 3.0 in, resize debounced 50 ms.
- **Tag bar** — same widget fed by `build_tag_bar_df`; always grouped by `Tag`; ignores Group Tickers and Return % toggles.

### Toggle matrix
| Toggle | Scatter | Treemap | Ticker bar | Tag bar |
|---|---|---|---|---|
| Log X | x-scale | — | — | — |
| Group Tickers | grouped dots | grouped tiles | grouped bars | always grouped |
| Return % | Y axis | — | X axis | — |
| Sort A–Z | — | — | order | order |

## Key Contracts
- `load_and_compute(status_cb=None) -> DataFrame` — fills price/PnL columns; `status_cb(stage)` drives the status bar.
- `get_price_data(ticker) -> (last_price, last_close, pct_move)` — `None`s on failure; transparently split-adjusts `last_close`.
- `get_drawdowns(tickers) / get_default_drawdowns() -> {ticker: {"Today": dec|None, "6W": dec|None, "ATH": dec|None}}`.
- `get_intraday_prices() -> {ticker: {"hist": DataFrame|None, "prev_close": float|None}}`.
- `build_bar_df(df, sort_by_name, grouped, return_mode)` → always has `Label`, `Source`, `Value`. Shared by `draw_bar`.
- `build_tag_bar_df(df, sort_by_name)` → `Label`, `Tag`, `Value` (no `return_mode`).
- `build_grouped_scatter_df(df)` → one row per `Ticker Alias` with summed `PnL`, summed `SOD VALUE`, comma-joined `Sources`, and `Return`.
- `draw_scatter(...)` returns the plotted DataFrame; `ScatterPane.scatter_df` holds it for hover + CSV export.
- `draw_treemap(...)` returns `(rects, plot_data)`; `rects` are squarify dicts (`x, y, dx, dy`) used for hit-testing.
- `RunLoop(...).run_once() / .toggle_auto()` — calls `on_result(RunResult(plot_df, etf_dd, custom_dd, intraday))` on the main thread.

## Gotchas
- `PCT_MOVE` is a **decimal**, not %×100. `PnL = SOD VALUE * PCT_MOVE`.
- Every tkinter call from a worker thread must go through `root.after(0, ...)` — `RunLoop._worker`, `DrawdownTable._add_worker`, `PnLApp._refresh_custom_drawdowns` all rely on this.
- `ScrollableBarChart`: `_canvas_widget.config(width=w, height=h_px)` triggers matplotlib's `<Configure>` (recreates `_tkphoto`). Do **not** unbind it. Always call `_canvas.draw()` explicitly — needed even when size is unchanged (e.g. sort toggle).
- Bar/tag bar hover uses `round(event.ydata)` for hit detection — `bar.contains(event)` is unreliable on Windows with DPI scaling.
- Treemap hover iterates the rects from `draw_treemap`; matching rows live on `TreemapPane._df`.
- `RunLoop.toggle_auto` must cancel **both** `_auto_after_id` (next run) and `_countdown_id` (per-second tick).
- `HighCache.get_or_fetch` resolves `_fetch_highs` at call time (not as a default arg) so `patch.object(data, "_fetch_highs", ...)` works in tests.
- Auto-detected splits silently rewrite `last_close` inside `get_price_data` — check the log for the INFO line before assuming a bug.

## Tests
`pytest tests/ -q`. Coverage:
- `tests/test_data.py` — `_detect_split_ratio`, `HighCache`, `validate_ticker`, `_fetch_drawdown` (yfinance mocked).
- `tests/test_charts.py` — `build_bar_df`, `build_grouped_scatter_df`, `build_tag_bar_df` against `sample_positions` in `conftest.py`.

## Git
- Always include `.claude/` in commits.
- Remote: https://github.com/marange63/PnL-Monitor.git

---
> Source: [marange63/PnL-Monitor](https://github.com/marange63/PnL-Monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
