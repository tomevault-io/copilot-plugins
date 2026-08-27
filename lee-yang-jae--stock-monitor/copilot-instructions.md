## stock-monitor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

US stock real-time monitoring, backtesting, and portfolio analysis application built with Python/Tkinter. Uses Yahoo Finance (yfinance) for market data. UI and comments are in Korean.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Run the application
python stock_monitor_gui.py

# Build Windows executable
pyinstaller stock_monitor_gui.spec
```

There are no tests or linting configured.

## Architecture

```
stock_monitor_gui.py              # Main Tkinter GUI, entry point (AppState singleton)
  └── modules/                    # All supporting modules (added to sys.path at startup)
      ├── __init__.py
      ├── stock_score.py          # Technical analysis (RSI, MA, MACD, Bollinger, momentum, Ichimoku)
      ├── market_trend_manager.py # US market session detection, trend caching, volatility regime
      ├── config.py               # JSON config loading with recursive default merging, atomic writes
      ├── backtest_popup.py       # Strategy backtesting engine + matplotlib charts (8 strategies)
      ├── news_panel.py           # Finviz news scraping, sentiment classification, ticker linking
      ├── data_cache.py           # SQLite cache for yfinance data (delta updates, TTL-based expiry)
      ├── pattern_recognition.py  # Chart pattern detection (double top/bottom, H&S, triangles) via scipy
      ├── fundamental_score.py    # Valuation scoring, Piotroski F-Score, factor scoring
      ├── portfolio_analysis.py   # Correlation, optimization (4 methods + efficient frontier), Black-Litterman, Fama-French
      ├── portfolio_backtest.py   # Portfolio-level backtest: monthly momentum top-N rebalancing vs SPY (분석 menu)
      ├── holdings_manager.py     # Portfolio holdings CRUD (holdings.json), position/P&L calculation
      ├── quant_screener.py       # Quantitative screening (7 strategies: buffett/graham/lynch/greenblatt/dividend/momentum/multifactor)
      ├── screener_popup.py       # Screener UI popup with Treeview results + detail panel
      ├── stock_universe.py       # Stock universe providers (S&P500/NASDAQ100/DOW30 bundled + online + CSV) + search_symbols() (ticker/company-name lookup via yf.Search with HTTP fallback)
      ├── help_texts.py           # Centralized Korean help/tooltip strings
      ├── ui_components.py        # Reusable Tooltip / HelpTooltip widgets (theme-aware)
      ├── theme.py                # Dark/light theme system (ttk 'clam' styles, tk option_add defaults, matplotlib rcParams)
      ├── price_alerts.py         # Price alert system (alerts.json, threshold checks, toast notifications, manager dialog)
      ├── korean_aliases.py       # Korean stock-name alias dictionary (korean_aliases.json, seeded from built-in defaults; manager dialog; powers Korean search + table name suffix)
      ├── stock_memos.py          # Per-ticker memos (stock_memos.json, multiline editor dialog; 📝 marker in table)
      └── file_io.py              # Shared atomic JSON write (tempfile+fsync+os.replace), corrupt-file backup, and backup_files() (move files to a backup dir); used by all JSON persistence paths + the 데이터 초기화 feature
