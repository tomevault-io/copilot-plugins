## finnhub-free-dashboard

> Free-tier data extraction from the **Finnhub** REST API. The shared API key lives

# Finnhub Data Project

Free-tier data extraction from the **Finnhub** REST API. The shared API key lives
in `.env` as `FINNHUB_API_KEY` and is loaded by `python-dotenv` in every script.

- **Base URL**: `https://finnhub.io/api/v1`
- **Auth**: every request carries `?token=<FINNHUB_API_KEY>` as a query param
- **Rate limit**: 60 req/min on free tier
- **Source of truth**: the live Swagger spec at
  `https://finnhub.io/static/swagger.json` (114 documented endpoints)
- **Re-probe**: `python probe_endpoints.py` hits every endpoint and bucketing it
  as `ok / empty / forbidden / other`. Run after any plan change.

## Project layout

```
finhub data/
├── .env                 # FINNHUB_API_KEY=...  (real key, gitignored — see .env.example)
├── finnhub_free.py      # exercises every endpoint the free key can reach
├── probe_endpoints.py   # auto-probe against the live Swagger spec
├── dashboard/           # Streamlit research dashboard (see below)
└── CLAUDE.md            # this file
```

## Dashboard

`dashboard/` is a local Streamlit app over the same 27 free endpoints. Type
any ticker in the sidebar and see a 6-tab research view. Uses `../.env`
for the API key and a SQLite cache at `dashboard/.cache/finnhub.db` to stay
under the 60 req/min free-tier rate limit.

```
cd "C:\Users\thesh\Desktop\finhub data\dashboard"
pip install -r requirements.txt   # adds streamlit
streamlit run app.py
```

| File | Purpose |
|---|---|
| `dashboard/app.py` | Streamlit UI: sidebar (symbol, refresh, asset class) + 6 tabs |
| `dashboard/finnhub_client.py` | Typed per-endpoint functions returning `Optional[...]` (never raise) |
| `dashboard/cache.py` | SQLite TTL cache, key = sha1(path + sorted params) |
| `dashboard/ai_client.py` | OpenRouter-backed AI analysis (button-triggered, cached) |
| `dashboard/.streamlit/config.toml` | Theme (dark) + server config |
| `dashboard/requirements.txt` | `streamlit>=1.30`, `requests>=2.28` |
| `dashboard/README.md` | Quick-start |

The 7 tabs:

1. **Overview** — quote card (price, change, day range, prev close), profile, peers, market status, daily-bar plotly chart with annotation that historical OHLCV is paid-tier only
2. **Financials** — `/stock/metric` table, `/stock/earnings` table, latest 10-K XBRL summary (Net sales, Net income, Total assets, Equity, etc.)
3. **Analyst & Insider** — `/stock/recommendation` stacked bar chart (6 months), top 25 `/stock/insider-transactions`, `/stock/insider-sentiment`
4. **Government / Alt-Data** — `/stock/lobbying`, `/stock/uspto-patent`, `/stock/visa-application`, `/stock/usa-spending`
5. **Filings & News** — `/stock/filings` (form, date, link) + `/news?category=general` (top 20 headlines, clickable)
6. **Reference** — IPO calendar, FDA calendar, market holidays, country list, COVID-19 US, symbol universe, plus crypto/forex reference when those asset classes are selected
7. **AI Analysis** — button-triggered OpenRouter analyses (overall brief, financials & earnings, government/alt-data, insider sentiment). Reads `OPENROUTER_API_KEY` + `OPENROUTER_MODEL` from `.env`; default `meta-llama/llama-3.3-70b-instruct:free` (free-tier — paid models 402 on a $0 account). Free `:free` models 429 unpredictably at peak, so `_chat()` falls back through `[nvidia/nemotron-3-super-120b-a12b:free, openai/gpt-oss-120b:free, qwen/qwen3-next-80b-a3b-instruct:free]` until one responds. Cached 6h, deterministic on (symbol, section, model). Cleared by the "Refresh this symbol" button.

**TTL policy** (per-endpoint, in seconds):

| Family | TTL |
|---|---|
| `/quote` | 60 |
| `/news`, `/stock/market-status` | 300 |
| All other per-symbol | 3600 |
| Reference data (country, COVID, FDA, IPO, exchanges, `/stock/symbol`) | 86400 |

**Use the client outside Streamlit:**

```python
from dashboard.finnhub_client import quote, financials_reported, latest_xbrl_summary
print(quote("AAPL"))                              # {'c': 293.08, ...}
fr = financials_reported("AAPL", freq="annual")    # long-format XBRL DataFrame
print(latest_xbrl_summary(fr))                    # latest 10-K headline figures
```

To wipe the cache, delete `dashboard/.cache/finnhub.db` or hit the
"Refresh this symbol" button in the sidebar.

