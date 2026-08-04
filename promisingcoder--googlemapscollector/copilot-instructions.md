## googlemapscollector

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Google Maps Business Extractor (`gmaps-extractor`) -- A pip-installable Python library that reverse-engineers Google Maps' internal protobuf API to extract business information at scale using raw HTTP requests. No browser automation, no official API key.

**Input:** Area name (e.g., "New York, USA") + Category (e.g., "lawyers")
**Output:** JSON + CSV files with all matching businesses, including details and reviews.

**Package names:** `gmaps-extractor` (PyPI), `gmaps_extractor` (import)
**Current version:** 2.0.0

## Installation

```bash
# Core library (httpx only, no FastAPI)
pip install gmaps-extractor

# With optional FastAPI server (for CLI or direct API access)
pip install gmaps-extractor[server]

# Development (testing, linting, type checking)
pip install -e ".[dev]"

# Documentation
pip install -e ".[docs]"
```

## Commands

### Running Tests
```bash
pytest                          # Run all 570+ tests
pytest tests/ -x               # Stop on first failure
pytest tests/test_client.py    # Run specific test file
pytest -k "test_search"        # Run tests matching pattern
coverage run -m pytest && coverage report  # With coverage
```

### Linting and Type Checking
```bash
ruff check gmaps_extractor/    # Lint
ruff format gmaps_extractor/   # Format
mypy gmaps_extractor/          # Type check
```

### Building
```bash
python -m build                # Build sdist and wheel
mkdocs build                   # Build documentation site
mkdocs serve                   # Local docs preview at localhost:8000
```

## Library Usage (v2.0.0)

### Sync (default -- no server needed)
```python
from gmaps_extractor import GMapsExtractor

with GMapsExtractor(proxy="http://user:pass@host:port") as extractor:
    result = extractor.collect("New York, USA", "lawyers", enrich=True)
    result = extractor.collect_v2("Paris, France", "restaurants", reviews=True)
```

### Async
```python
async with GMapsExtractor(proxy="http://user:pass@host:port") as extractor:
    result = await extractor.async_collect_v2("NYC", "lawyers", enrich=True)
    async for biz in extractor.stream_collect_v2("NYC", "lawyers"):
        print(biz["name"])
```

### Events
```python
from gmaps_extractor import GMapsExtractor, EventType

def on_found(event):
    print(f"Found: {event.data}")

with GMapsExtractor(proxy="...", on_business_found=on_found) as ext:
    ext.collect("NYC", "lawyers")
```

### Legacy (server-based, requires `[server]` extra)
```python
with GMapsExtractor(proxy="...", use_server=True) as extractor:
    result = extractor.collect("NYC", "lawyers")
```

## Architecture (v2.0.0)

