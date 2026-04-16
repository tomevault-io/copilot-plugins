## gemini-extension

> You have access to the Browserless.io REST APIs through custom commands. These let you scrape webpages, take screenshots, generate PDFs, search the web, map site structures, run custom browser automation, download files, export pages for offline use, run Lighthouse performance audits, and crawl entire websites.

# Browserless Extension

You have access to the Browserless.io REST APIs through custom commands. These let you scrape webpages, take screenshots, generate PDFs, search the web, map site structures, run custom browser automation, download files, export pages for offline use, run Lighthouse performance audits, and crawl entire websites.

## Authentication

All Browserless API calls require a token. The token is available as `$BROWSERLESS_TOKEN` (configured via extension settings). The API base URL is `$BROWSERLESS_API_URL` (defaults to `https://production-sfo.browserless.io` if not set).

Before making any API call, resolve credentials:

```bash
# Token comes from extension settings (BROWSERLESS_TOKEN env var)
# Fall back to ~/.browserless/.env if settings aren't configured
[ -z "$BROWSERLESS_TOKEN" ] && [ -f ~/.browserless/.env ] && source ~/.browserless/.env
API_URL="${BROWSERLESS_API_URL:-https://production-sfo.browserless.io}"
```

If no token is available, ask the user to configure it via extension settings or run `/browserless:auth`.

## Available Commands

- `/browserless:auth` — Configure API token and region
- `/browserless:smart-scrape <url>` — Scrape a webpage with cascading strategies
- `/browserless:screenshot <url>` — Capture a screenshot of a webpage
- `/browserless:pdf <url>` — Generate a PDF from a webpage
- `/browserless:search <query>` — Search the web
- `/browserless:map <url>` — Discover all URLs on a website
- `/browserless:function <task>` — Execute custom Puppeteer JavaScript
- `/browserless:download <task>` — Run Puppeteer code and download files generated during execution
- `/browserless:export <url>` — Export a webpage in its native format (HTML, PDF, image, ZIP)
- `/browserless:performance <url>` — Run a Lighthouse audit (performance, accessibility, SEO)
- `/browserless:crawl <url>` — Crawl a website and scrape every discovered page

## API Regions

| Region | URL |
|--------|-----|
| SFO (US West, default) | `https://production-sfo.browserless.io` |
| LON (Europe) | `https://production-lon.browserless.io` |
| AMS (Amsterdam) | `https://production-ams.browserless.io` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/browserless) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