## Shared helpers

```python
import os, requests
from dotenv import load_dotenv

load_dotenv()                      # picks up .env in the cwd
API_KEY = os.getenv("FINNHUB_API_KEY")
BASE    = "https://finnhub.io/api/v1"

def get(path, **params):
    params["token"] = API_KEY
    r = requests.get(f"{BASE}{path}", params=params, timeout=10)
    r.raise_for_status()
    return r.json()
```

## Free-tier endpoint catalog (27 of 114)

Every endpoint below returned 200 + non-empty data on 2026-06-24 against the key
in `.env`. The remaining 87 endpoints in the Swagger spec are paid-tier and
return HTTP 403 (`You don't have access to this resource`).

### Core stock data

| Method | Path | Required params | Returns |
|---|---|---|---|
| GET | `/quote` | `symbol` | `{c, d, dp, h, l, o, pc, t}` current/daily price |
| GET | `/search` | `q` | `{count, result:[{symbol, description, type, displaySymbol}]}` |
| GET | `/stock/profile2` | `symbol` | `{name, ticker, exchange, currency, country, finnhubIndustry, marketCapitalization, ...}` |
| GET | `/stock/peers` | `symbol` | `["AAPL", "SNDK", "DELL", ...]` |
| GET | `/stock/symbol` | `exchange` | full ticker list for an exchange (e.g. `exchange=US` -> ~30k symbols) |
| GET | `/stock/metric` | `symbol`, `metric` | `{metric:{...133 keys}, series:{...}, metricType}` (use `metric=all`) |
| GET | `/stock/recommendation` | `symbol` | `[{"period","strongBuy","buy","hold","sell","strongSell"}]` monthly analyst consensus |
| GET | `/stock/earnings` | `symbol`, `limit?` | EPS actual vs estimate per quarter, with `% surprise` |
| GET | `/stock/financials-reported` | `symbol`, `freq?` | `{cik, symbol, data:[{accessNumber, year, quarter, form, report:{bs, ic, cf}}]}` XBRL as reported (10-K/10-Q) |
| GET | `/stock/insider-transactions` | `symbol` | `{data:[{name, share, change, transactionDate, transactionPrice, isDerivative}]}` |
| GET | `/stock/insider-sentiment` | `symbol`, `from`, `to` | `{data:[{year, month, change, mspr}]}` (empty for some tickers) |
| GET | `/stock/filings` | `symbol?` | SEC filings index `[{"accessNumber","form","filedDate","reportUrl"}]` |

### Market state

| Method | Path | Required params | Returns |
|---|---|---|---|
| GET | `/stock/market-status` | `exchange` (e.g. `US`) | `{isOpen, holiday, session, t, timezone}` |
| GET | `/stock/market-holiday` | `exchange` | `{data:[{eventName, atDate}]}` upcoming holidays |

### Alternative / government data

All four work freely and return rich data.

| Method | Path | Required params | Returns |
|---|---|---|---|
| GET | `/stock/lobbying` | `symbol`, `from`, `to` (YYYY-MM-DD) | `{data:[{year, period, expenses, documentUrl}]}` Senate LDA filings |
| GET | `/stock/uspto-patent` | `symbol`, `from`, `to` | `{data:[{patentNumber, filingDate, applicationNumber, ...}]}` |
| GET | `/stock/visa-application` | `symbol`, `from`, `to` | `{data:[{year, quarter, sponsor, newApproval, continuingApproval, ...}]}` H-1B |
| GET | `/stock/usa-spending` | `symbol`, `from`, `to` | `{data:[{description, agency, amount, awardDate}]}` federal contracts |

### Calendars & reference

| Method | Path | Required params | Returns |
|---|---|---|---|
| GET | `/calendar/ipo` | `from`, `to` | `{ipoCalendar:[{date, exchange, name, symbol, price, shares}]}` |
| GET | `/fda-advisory-committee-calendar` | — | `[{date, event, ticker, ...}]` |
| GET | `/country` | — | `[{code, currency, exchange, ...}]` ISO country metadata |
| GET | `/covid19/us` | — | state-level US COVID series |

### Crypto reference (no candles on free)

| Method | Path | Required params | Returns |
|---|---|---|---|
| GET | `/crypto/exchange` | — | `["Binance","Coinbase Pro", ...]` |
| GET | `/crypto/symbol` | `exchange` | `[{symbol, description, displaySymbol, currency, ...}]` |

### Forex reference (no candles on free)

| Method | Path | Required params | Returns |
|---|---|---|---|
| GET | `/forex/exchange` | — | `["oanda","forex.com", ...]` |
| GET | `/forex/symbol` | `exchange` | `[{symbol, description, displaySymbol, currency}]` |

### News

