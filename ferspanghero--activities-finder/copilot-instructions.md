## activities-finder

> An automated tool that aggregates event listings from curated Metro Vancouver sources, filters by date/city/activity type, and presents a consolidated table.

# Activities Searcher

An automated tool that aggregates event listings from curated Metro Vancouver sources, filters by date/city/activity type, and presents a consolidated table.

Read `README.md` for full project description and architecture.

## Project Structure

- `src/` — deterministic extraction pipeline (models, parsers, renderers, fetch, CLI)
- `tests/` — pytest test suites (including `test_parsers/`, `test_renderers/`, `fixtures/`)
- `project_files/` — project documentation (v1/, v2/, v3/, v4/, v5/ subdirectories + backlog)
- `.claude/commands/` — slash command definitions

## Key Files

- `src/models.py` — `Event`, `FetchResult`, `SourceStatus` dataclasses
- `src/pipeline.py` — orchestrator: fetch → parse → aggregate
- `src/parsers/` — source-specific parsers (do604, dailyhive, rhythmchanges, showhub, infidelsjazz, bcaletrail)
- `src/renderers/` — markdown and HTML renderers
- `src/fetch_sources.py` — async parallel fetcher (raw HTML/JSON)
- `src/__main__.py` — CLI entry point for `python3 -m src`
- `.claude/commands/find-activities.md` — `/find-activities` slash command definition
- `pytest.ini` — test configuration

## Commands

- `python3 -m src --from YYYY-MM-DD --to YYYY-MM-DD [--format markdown|html] [--output FILE]` — deterministic pipeline (max 14 day range; use same date for single day)
- `pytest` — run all tests
- `pytest -m "not integration"` — run unit tests only

## Default City List

Vancouver, Burnaby, New Westminster, Richmond, North Vancouver (City), Lynn Valley, Deep Cove, West Vancouver, Horseshoe Bay, Surrey, White Rock, Port Moody, Port Coquitlam, Coquitlam, Langley (City), Langley (Township), Delta, Pitt Meadows, Maple Ridge, Mission, Abbotsford, Chilliwack, Squamish, Anmore, Belcarra

### City Modes

1. **Default** — No cities specified → use the full list above
2. **Additive** — User says "also include [cities]" → add to the default list
3. **Override** — User says "only [cities]" → replace the default list entirely

Examples:
- "What's happening today?" → default (all cities)
- "What's happening today, also include Whistler" → additive
- "What's happening in Port Moody only" → override (only Port Moody)

## Activity Types

1. Live music
2. Festivals
3. Sports events
4. Art exhibitions / gallery openings
5. Food/drink events (tastings, markets)
6. Outdoor activities (hikes, group runs)
7. Community events (markets, fairs)
8. Trivia nights / pub events
9. Theatre & performing arts
10. Cultural celebrations
11. Family events

Default: search all types. User narrows by specifying types in their query.

## Sources

Curated sources fetched and parsed by the `src/` pipeline. No broad web search.

### Source 1: Do604
- **Type:** Local aggregator (all event types)
- **Strategy:** HTML scrape
- **URL pattern:** `https://do604.com/events/{YYYY}/{MM}/{DD}`
- **Date in URL:** Yes — one page per day (fetched per-day in date ranges)
- **Fields:** Event name, venue, city, time

### Source 2: Daily Hive Vancouver
- **Type:** Local aggregator (all event types)
- **Strategy:** Parse `__NEXT_DATA__` JSON embedded in HTML
- **URL pattern:** `https://dailyhive.com/vancouver/listed/events?after={from_date}&before={to_date}`
- **Date in URL:** Yes — supports date ranges natively
- **Fields:** Event name, venue, city, date range
- **Note:** Returns multi-day events. Parser filters to events overlapping the query range.

### Source 3: Rhythm Changes
- **Type:** Jazz & live music listings
- **Strategy:** HTML scrape
- **URL pattern:** `https://rhythmchanges.ca/gigs/`
- **Date in URL:** No — page lists an entire month organized by week (Sun-Sat). The script notes which week the target date falls in.
- **Fields:** Event name, venue, date, time

### Source 4: ShowHub
- **Type:** Local live music & shows
- **Strategy:** HTML scrape
- **URL pattern:** `https://showhub.ca/weekly-listings/`
- **Date in URL:** No — page lists an entire week. Filter by target date from content.
- **Fields:** Event name, venue, date, time

### Source 5: Infidels Jazz
- **Type:** Jazz events
- **Strategy:** WordPress REST API (JSON)
- **URL pattern:** `https://theinfidelsjazz.ca/wp-json/tribe/events/v1/events/?start_date={from_date}&end_date={to_date}`
- **Date in URL:** Yes — supports date ranges natively
- **Fields:** Event name, venue, city, address, time, ticket price, description, event URL

### Source 6: BC Ale Trail
- **Type:** Brewery events aggregator (BC-wide)
- **Strategy:** Parse `data-event-times` JSON embedded in HTML event cards
- **URL pattern:** `https://bcaletrail.ca/events/?date-start={from_date}&date-end={to_date}`
- **Date in URL:** Yes — supports date ranges natively. Parser also filters by date range from embedded JSON timestamps.
- **Fields:** Event name, venue, city, address, time
- **Note:** Events span all of BC, not just Metro Vancouver. City data comes from embedded JSON.

## Behavior Rules

- Be exhaustive: process ALL source sections from the script output, don't stop early
- Include both free and ticketed events
- Note if venues are closed due to holidays
- Search for both scheduled events AND regular recurring programming
- Date filtering uses the `--from`/`--to` range (default: today for both)
- If a source section shows an ERROR, note it and continue with remaining sources

## Output Format

### Summary header
```
Found X events across Y cities from Z sources
```

### Event table
| Event Name | City | Exact Address | Date | Time | Source Link |
|---|---|---|---|---|---|

### Sources checked footer
```
Sources checked: Do604 (12), Daily Hive (8), Rhythm Changes (ERROR: ...), ShowHub (6), Infidels Jazz (1)
```

### Output files
- `--format html` → `events-YYYY-MM-DD.html` (single day) or `events-FROM_to_TO.html` (range) — self-contained HTML with sortable columns and clickable links. Generated by `src/renderers/html_renderer.py`.
- `--format markdown` → `events-YYYY-MM-DD.md` (single day) or `events-FROM_to_TO.md` (range) — markdown table. Generated by `src/renderers/markdown_renderer.py`.

---
> Source: [ferspanghero/activities-finder](https://github.com/ferspanghero/activities-finder) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