```
gmaps_extractor/
├── __init__.py              # Package entry: exports, lazy imports, config shim, NullHandler
├── extractor.py             # GMapsExtractor class (high-level API) + CollectionResult
├── client.py                # GMapsClient: sync direct HTTP to Google Maps (DEFAULT)
├── async_client.py          # AsyncGMapsClient: async direct HTTP via httpx.AsyncClient
├── settings.py              # GMapsSettings dataclass (replaces monkey-patching config)
├── events.py                # EventEmitter + EventType + Event (lifecycle hooks)
├── progress.py              # ProgressReporter (attaches to EventEmitter)
├── config_manager.py        # ExtractorConfig (DEPRECATED - legacy bridge to config.py)
├── exceptions.py            # Custom exception hierarchy
├── _config_defaults.py      # Safe fallback config for pip-only installs
├── config.py                # Proxy, cookies, rate limits (gitignored, user-specific)
├── config.example.py        # Template config with placeholders
├── server.py                # FastAPI server (OPTIONAL: pip install gmaps-extractor[server])
├── cli.py                   # CLI: gmaps-collect (V1)
├── cli_v2.py                # CLI: gmaps-collect-v2 (V2)
├── cli_enrich.py            # CLI: gmaps-enrich-reviews
├── decoder/
│   ├── pb.py                # Decodes Google's !field_type_value protobuf format
│   ├── curl.py              # Parses curl commands
│   └── request.py           # Combined request decoder
├── parsers/
│   ├── business.py          # Extracts businesses from search response arrays
│   ├── place.py             # Extracts place details (hours, phone, website, photos)
│   └── reviews.py           # Extracts reviews from place responses
├── geo/
│   ├── grid.py              # GridCell, AreaBoundary, grid generation, boundary math
│   └── nominatim.py         # OpenStreetMap Nominatim API (boundaries + sub-areas)
└── extraction/
    ├── search.py            # Builds search queries (uses GMapsClient or legacy server)
    ├── enrichment.py        # Sync enrichment (details + reviews per business)
    ├── collector.py         # V1 sync orchestrator (parallel grid search)
    ├── collector_v2.py      # V2 sync orchestrator (resumable, adaptive, JSONL)
    ├── async_collector.py   # V2 async orchestrator (asyncio.gather/TaskGroup)
    └── async_enrichment.py  # Async enrichment (asyncio.Semaphore concurrency)

tests/                       # 570+ tests (pytest + pytest-asyncio)
docs/                        # MkDocs Material site
collect.py                   # Legacy CLI entry point (V1)
collect_v2.py                # Legacy CLI entry point (V2)
enrich_reviews_only.py       # Standalone reviews enrichment tool
run_server.py                # Starts the FastAPI server
pyproject.toml               # PEP 621 packaging, console scripts, tool config
CHANGELOG.md                 # Keep a Changelog format
```

## Layered Architecture

```
User code
  │
  ▼
GMapsExtractor (extractor.py)          ← High-level API, context manager, events
  │
  ├─── GMapsClient (client.py)         ← Sync HTTP to Google Maps (DEFAULT path)
  │    └── httpx.Client
  │
  ├─── AsyncGMapsClient (async_client.py) ← Async HTTP (for async_collect_v2, stream_collect_v2)
  │    └── httpx.AsyncClient
  │
  ├─── GMapsSettings (settings.py)     ← All config in one dataclass, passed explicitly
  │
  ├─── EventEmitter (events.py)        ← Lifecycle callbacks (zero overhead when unused)
  │    └── ProgressReporter (progress.py)
  │
  └─── [Legacy] FastAPI Server (server.py) ← Only when use_server=True
       └── uvicorn
```

## Data Flow

```
1. GMapsExtractor.collect_v2() / async_collect_v2() / stream_collect_v2()
       │
2. Nominatim API → Get area boundaries (AreaBoundary)
       │
3. Generate grid cells covering area (or subdivide → sub-areas → grids)
       │
4. Parallel search across all cells via GMapsClient.search():
   → Build search URL with protobuf parameters (build_search_url)
   → GET request to google.com/search?tbm=map&pb=...
   → Parse response: strip )]}'  prefix → JSON → extract_businesses()
   → Paginate (400 results per page, offset += 400)
   → Adaptive rate limiting with exponential backoff + jitter
   → Deduplicate by place_id AND hex_id
       │
5. Boundary filter: remove results outside target area + buffer
       │
6. [Optional] Parallel enrichment via GMapsClient.place_details() / reviews():
   → Place details: GET google.com/maps/preview/place?pb=...
   → Reviews: GET google.com/maps/rpc/listugcposts?pb=...
   → Cookie validation + auto-refresh on 429/consent redirect
       │
7. EventEmitter.emit(COLLECTION_COMPLETE, ...)
       │
8. Return CollectionResult / Save to JSON + CSV + JSONL
```

## Key Design Decisions

### Why direct HTTP instead of browser automation
Google Maps' internal search endpoint returns structured data (nested JSON arrays) via simple GET requests. No JavaScript rendering needed. This makes extraction 10-100x faster than Selenium/Puppeteer.

### Why grid-based search with deduplication
Google Maps limits results to ~400 per search query within a viewport. To get complete coverage of a large area, the area is divided into a grid of overlapping cells. Results are deduplicated by `place_id` and `hex_id` across cells.

### Why protobuf reverse engineering
The `pb` URL parameter encodes protobuf-like structured data. The format (`!{field}{type}{value}`) was reverse-engineered from browser network traffic. It controls search location, radius, pagination, and response fields.