| Method | Path | Required params | Returns |
|---|---|---|---|
| GET | `/news` | `category` | `[{id, headline, summary, source, url, image, datetime, related}]`. Valid categories: `general, forex, crypto, merger, ...` |

## What is NOT available on free (HTTP 403)

Major gaps to be aware of - do not waste time calling these:

- **Historical OHLCV**: `/stock/candle`, `/crypto/candle`, `/forex/candle`
- **Earnings transcripts**: `/stock/transcripts`, `/stock/transcripts/list`
- **Full price-history**: `/stock/tick`, `/stock/bbo`, `/stock/bidask`
- **Simplified statements**: `/stock/financials` (use `/stock/financials-reported` instead)
- **Estimates**: `/stock/eps-estimate`, `/stock/revenue-estimate`, `/stock/ebit-estimate`, etc.
- **Analyst targets**: `/stock/price-target`, `/stock/upgrade-downgrade` summary is gated
- **Sentiment**: `/news-sentiment`, `/stock/social-sentiment`, `/stock/filings-sentiment`
- **Reference data**: `/stock/profile` (use `/stock/profile2`), `/stock/split`, `/stock/dividend`
- **Macro/commodities/forex rates**: `/economic`, `/economic/code`, `/forex/rates`
- **Institutional**: `/institutional/*`, `/stock/ownership`, `/stock/fund-ownership`, `/mutual-fund/*`
- **ETF/bond/airline/pattern/scan**: `/etf/*`, `/bond/*`, `/airline/*`, `/scan/*`, `/indicator`
- **AI/filings search**: `/ai-chat`, `/global-filings/*`, `/stock/international-filings`
- **Theme/innovation data**: `/stock/investment-theme`, `/stock/similarity-index`

## Conventions

- All scripts load the key from `.env` via `python-dotenv`. Never hard-code it.
- Use the shared `get(path, **params)` helper - it appends `?token=` automatically.
- Datetime params: `from`/`to` are **epoch seconds** for stock endpoints (insider-sentiment,
  congressional-trading) but **YYYY-MM-DD strings** for the alt-data endpoints
  (lobbying, patents, visa, usa-spending).
- Date keys in responses are mixed: timestamps use epoch seconds (`/quote`, `/news`),
  alt-data uses ISO strings (`YYYY-MM-DD`).
- Tolerate empty arrays: `/stock/insider-sentiment` returns `{data:[]}` for some
  symbols even though the call succeeds.
- Default to `freq=annual` on `/stock/financials-reported` unless you specifically
  want quarterly (set `freq=quarterly`).
- When scraping a long response (e.g. `/stock/symbol?exchange=US` = 30k rows),
  wrap the response in `json.dump` or stream it; do not paste the whole thing
  into chat.

## Testing a new endpoint

```bash
# 1. Confirm the call works for this key
curl -s "https://finnhub.io/api/v1/stock/recommendation?symbol=AAPL&token=$FINNHUB_API_KEY" | head -c 400

# 2. Re-run the full probe after any plan change
python probe_endpoints.py | head -30

# 3. Add to finnhub_free.py if it returned 200 with real data
```

## Free endpoint sweep (`finnhub_free.py`)

Exercises every endpoint the free key can reach. Modes:

| Command | What it does |
|---|---|
| `python finnhub_free.py` | Run all 27 endpoints, print a readable summary (truncated) |
| `python finnhub_free.py <filter>` | Run a single endpoint (e.g. `quote`, `news`, `financials-reported`) |
| `python finnhub_free.py --list` | Print the 27 working paths and exit |
| `python finnhub_free.py --json [path]` | Dump every response as compact JSON to `path` (default `finnhub_<utc_stamp>_compact.json`) |
| `python finnhub_free.py --json-pretty [path]` | Same, indented |
| `python finnhub_free.py <filter> --json [path]` | Dump just one endpoint's response |

JSON envelope:

```json
{
  "key": "abc123...wxyz",
  "fetched_at": "2026-06-24T17:54:36+00:00",
  "endpoints": {
    "GET /quote":                    {"ok": true, "data": {...}},
    "GET /news":                     {"ok": true, "data": [...]},
    "GET /stock/financials-reported": {"ok": true, "data": {...}}
  }
}
```

The full sweep produces a ~9 MB file (30k US tickers from `/stock/symbol`,
100 news items, 250 patents, 500 visa records, full XBRL filings, etc.).
Read with `json.load(open(p, encoding="utf-8"))` — the file is UTF-8.

## Quick references

- Swagger spec: <https://finnhub.io/static/swagger.json>
- Docs landing: <https://finnhub.io/docs/api/>
- AAPL sanity check: `python finnhub_free.py quote` (runs a single endpoint; full sweep with no args)

---
> Source: [The-Options-Guru/finnhub-free-dashboard](https://github.com/The-Options-Guru/finnhub-free-dashboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
