## ccstockworkenv

> CCStockWorkEnv is a Claude Code environment for multi-market stock research & financial analysis, accessed via Claude Telegram Bot (ctb). Supports US, Taiwan, and China markets.

# CCStockWorkEnv — 股票研究分析環境

## 專案概述

CCStockWorkEnv is a Claude Code environment for multi-market stock research & financial analysis, accessed via Claude Telegram Bot (ctb). Supports US, Taiwan, and China markets.

**Launch:** `ctb /Users/wanghsuanchung/Projects/CCStockWorkEnv`

## 核心原則

1. **Fail-fast** — No overprotective try-except. Let errors propagate with clear messages.
2. **Direct dict access** — Use `dict["key"]` by default. Only `.get()` for truly optional fields.
3. **No workarounds** — Investigate root causes, don't mask bugs with fallbacks.
4. **No over-engineering** — Only build what's needed now.

## 回應方式 — 優先產生網頁報告

When the user's request involves **multi-stock analysis, financial data, company research, industry overview, or any response that would exceed ~10 lines of text**, Claude MUST generate an HTML report on the web server instead of sending a long Telegram message.

### Rules

1. **Always reply in the original chat** — Respond directly in the conversation. The ctb bot delivers your reply to the original Telegram chat/group automatically. Do NOT call `send_message.py` or `send_mail.py` unless the user explicitly asks to send a message/email, or in scheduled tasks that run without a conversation. Never use email or Telegram send tools as a substitute for replying.
2. **Web report over Telegram text** — If the answer contains tables, financial data, multiple stocks, or detailed analysis, always produce an HTML report in `output/` and reply with the web server URL. Never dump walls of text into Telegram.
3. **Embed interactive charts** — For any stock mentioned in the report, embed a K-line chart via iframe. **MUST use relative path** (starts with `/`), NEVER use `http://localhost:8800` or any absolute URL — mobile users cannot reach localhost.
   ```html
   ✅ <iframe src="/charts/TICKER/?market=XX&period=1y&embed=1" ...>
   ❌ <iframe src="http://localhost:8800/charts/TICKER/..." ...>
   ❌ <iframe src="http://<EXTERNAL_IP>/charts/TICKER/..." ...>
   ```
   Full tag: `<iframe src="/charts/TICKER/?market=XX&period=1y&embed=1" style="width:100%;height:420px;border:1px solid #e0e0e0;border-radius:6px;" loading="lazy"></iframe>`
4. **Provide evidence** — Back up claims with data. Include: stock price charts (embedded iframes), key financial ratios in tables, data source attribution, and relevant metrics (P/E, P/B, ROE, etc.).
5. **Telegram reply should be short** — Only reply with a brief summary (2-3 sentences) + the report URL.
   **Report URL construction:**
   - Read `config.json` → `web_server.fixed_ip` and `web_server.external_port`
   - If `fixed_ip` is set (non-empty, not null): use `http://<fixed_ip>:<external_port>/reports/SLUG/`
   - If `fixed_ip` is missing/empty: fallback to `http://localhost:<internal_port>/reports/SLUG/` and **warn the user** that the URL is only accessible on the local network, not from mobile devices outside the LAN
   - Example: `📊 騰訊 vs 阿里巴巴個股分析報告已產生，包含財務數據、K線圖及關鍵指標。\n🔗 http://<FIXED_IP>:<EXTERNAL_PORT>/reports/SLUG/`
6. **Short answers stay in Telegram** — Simple queries like "台積電現在股價多少?" can be answered directly in Telegram without generating a report.

### Examples of when to generate a report

- "幫我查詢騰訊和阿里巴巴的個股資訊及財務數據" → HTML report with company info tables, financial data, embedded K-line charts
- "找美國上市的光通訊公司，檢查去年財務狀況" → HTML report with screened companies, financial health analysis, charts
- "分析半導體產業趨勢" → HTML report with industry overview, key players, charts
- "比較 AAPL 和 MSFT" → HTML report with side-by-side comparison, charts

### Examples of when Telegram text is OK

- "台積電現在多少錢?" → Short price quote in Telegram
- "MU 的 P/E 是多少?" → One-liner answer in Telegram

## 語言慣例

- **Code & docs**: English
- **Claude 回應**: 繁體中文 (Traditional Chinese) — Claude MUST respond in Traditional Chinese at all times
- **User-facing output**: 繁體中文 (Traditional Chinese)
- **Commands 描述**: 繁體中文 (shown in Telegram)

## 專案結構

