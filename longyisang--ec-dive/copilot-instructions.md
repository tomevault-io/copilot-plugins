## ec-dive

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Project Overview

ec-dive is a cross-border e-commerce market intelligence platform. It scrapes Amazon Best Sellers rankings and 1688 supplier prices, computes profit margins, and visualizes results via Metabase dashboards. The project uses a **dual-track architecture**: 影刀 RPA as the primary data collector and Python Playwright as an exploratory alternative.

Design spec: `docs/superpowers/specs/2026-04-07-ec-dive-design.md`

## Commands

```bash
# Start infrastructure (MySQL + Metabase)
cd docker && docker compose up -d

# Install Python dependencies
pip install -r requirements.txt

# Install Playwright browser (one-time)
playwright install chromium

# Run individual scrapers
python -m src.scrapers.exchange_rate
python -m src.scrapers.amazon_bsr

# Run profit margin calculation (requires exchange_rate + amazon + 1688 data)
python -m src.processor.margin

# Run alert detection (requires margin data + at least 2 price snapshots)
python -m src.alerter.detector

# Run full Python pipeline (exchange_rate → margin → alerter → bitable → bot)
python run_all.py

# Verify DB data
python -c "from src.db import get_connection; conn = get_connection(); print('DB OK'); conn.close()"
```

## Architecture

**Dual-track collection** → Python processing → MySQL storage → Metabase visualization

- `src/config.py` — All config via `os.getenv()` with defaults. Add new config here.
- `src/db.py` — Single `get_connection()` returning pymysql DictCursor. All modules share this.
- `src/scrapers/` — Data collection modules. Each follows the pattern: `fetch_*()` → `save_*()` → `run()` entry point with `if __name__` block.
- `src/processor/margin.py` — Margin calculation: joins latest Amazon price + 1688 supplier price + exchange rate, computes `estimated_margin`, writes back to `products`.
- `src/alerter/` — Anomaly detection, writes to `alerts` table
- `src/feishu/` — Feishu (Lark) Open API integration: bitable writes + bot notifications
- `src/placeholders/` — Stub modules for LLM analysis and pytrends (to be implemented)
- `docker/init.sql` — MySQL schema (6 tables: products, prices, reviews, trends, alerts, task_runs)
- `rpa/` — 影刀 RPA flow exports and flow design docs (not managed by Python)

## Key Patterns

- **Scraper pattern**: Every scraper in `src/scrapers/` has `run()` as entry point and logs execution to `task_runs` table.
- **DB writes**: Use `get_connection()`, `with conn.cursor()`, parameterized `%s` queries, `conn.commit()` in try/finally with `conn.close()`.
- **Upsert**: Products use `INSERT ... ON DUPLICATE KEY UPDATE` on the `asin` UNIQUE key.
- **Price snapshots**: Each scrape inserts new rows (not updates) to `prices` for time-series tracking. Exchange rate baseline records have `product_id=NULL`.
- **CSS selectors**: Amazon BSR selectors are centralized in `SELECTORS` dict at top of `amazon_bsr.py` for easy updating when DOM changes.

## Technical Notes

- **playwright-stealth v2.x**: API is `Stealth().use_sync(context)`, NOT the old `stealth_sync(page)`.
- **Amazon geolocation**: Amazon returns localized content based on IP. Prices may be in non-USD currencies. The scraper extracts currency code alongside price.
- **Windows environment**: Use Unix shell syntax in bash (forward slashes, /dev/null). Chinese log output may show as garbled text in Windows terminal — this is cosmetic only.
- **1688 supplier scraping**: Handled by 影刀 RPA (see `rpa/supplier_1688/flow_design.md`), not Python. Python approach was abandoned due to aggressive anti-scraping.
- **Metabase**: Uses built-in H2 database for its own metadata (not MySQL). Connect to MySQL `ec_dive` database as a data source through the Metabase UI at localhost:3000.

---
> Source: [LongYiSang/ec-dive](https://github.com/LongYiSang/ec-dive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-05 -->
