## swimy

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ymca-swim-sync** is a Python CLI tool that fetches YMCA swim schedules (Lionville + West Chester, Open Swim + Lap Swim) and exports/syncs them to calendars. It supports multiple output sinks: ICS files, Google Calendar, and CalDAV.

## Development Commands

```bash
# Setup (creates venv, installs deps including optional extras)
./setup.sh

# Run the CLI
uv run ymca-swim-sync -d                              # Single day
uv run ymca-swim-sync -w --start 2025-12-22           # Week starting at date
uv run ymca-swim-sync -m                              # Month (~31 days)
uv run ymca-swim-sync -w --dry-run                    # Dry run (no calendar push)
uv run ymca-swim-sync -w --sink gcal --gcal-calendar-id primary  # Sync to Google Calendar

# Install optional dependencies
uv pip install -e ".[playwright]"   # For JS-rendered pages
uv pip install -e ".[gcal]"         # For Google Calendar sync
uv pip install -e ".[caldav]"       # For CalDAV sync
```

## Architecture

```
src/ymca_swim_sync/
├── cli.py          # Entry point, argument parsing, orchestration
├── config.py       # AppConfig dataclass, default URL/paths
├── fetch.py        # FetchResult, schedule fetching (requests + Playwright fallback)
├── model.py        # SwimEvent dataclass, UID generation, fingerprinting
├── store.py        # SQLite persistence, Diff calculation (added/removed/modified)
├── export_ics.py   # ICS file generation using icalendar
├── gcal.py         # Google Calendar OAuth + upsert
└── caldav_sink.py  # CalDAV upload
```

**Data Flow:**
1. `cli.py` parses args, creates `AppConfig`
2. `fetch.py` fetches HTML from YMCA schedule URL for each day in range, parses embedded JSON (or falls back to Playwright for JS-rendered content)
3. `model.py` defines `SwimEvent` with deterministic UID (SHA1 of url+title+location+room+times)
4. `store.py` persists events to SQLite, calculates diff from previous run
5. Sink modules (`export_ics.py`, `gcal.py`, `caldav_sink.py`) output to chosen destination

**Exit Codes:** Returns exit code 2 when schedule changes are detected (useful for cron alerting).

## Key Patterns

- **Fetching Strategy:** First attempts to parse schedule data from embedded JSON in HTML (Drupal/window.__DATA__ patterns). Falls back to Playwright for JS-rendered pages.
- **Change Detection:** Events have a `fingerprint()` method (SHA256 of key fields). The `store.py` module compares fingerprints to detect modifications.
- **State Directory:** Defaults to `~/.cache/ymca-swim-sync/` for SQLite DB and ICS output.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jpoley) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