```
CCStockWorkEnv/
├── CLAUDE.md                    # This file
├── .claude/commands/            # CTB slash commands (14)
├── .claude/skills/              # Domain knowledge (8)
├── .claude/agents/              # Specialized agents (2)
├── tool_scripts/
│   ├── send_telegram/           # Telegram messaging
│   ├── send_mail/               # Email via Mailgun
│   ├── market_data/             # Market data API abstraction
│   ├── financial_calc/          # Z-Score, F-Score, screener
│   ├── db_ops/                  # SQLite operations
│   ├── report_gen/              # Report & chart generation
│   └── web_server/              # Django report viewer (RWD)
├── schedules/                   # Scheduled task scripts
├── data/                        # SQLite DB, exports, charts, logs (gitignored)
├── output/                      # Timestamped reports (gitignored)
└── prompts/                     # YYYYMMDD_N_description.md
```

## Python 執行模式

All tool_scripts run via uv:
```bash
cd tool_scripts/<subfolder> && uv run python <script>.py [args]
```

## prompts/ 命名

Format: `YYYYMMDD_N_description.md` where N is sequential for the day.

## 資料庫

SQLite at `data/ccstockworkenv.db`. WAL mode enabled. Schema managed by `db_ops/db_manager.py`.

## API 抽象層

```
fetcher_base.py  →  MarketDataFetcher (ABC)
fetcher_us.py    →  USFetcher    [yfinance]
fetcher_tw.py    →  TWFetcher    [twstock/FinMind]
fetcher_cn.py    →  CNFetcher    [AKShare]
fetcher_factory.py → get_fetcher(market) → MarketDataFetcher
```

Market codes: `US`, `TW`, `CN`

## 財務指標

| Metric | Module | Purpose |
|--------|--------|---------|
| Altman Z-Score | `zscore.py` | Bankruptcy risk (>2.99 safe, <1.81 distress) |
| Piotroski F-Score | `fscore.py` | Financial strength (0-9, ≥7 strong) |
| Opportunity Score | `opportunity_score.py` | Weighted composite score |
| Key Ratios | `ratios.py` | P/E, P/B, ROE, ROA, D/E, margins |

## 財務分析規範 — 「財務狀況」的標準

When the user asks about a company's 財務狀況 (financial status/health), Claude MUST perform **deep financial research** — not just surface-level revenue or earnings. Use `fetcher_factory.py` to pull actual financial data (`financials`, `metrics`, `quote`) and present them in structured tables.

### Required analysis dimensions

**1. 盈利能力 (Profitability)**

| Metric | 中文 | How to interpret |
|--------|------|------------------|
| Gross Margin | 毛利率 | >40% excellent, <20% weak |
| Operating Margin | 營業利潤率 | Industry-dependent |
| Net Margin | 淨利率 | Higher = stronger pricing power |
| ROE | 股東權益報酬率 | >15% strong, <5% weak |
| ROA | 資產報酬率 | >5% good for capital-intensive |
| EPS & EPS Growth | 每股盈餘及年增率 | Trend matters more than absolute |

**2. 財務健康 (Solvency & Liquidity)**

| Metric | 中文 | How to interpret |
|--------|------|------------------|
| Debt/Equity | 負債比率 | <1.0 conservative, >2.0 high leverage |
| Current Ratio | 流動比率 | >1.5 safe, <1.0 liquidity risk |
| Quick Ratio | 速動比率 | >1.0 safe |
| Interest Coverage | 利息保障倍數 | >3x safe, <1.5x danger |
| Altman Z-Score | Z分數 | >2.99 safe, <1.81 distress |

**3. 現金流 (Cash Flow) — 最重要的維度**

| Metric | 中文 | How to interpret |
|--------|------|------------------|
| Operating Cash Flow | 營業現金流 | Must be positive and growing |
| Free Cash Flow | 自由現金流 | OCF minus CapEx, must be positive |
| OCF / Net Income | 現金流/淨利 | >1.0 = high quality earnings |
| CapEx | 資本支出 | Context-dependent (growth vs maintenance) |

**4. 成長性 (Growth)**

| Metric | 中文 | How to interpret |
|--------|------|------------------|
| Revenue Growth YoY | 營收年增率 | Trend over 3-5 years |
| Net Income Growth YoY | 淨利年增率 | Should track or exceed revenue growth |
| Book Value Growth | 每股淨值成長 | Steady growth = compounding |

**5. 估值 (Valuation)**

