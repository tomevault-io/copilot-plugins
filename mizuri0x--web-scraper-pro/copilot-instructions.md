## web-scraper-pro

> Professional web scraper: CSS selectors (BeautifulSoup) or XPath (lxml), proxy rotation,


# web-scraper-pro

Professional web scraper: CSS selectors (BeautifulSoup) or XPath (lxml), proxy rotation,
exponential-backoff retry, autopagination, and JSON/CSV/SQLite export.

## Quick Start

```bash
export SCRAPER_URL=https://example.com && bash run.sh
bash run.sh --url https://example.com --css-selector "h2, p" --format csv --output results
bash run.sh --url https://news.ycombinator.com --xpath "//span[@class='titleline']/a" --pages 1
```

## Environment Variables

| Variable | Required | Default | Description |
|----------|----------|---------|-------------|
| `SCRAPER_URL` | YES | -- | Target URL |
| `CSS_SELECTOR` | optional | h1-h3/p/a/li | CSS selector |
| `XPATH_SELECTOR` | optional | -- | XPath expression (priority over CSS) |
| `OUTPUT_FORMAT` | optional | json | json, csv, or sqlite |
| `OUTPUT_PATH` | optional | scraped_data | Path without extension |
| `PROXY_LIST` | optional | -- | Comma-separated proxy URLs |
| `REQUEST_DELAY` | optional | 1.0 | Seconds between requests |
| `MAX_PAGES` | optional | 10 | Max pages to crawl |
| `MAX_RETRIES` | optional | 3 | Retries per request |

## CLI

```
bash run.sh [--url URL] [--css-selector SEL] [--xpath EXPR]
            [--format json|csv|sqlite] [--output PATH]
            [--pages N] [--delay SEC] [--retries N]
```

## Selector Guide

| Situation | Use |
|-----------|-----|
| Elements by class/tag | CSS: `div.product h2` |
| Attribute matching | XPath: `//a[@class='story']` |
| No selector | Default: h1-h3, p, a, li |

## Output

Each item: `{"id", "tag", "text", "href", "source_url", "scraped_at"}`

Stdout (JSON, always parseable):
```json
{"status": "ok", "items_scraped": 142, "output": "scraped_data.json", "format": "json"}
```

## Examples

```bash
# Products -> CSV
bash run.sh --url https://shop.example.com --css-selector "h2.name" --format csv --output products

# Multi-page + proxies -> SQLite
export SCRAPER_URL=https://blog.example.com
export PROXY_LIST=http://p1:8080,http://p2:8080
export MAX_PAGES=20 OUTPUT_FORMAT=sqlite
bash run.sh

# Query result
sqlite3 scraped_data.db "SELECT tag, text FROM scraped WHERE tag='h2' LIMIT 20;"
```

---
> Source: [Mizuri0x/web-scraper-pro](https://github.com/Mizuri0x/web-scraper-pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
