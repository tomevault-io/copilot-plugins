## stock-insight-skill

> Use when the user asks for real-time stock quotes, price checks, market data, or financial summaries for specific companies or tickers.


# Stock Insight

## Overview

This skill provides real-time stock market data using the `yfinance` Python library. It enables agents to fetch prices for US, HK, CN, and global markets.

**Key Capabilities:**
*   **Live Pricing**: Fetches real-time data for active markets.
*   **Extended Hours**: Automatically detects and returns **Pre-market** and **After-hours** prices for US stocks when available.
*   **Smart Fallback**: Prioritizes the most relevant "last trade" (e.g., Post-market price > Regular close) to ensure actionable insights.

## Usage

### Fetching Quotes

Use the `scripts/quote.py` script to fetch data for one or more tickers.

```bash
python3 /Users/j/.openclaw/skills/stock-insight/scripts/quote.py TICKER [TICKER ...]
```

### Ticker Suffixes

*   **US Markets**: No suffix (e.g., `AAPL`, `NVDA`, `TSLA`)
*   **Hong Kong**: `.HK` (e.g., `0700.HK` for Tencent, `06809.HK` for Montage Tech)
*   **Shanghai (A-Share)**: `.SS` (e.g., `600519.SS` for Moutai, `688008.SS` for Montage Tech A)
*   **Shenzhen (A-Share)**: `.SZ` (e.g., `000001.SZ`)

### Example Output

```json
{
  "MU": {
    "symbol": "MU",
    "name": "Micron Technology, Inc.",
    "price": 398.75,
    "market_state": "Pre-market",
    "currency": "USD",
    "change": 19.09,
    "pct_change": 5.03,
    "market_cap": 430946254848,
    "status": "success"
  }
}
```

*   **market_state**: Indicates if the price is from `Regular`, `Pre-market`, `After-hours`, or `Live` trading.

## Common Mistakes

*   **Wrong Suffix**: Forgetting `.HK` or `.SS` will often fail or return a different US OTC stock. Always check if the user implies a specific market (e.g., "A-share" -> .SS/.SZ, "Hong Kong" -> .HK).
*   **Market State Confusion**: If `price` differs significantly from yesterday's close, check `market_state`. Pre/Post-market volatility can be high and may not reflect the next official open.

---
> Source: [jae-jae/stock-insight-skill](https://github.com/jae-jae/stock-insight-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