| Metric | 中文 | How to interpret |
|--------|------|------------------|
| P/E | 本益比 | Compare to industry peers |
| P/B | 股價淨值比 | <1.0 may be undervalued |
| P/S | 股價營收比 | Useful for unprofitable companies |
| EV/EBITDA | 企業價值倍數 | <10 may be cheap |
| Dividend Yield | 股息殖利率 | >3% attractive for income |

**6. 財務品質 (Quality Scores)**

| Metric | 中文 | How to interpret |
|--------|------|------------------|
| Piotroski F-Score | F分數 | ≥7 strong, ≤3 weak |
| Receivable Turnover Days | 應收帳款周轉天數 | Lower = faster collection |
| Inventory Turnover Days | 存貨周轉天數 | Lower = better efficiency |

### Report format for financial analysis

Each company in the report MUST include:
1. **Company overview table** — name, ticker, market, sector, market cap
2. **Key metrics summary** — a single table with all ratios above (color-coded: green=good, yellow=watch, red=danger)
3. **Financial statements extract** — revenue, net income, total assets, total liabilities, OCF for the last 3-5 periods in a trend table
4. **K-line chart iframe** — embedded interactive chart
5. **Assessment** — brief text interpretation of the numbers

### Data sources

Use `fetcher_factory.py` CLI to get real data:
```bash
# Financial statements (income, balance sheet, cash flow)
cd tool_scripts/market_data && uv run python fetcher_factory.py financials TICKER --market XX --period annual

# Key metrics (P/E, P/B, ROE, margins, etc.)
cd tool_scripts/market_data && uv run python fetcher_factory.py metrics TICKER --market XX

# Current quote (price, market cap, volume)
cd tool_scripts/market_data && uv run python fetcher_factory.py quote TICKER --market XX
```

## 工具路徑表

| Tool | Path | Usage |
|------|------|-------|
| Send Telegram | `tool_scripts/send_telegram/send_message.py` | `--message "text"` / `--send-file path` |
| Send Email | `tool_scripts/send_mail/send_mail.py` | `--subject "s" --body "b"` |
| DB Manager | `tool_scripts/db_ops/db_manager.py` | `--init` / `--migrate` |
| Stock Ops | `tool_scripts/db_ops/stock_ops.py` | CRUD for stock universe |
| Price Ops | `tool_scripts/db_ops/price_ops.py` | Daily price CRUD + bulk upsert |
| Financial Ops | `tool_scripts/db_ops/financial_ops.py` | Financial + health scores CRUD |
| Research Cache | `tool_scripts/db_ops/research_cache_ops.py` | Cache freshness check + mark |
| Screening Ops | `tool_scripts/db_ops/screening_ops.py` | Screening results CRUD |
| Watchlist Ops | `tool_scripts/db_ops/watchlist_ops.py` | Watchlist & notes CRUD |
| Fetcher Factory | `tool_scripts/market_data/fetcher_factory.py` | `get_fetcher("US")` |
| Z-Score | `tool_scripts/financial_calc/zscore.py` | `calculate_zscore(data)` |
| F-Score | `tool_scripts/financial_calc/fscore.py` | `calculate_fscore(data)` |
| Ratios | `tool_scripts/financial_calc/ratios.py` | `calculate_ratios(data)` |
| Screener | `tool_scripts/financial_calc/screener.py` | `screen(criteria, market)` |
| Report Gen | `tool_scripts/report_gen/markdown_report.py` | Generate markdown reports |
| Chart Gen | `tool_scripts/report_gen/chart_gen.py` | matplotlib charts |
| Web Server | `tool_scripts/web_server/` | Django report viewer (port 8800) |

## 報告產生規範

All reports MUST follow `.claude/skills/report_generation_guide.md`. Key requirements:

1. **Mobile-first HTML** — viewport meta, RWD CSS, tables in `.table-container`
2. **Publish via URL** — send web server link to Telegram, not raw HTML files
3. **Web server** — Django at `http://localhost:8800`, auto-reads from `output/`
4. **Report naming — timestamp FIRST** — Both files and directories MUST use `YYYYMMDD_HHMMSS_<type>` format. The Django scanner regex requires `^\d{8}_\d{4,6}_` at the start. Reversed naming like `<type>_YYYYMMDD` will silently 404.
   ```
   ✅ output/20260301_104212_cn_hbm_companies/index.html
   ❌ output/cn_hbm_companies_20260301_104212/index.html
   ```