### Why GMapsClient replaces the FastAPI server (v2.0.0)
The v1 architecture routed all Google requests through a local FastAPI server. This added unnecessary complexity (server lifecycle, port management, threading) and dependencies (FastAPI, uvicorn, pydantic). GMapsClient makes direct HTTP requests using httpx, eliminating these issues.

### Why GMapsSettings replaces config monkey-patching (v2.0.0)
The v1 config system used module-level globals that were monkey-patched at runtime via `ExtractorConfig.apply()`. This caused import-time binding issues and made the system fragile. GMapsSettings is a clean dataclass passed explicitly to components.

## Critical Code Paths -- DO NOT MODIFY WITHOUT UNDERSTANDING

### Protobuf String Builders (client.py)
`build_search_url()`, `build_place_pb_string()`, `build_reviews_pb_string()` construct the `pb` URL parameter for Google Maps endpoints. These are reverse-engineered from browser traffic and are extremely fragile. Changing field order, adding/removing fields, or modifying delimiters WILL break requests silently (Google returns empty results rather than errors).

### Parser Array Index Paths (parsers/)
Google Maps responses are deeply nested JSON arrays with no field names. Business data, place details, and reviews are extracted by hardcoded array indices. These indices correspond to specific fields in Google's internal protobuf schema:

**business.py** (search response):
- `data[i][14][11]` = business name
- `data[i][14][18]` = address
- `data[i][14][78]` = place_id
- `data[i][14][10]` = hex_id
- `data[i][14][89]` = ftid
- `data[i][14][4][7]` = rating
- `data[i][14][4][8]` = review_count
- `data[i][14][9][2]` = latitude
- `data[i][14][9][3]` = longitude
- `data[i][14][178][0][0]` = phone
- `data[i][14][7]` = website (needs URL extraction from /url?q=)
- `data[i][14][13]` = categories array

**place.py** (place details response):
- `data[6][11]` = name
- `data[6][18]` = address
- `data[6][78]` = place_id
- `data[6][4][7]` = rating
- `data[6][9][2/3]` = lat/lng
- `data[6][13]` = categories
- `data[6][34]` = hours (old format)
- `data[6][203][0]` = hours (new format, as of Jan 2025)
- `data[6][36]` = photos
- `data[6][178]` = phone
- `data[6][32]` = description
- `data[6][100]` = amenities/attributes

**reviews.py** (listugcposts response):
- `data[2]` = reviews array
- `data[1]` = next_page_token
- `review[0][1][4][5][0]` = author name
- `review[0][2][0][0]` = rating
- `review[0][1][6]` = date
- `review[0][2][15][0][0]` = review text

### Cookie Chain (client.py: `_fetch_fresh_cookies`)
The cookie fetch follows a specific sequence to establish a valid Google session:
1. Visit `google.com` -- gets initial cookies
2. Visit `consent.google.com` -- accepts cookie consent
3. Visit `google.com/maps` -- gets NID and AEC session cookies
If NID is not received, the session is invalid. A fresh SOCS consent cookie is generated with the current timestamp.

### Response Validation (client.py: `_is_valid_response`)
Invalid response signals: HTTP 429, 401, 403; redirect to `consent.google.com` (cookie expired); body < 100 chars after prefix strip. On invalid response, cookies are refreshed and the request is retried up to 2 times.

## Console Script Entry Points (pyproject.toml)

| Command | Module:Function |
|---------|-----------------|
| `gmaps-collect` | `gmaps_extractor.cli:main` |
| `gmaps-collect-v2` | `gmaps_extractor.cli_v2:main` |
| `gmaps-enrich-reviews` | `gmaps_extractor.cli_enrich:main` |
| `gmaps-server` | `gmaps_extractor.server:run_server` |

## Key Configuration (GMapsSettings defaults)