```

`stock_monitor_gui.py` adds `modules/` to `sys.path` at startup, so all inter-module imports (`import config`, `from fundamental_score import ...`) work unchanged.

### Data Flow

GUI spawns a daemon thread (`monitor_stocks`) that refreshes every 60 seconds. Each refresh uses `ThreadPoolExecutor(max_workers=10)` to call `fetch_stock_data()` in parallel for all tickers in `watchlist.json`. Rows update incrementally as each ticker completes (`upsert_record_row()` via `root.after`); `app._row_by_ticker` maps tickers to Treeview rows and `app.all_records` keeps the latest record per ticker (used by the toolbar search filter). On startup, the last refresh results are restored instantly from `table_snapshot.json` before fresh data arrives. `refresh_table_once()` is guarded against re-entry by `app._refresh_guard`.

`fetch_stock_data()` minimizes slow yfinance calls: live quotes come from `fast_info` and fundamentals from the 24h SQLite `fundamental_cache` (`_get_quote_bundle()`); the full `.info` scrape only runs during pre/after-market sessions (pre/post prices exist only there). Earnings dates are cached 12h in `misc_cache`; higher-timeframe trend is cached in-memory for 10 minutes.

### Backtesting

Double-clicking a ticker row (or clicking a ticker in the news panel) opens `backtest_popup.py`, which runs one of 8 strategies (macd, rsi, bollinger, ma_cross, macd+rsi, momentum_signal, momentum_return_ma, ichimoku) against historical data and renders matplotlib charts with buy/sell markers. Includes strategy comparison and sensitivity analysis embedded in the result container.

### Configuration

`config.py` loads `config.json` with recursive merging against defaults. Three presets (short/middle/long) control period, interval, and indicator parameters. Access via `config.config` proxy (lazy-loaded, thread-safe). `get_risk_free_rate()` fetches ^TNX with 1hr cache, 4.5% fallback.

### Data Caching

`data_cache.py` provides SQLite-based caching for yfinance downloads with TTL-based expiry and delta updates. Also has `fundamental_cache` table (24hr TTL, used by the main fetch path and the quant screener) and a generic `misc_cache` key-value table (`get_misc`/`store_misc`, e.g. earnings dates). Integrated into `fetch_stock_data()` and `_retry_download()`. All stored indexes are normalized to UTC (`_to_utc_index`) and loaded with `pd.to_datetime(format="mixed")`; `_store_data` deletes existing rows before inserting to keep one canonical row set per (ticker, interval).

### Portfolio Analysis

`portfolio_analysis.py` provides correlation matrix, 4 optimization methods (max Sharpe, min variance, risk parity, equal weight) with an efficient-frontier chart (CML included), Black-Litterman model, and Fama-French factor analysis. The 포트폴리오 평가 popup includes VaR/CVaR, stress tests, risk contribution, and an underwater (drawdown) chart. `portfolio_backtest.py` simulates monthly momentum top-N rebalancing (dual momentum optional) across the whole watchlist vs SPY buy&hold. The 분석 menu also has an earnings calendar (실적발표 일정) built from the `misc_cache` earnings dates. All accessible from the 분석 menu.

Screener filter presets (strategy/universe/filters/weights combos) are saved under `config.json` `screener.filter_presets` via the 프리셋 row in `screener_popup.py`.

## Threading Model

- **Main thread**: Tkinter event loop only. All UI updates must go through `root.after()` or `popup.after()`.
- **Monitor thread**: Daemon thread with 60s refresh loop using `ThreadPoolExecutor(max_workers=10)` for parallel ticker fetching.
- **News thread**: Daemon thread for 5-minute news refresh cycle.
- **Backtest/analysis**: Run in background threads with UI cleanup via `popup.after(0, callback)`.
- **Thread safety**: `app.watchlist_lock` protects watchlist access. `app.news_lock` protects cached news. Config uses lazy init with `_config_lock`. `data_cache.py` uses its own `_lock` for SQLite access.

## Theming

`modules/theme.py` owns all colors and fonts (dark default, light optional; `config.json` key `ui_theme`). `theme.apply_theme(root)` must be called **before** building widgets (it sets ttk styles and `option_add` defaults for classic tk widgets). Other modules read colors via `theme.get_palette()` / `theme.color(key)` — do not hardcode hex colors. `theme.apply_matplotlib_theme()` sets chart rcParams. Table signal tag colors live in the palette (`tag_buy_bg` etc.) and are applied by `_apply_table_tag_colors()` in the main GUI. The toolbar ◐ button toggles themes at runtime (ttk restyles immediately; long-lived classic tk widgets are re-colored by `_retheme_classic_widgets()`; popups need a restart).

Gotchas: tkcalendar `DateEntry` must receive `**theme.dateentry_kwargs()` or its dropdown calendar renders light. Windows title bars are darkened via DWM (`theme.set_dark_title_bar`); all `tk.Toplevel`s get it automatically. The native Windows menubar cannot be themed (known limitation). Heatmap-style cell text (correlation matrix, sensitivity tables) intentionally stays `"white"/"black"` relative to the cell color — don't convert those to palette colors. Scrollbar thumb (`scrollbar` palette key) must keep ~3:1 contrast against the trough.

## Key Data Files

- `config.json` — user settings (auto-created on first run, auto-saved on changes; includes `ui_theme`, `view_columns_mode`)
- `watchlist.json` — monitored ticker list (auto-created, modified via GUI)
- `holdings.json` — portfolio holdings with transactions
- `alerts.json` — price alerts + trigger history (managed via `modules/price_alerts.py`; `{"alerts":[...], "history":[...]}`, auto-migrates from legacy list). Alert types: `price` (target $ cross) and `pct` (vs previous-close % move, using `StockData.rate`). One-shot (disabled after triggering, reactivatable); history persists across reactivation, capped at 200. The history also collects automatic events: `signal` (momentum signal transitions, e.g. 관망→매수) and `w52_high`/`w52_low` (52-week breakouts) — detected in `update_table()` via `_detect_quant_events()` (state in `app._last_signals`/`app._last_week52`, seeded silently from snapshots), toggled by config keys `signal_change_alert`/`week52_alert` under master `alert_enabled`.
- `korean_aliases.json` — Korean name→ticker alias dictionary (auto-seeded from `korean_aliases._DEFAULT_ALIASES` on first run; user add/remove via 종목 > 한글 별칭 관리). Used by symbol search, table filter, alert resolver, and the " · 한글명" suffix on 종목명.
- `stock_memos.json` — per-ticker memos (`{ticker: {text, updated}}`, edited via context menu 메모 편집; 📝 marker in 종목명)
- `table_snapshot.json` — last refresh results, rendered instantly at startup (discarded if `StockData` fields change)
- `modules/stock_data_cache.db` — SQLite cache for yfinance data (price, fundamentals, misc key-value)
- `logs/app.log` — rotating log (5MB/file, 5 backups, 30-day retention); third-party DEBUG logs (yfinance/peewee) are suppressed

All user-data JSON files (and `*.db`, `logs/`, `data_backup_*/`) are gitignored — auto-recreated on first run. The 설정 popup has a **데이터 초기화** button (`reset_all_data()` in `stock_monitor_gui.py`) that backs up existing data files to `data_backup_<timestamp>/` (via `file_io.backup_files`), then re-seeds user data to its initial format (watchlist→SPY/QQQ, holdings/alerts/memos→empty, aliases→built-in defaults, custom universe/snapshot→deleted) while keeping `config.json` and the cache DB — applied live, no restart. Per-module reset helpers: `price_alerts.reset_all`, `stock_memos.reset_all`, `korean_aliases.reset_to_defaults`, `stock_universe.clear_custom_universe`, `holdings_manager.save_holdings({})`.

## Important Constraints

- yfinance `interval`/`period` combinations have strict limits (e.g., 1m interval max 7 days, 5m max 60 days). See `auto_set_interval_by_period()` in `stock_score.py`.
- All file I/O uses UTF-8 encoding explicitly.
- The PyInstaller spec includes hidden imports for all modules — update `stock_monitor_gui.spec` when adding new dependencies.
- Backtest popup passes stock as `"CompanyName (TICKER)"` format string; `ticker_symbol` is extracted via `stock.split('(')[-1].split(')')[0]`.
- Matplotlib figures must be tracked in `open_figures[]` and closed on popup destroy to prevent memory leaks.
- `StockData` namedtuple fields must match table insert order in `_build_row_payload()` (used by `update_table()`/`upsert_record_row()`). Changing fields also invalidates `table_snapshot.json` (by design).
- Commission/slippage config: `backtest.commission_rate`, `backtest.slippage_pct` in config.json.
- Quant screener strategies reuse `fundamental_score.py` functions (`valuation_score`, `factor_score`, `piotroski_fscore`).
- The 퀀트 점수 table column shows `calculate_factor_score()` (V+M+Q, 0~9 + grade) computed per ticker in `fetch_stock_data()` (`StockData.quant_score`/`quant_grade`).

---
> Source: [LEE-YANG-JAE/stock_monitor](https://github.com/LEE-YANG-JAE/stock_monitor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