5. **Pre-publish verification (MANDATORY)** — NEVER send a report URL to the user before verification passes. Follow this exact sequence:
   ```bash
   # Step A: Ensure web server is running
   curl -s -o /dev/null -w "%{http_code}" http://localhost:8800 | grep -q 200 || \
     (cd tool_scripts/web_server && bash start_server.sh)

   # Step B: Verify the report URL returns HTTP 200 (NOT 404)
   curl -s -o /dev/null -w "%{http_code}" http://localhost:8800/reports/<SLUG>/
   # If 404 → naming is wrong. Fix the directory name before proceeding.

   # Step C: Browser verification via Playwright (MANDATORY)
   # Mobile screenshot (375x812)
   npx playwright screenshot --browser chromium --viewport-size "375,812" --full-page --wait-for-timeout 3000 \
     "http://localhost:8800/reports/<SLUG>/" /tmp/report_mobile.png
   # Desktop screenshot (1280x800)
   npx playwright screenshot --browser chromium --viewport-size "1280,800" --full-page --wait-for-timeout 3000 \
     "http://localhost:8800/reports/<SLUG>/" /tmp/report_desktop.png
   # Read both screenshots to verify:
   #   - Title displays fully, no truncation
   #   - Tables are inside .table-container (horizontally scrollable)
   #   - Cards and badges render correctly
   #   - Text is readable (>= 13px)
   #   - No horizontal overflow on body
   #   - Color coding correct (green/orange/red)

   # Step D: Only after Step B AND Step C pass, reply with URL
   ```
   If Step B returns 404, DO NOT send the URL. Debug the naming issue first.
   If Step C screenshots show layout issues, fix the HTML before sending.

## 研究快取 — Cache-First Workflow

When performing financial research, Claude MUST use the cache-first pattern to avoid redundant API calls. Fetched data is stored in the DB; subsequent requests read from cache.

### Freshness policy

| data_type | Max age | Rationale |
|-----------|---------|-----------|
| `financials` | 90 days | Quarterly filing cycle |
| `metrics` | 24 hours | Price-dependent ratios |
| `company_info` | 180 days | Rarely changes |

`health_scores` has no independent freshness — it is recomputed whenever financials are updated.

### Trigger conditions

Any prompt involving 財務狀況 / 財報 / 健康 / 比較 / 篩選 triggers the cache-first workflow. Simple price queries do NOT trigger storage.

### Workflow — single stock financial analysis

```bash
# 1. Check cache freshness
cd tool_scripts/db_ops && uv run python research_cache_ops.py --is-fresh AAPL US financials

# 2. If stale/missing → fetch + save + compute
cd tool_scripts/market_data && uv run python fetcher_factory.py financials AAPL --market US --period annual
cd tool_scripts/db_ops && uv run python financial_ops.py --bulk-upsert --json '<json_data>'
cd tool_scripts/db_ops && uv run python financial_ops.py --compute-health AAPL --market US
cd tool_scripts/db_ops && uv run python research_cache_ops.py --mark AAPL US financials

# 3. If fresh → read directly from DB
cd tool_scripts/db_ops && uv run python financial_ops.py --get AAPL --market US --period annual
cd tool_scripts/db_ops && uv run python financial_ops.py --get-health AAPL --market US

# 4. Also check metrics cache (24h freshness)
cd tool_scripts/db_ops && uv run python research_cache_ops.py --is-fresh AAPL US metrics
# If stale: fetch metrics → mark cache with --data '<json>'
```

### Multi-stock research

For multi-stock requests, run the single-stock workflow for each ticker. Companies with fresh cache skip API calls entirely.

### Data flow

```
API fetch (financials) → financials table → compute → health_scores table
API fetch (metrics)    → research_cache.data_json
```

## 排程任務

When users ask for recurring/scheduled tasks, follow `.claude/skills/scheduler_sop.md`. Key points:

1. **Script** → `schedules/<task_name>.sh` (shell script with Telegram notifications)
2. **Plist** → `~/Library/LaunchAgents/com.ccstockworkenv.<task_name>.plist` (launchd scheduler)
3. **Logs** → `data/logs/<task_name>.log`
4. **launchctl limitation** — `launchctl load` cannot run from Claude Code. Create the files, then tell the user to run `launchctl load` in Terminal.app.
5. **Always notify** — Both success and failure must send Telegram messages.

## 免責聲明

All analysis is for research and educational purposes only. Not investment advice. Always do your own due diligence before making investment decisions.

---
> Source: [GoatWang/CCStockWorkEnv](https://github.com/GoatWang/CCStockWorkEnv) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