| Setting | Default | Purpose |
|---------|---------|---------|
| `results_per_page` | 400 | Results per Google Maps search request |
| `max_radius` | 5000 | Search radius in meters |
| `default_workers` | 20 | Concurrent cell queries |
| `max_workers` | 50 | Hard max for worker count |
| `delay_between_cells` | 0.05s | Rate limiting between cell queries |
| `delay_between_pages` | 0.1s | Rate limiting between pagination requests |
| `delay_between_details` | 0.2s | Rate limiting between detail requests |
| `delay_between_reviews` | 0.2s | Rate limiting between review requests |
| `cookies_ttl` | 1800s | Cookie cache TTL (30 minutes) |
| `viewport_dist` | 10000 | Viewport distance for search |

## Config Resolution Order

When using `GMapsExtractor` (library API), configuration priority is:
1. Constructor arguments (highest priority)
2. Environment variables (`GMAPS_PROXY_HOST`, `GMAPS_PROXY_USER`, `GMAPS_PROXY_PASS`, `GMAPS_COOKIES`)
3. `config.py` / `_config_defaults.py` values (lowest priority)

The new `GMapsSettings.from_env()` handles this resolution cleanly. The legacy `ExtractorConfig.apply()` is deprecated but still called for backward compatibility with modules that read from `config.py` globals.

## Exception Hierarchy (exceptions.py)

```
GMapsExtractorError (base)
├── ServerError          -- server start/connection failure (use_server=True only)
├── BoundaryError        -- Nominatim area resolution failure
├── ConfigurationError   -- invalid/incomplete config
├── RateLimitError       -- retry capacity exceeded
└── AuthenticationError  -- proxy or cookie auth failure
```

## Event System (events.py)

`EventType` constants:
- `COLLECTION_START` -- emitted with area, category, total_cells, mode
- `CELL_COMPLETE` -- emitted per cell with businesses_found, cells_remaining
- `BUSINESS_FOUND` -- emitted per business (high volume, no-op in ProgressReporter)
- `ENRICHMENT_START` -- emitted with total, workers
- `ENRICHMENT_COMPLETE` -- emitted per enriched business
- `RATE_LIMIT` -- emitted on backoff with delay_seconds
- `CHECKPOINT_SAVED` -- emitted on checkpoint save
- `SEARCH_COMPLETE` -- emitted after all cells queried
- `COLLECTION_COMPLETE` -- emitted at end with total_businesses, total_time
- `ERROR` -- emitted on errors with error, context

## Common Tasks

### How to add a new output field
1. Find the field's array index in a Google Maps response (use browser DevTools network tab)
2. Add extraction in the appropriate parser (`parsers/business.py` for search, `parsers/place.py` for details)
3. Add the field name to `CSV_COLUMNS` in `config.py` / `_config_defaults.py`
4. Add the field to `OUTPUT_SCHEMA` in `config.py` / `_config_defaults.py`
5. Add tests in `tests/test_parsers.py`

### How to update search template if Google changes format
1. Capture a working search request from Chrome DevTools (Network tab, filter by `search?tbm=map`)
2. Compare the `pb` parameter with `build_search_url()` in `client.py`
3. Update `build_search_url()` -- preserve exact field order and types
4. Also update `SEARCH_CURL_TEMPLATE` in `settings.py` and `config.py` (for legacy path)

### How to update parser indices if Google changes response format
1. Capture a response from Chrome DevTools and save the JSON
2. Navigate the nested arrays to find the field you need
3. Update the index path in the relevant parser
4. Update the docstring comments at the top of the parser file
5. Update the index mapping in this CLAUDE.md
6. Add a test fixture with the new response format

### How to add a new event type
1. Add the constant to `EventType` class in `events.py`
2. Emit it at the appropriate point: `self._events.emit(EventType.NEW_TYPE, key=value)`
3. Add a handler in `ProgressReporter` if it should produce console output
4. Add tests in `tests/test_events.py`

### How to add a new CLI command
1. Create `gmaps_extractor/cli_new.py` with `def main():` entry point
2. Add to `[project.scripts]` in `pyproject.toml`
3. Reinstall: `pip install -e .`

## Testing

Tests are in `tests/` using pytest + pytest-asyncio. 570+ tests cover:
- Parsers (business, place, reviews)
- Decoders (pb, curl, request)
- Geo modules (grid, nominatim)
- Event system (EventEmitter, ProgressReporter)
- Client (GMapsClient)
- Async client (AsyncGMapsClient)
- Settings (GMapsSettings)
- Async collector and enrichment

