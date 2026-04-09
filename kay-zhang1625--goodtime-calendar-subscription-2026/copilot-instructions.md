## goodtime-calendar-subscription-2026

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an automated calendar scraper for "好時光女生運動樂園" (Goodtime Women's Sports Studio). It logs into the 17fit.com/goodtime booking site via Selenium, scrapes upcoming class reservations, and produces a `mycalendar.ics` file hosted via GitHub Pages for calendar subscription.

## Running the Scraper

```bash
pip install -r requirements.txt
python main.py
```

Requires environment variables `ICS_GOODTIME_USERNAME` and `ICS_GOODTIME_PASSWORD`, and a locally-installed Google Chrome.

## Architecture

The pipeline has three stages orchestrated by `main.py`:

1. **Scraping** (`spider_goodtime.py`) — Headless Selenium Chrome logs in and extracts class data; BeautifulSoup parses the HTML. Returns a list of events and a sync mode.

2. **JSON diffing** (`spider_goodtime.py:update_json_with_crawler()`) — Compares scraped results against `events.json` using composite keys (`date|start_time|title`). Returns one of four sync modes:
   - `None` — no changes, skip ICS update
   - `"init"` — first run, build from scratch
   - `"append"` — only new events, add incrementally
   - `"replace"` — cancellations detected, rebuild entirely

3. **ICS generation** (`gen_ics.py`) — Converts `events.json` to `mycalendar.ics` using the sync mode to decide between a full rebuild or an append. All times use `Asia/Taipei` (UTC+8).

## CI/CD

`.github/workflows/ics-action.yml` runs on a cron every 2 days (`0 4 */2 * *`) and on manual dispatch. It installs Chrome, runs `main.py`, then auto-commits `events.json` and `mycalendar.ics` and deploys to GitHub Pages.

The `mycalendar.ics` file is publicly accessible at `https://[USERNAME].github.io/[REPO]/mycalendar.ics` for calendar subscription.

## Key Implementation Details

- Login flow simulates clicks: username field → next button → password field → login button (CSS selector `.st-mb-0.st-text-black.st-p-3.st-w-full`)
- Events are scraped from `div#unfinish .row.record`; dates/times extracted via regex
- UIDs in the ICS file are generated from `date|start_time|title` to stay stable across runs
- Historical (past) events are preserved in the ICS file even after they occur

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kay-Zhang1625)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kay-Zhang1625)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