### Running tests
```bash
pytest                             # All tests
pytest tests/test_client.py        # Specific file
pytest -k "test_search"            # Pattern match
pytest --asyncio-mode=auto         # Already configured in pyproject.toml
```

### What to mock
- All HTTP requests (httpx.Client, httpx.AsyncClient) -- never hit real Google
- Nominatim API calls -- use fixture boundaries
- File I/O in collector tests -- use tmp_path

### Test configuration
`pyproject.toml` sets:
- `asyncio_mode = "auto"` -- async tests auto-detected
- `testpaths = ["tests"]`
- `filterwarnings = ["ignore::DeprecationWarning"]`

## Output Schema

```json
{
  "metadata": {
    "area": "New York, USA",
    "category": "lawyers",
    "boundary": { "name": "...", "north": ..., "south": ..., "east": ..., "west": ... },
    "search_mode": "grid | subdivision",
    "enrichment": { "details_fetched": true, "reviews_fetched": true, "reviews_limit": 20 }
  },
  "statistics": {
    "total_collected": 1234,
    "duplicates_removed": 89,
    "filtered_outside_boundary": 56,
    "search_time_seconds": 120.5,
    "total_time_seconds": 340.2
  },
  "businesses": [
    {
      "name": "...", "address": "...", "place_id": "ChIJ...",
      "hex_id": "0x...:0x...", "ftid": "/g/...",
      "rating": 4.5, "review_count": 123,
      "latitude": 40.7128, "longitude": -74.0060,
      "phone": "+1 212-555-0123", "website": "https://...",
      "category": "Lawyer", "categories": ["Lawyer", "Legal Services"],
      "found_in": "Manhattan, New York",
      "hours": { "monday": "9:00 AM - 5:00 PM" },
      "reviews_data": [{ "review_id": "...", "author": "...", "rating": 5, "text": "...", "date": "..." }]
    }
  ]
}
```

## Google Maps PB Parameter Format

The `pb` URL parameter uses `!{field}{type}{value}` format:
- `!1s` -- string (search query)
- `!7i` -- integer (results count)
- `!8i` -- integer (pagination offset)
- `!2d`/`!3d` -- double (longitude/latitude)
- `!74i` -- integer (max radius in meters)
- `!Nm` -- message (N nested fields follow)

## V2 Collector Extras

- **Checkpoint/resume**: Saves state to `output/.checkpoint_*.json`, auto-resumes on restart
- **Adaptive rate limiter**: `RateLimiter` / `AsyncRateLimiter` with exponential backoff + jitter
- **Parallel enrichment**: Separate worker pool (default 5 workers)
- **JSONL streaming**: Writes businesses as collected
- **Retry queue**: Failed cells retried with increased retries (5 attempts)
- **Dual dedup**: By both `place_id` and `hex_id`

## v2.0.0 Key Changes from v1.0.0

- **Default path**: Direct HTTP via `GMapsClient` (no FastAPI server)
- **Async**: `async_collect()`, `async_collect_v2()`, `stream_collect_v2()`
- **AsyncGMapsClient**: Full async counterpart of GMapsClient
- **GMapsSettings**: Clean dataclass replacing monkey-patched config globals
- **EventEmitter**: Lifecycle hooks (COLLECTION_START, CELL_COMPLETE, etc.)
- **ProgressReporter**: Pluggable progress output via events
- **Logging**: `logging.NullHandler` by default, `verbose=True` adds StreamHandler
- **FastAPI optional**: `pip install gmaps-extractor[server]` for legacy server mode
- **Core dep**: Only `httpx>=0.25.0` (no FastAPI/uvicorn/pydantic for core)
- **Cookie improvements**: Auto-retry on 429/consent redirect, proactive refresh every 500 requests
- **Request freshness**: Rotating UA pool, browser-like headers, epoch timestamp in pb
- **570+ tests**: Full test suite with pytest-asyncio

---
> Source: [promisingcoder/GoogleMapsCollector](https://github.com/promisingcoder/GoogleMapsCollector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
