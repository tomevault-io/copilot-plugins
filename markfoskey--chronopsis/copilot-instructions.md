## chronopsis

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a web-based timeline visualization application that displays scraped Wikipedia data (primarily British and American novels) in an interactive, zoomable timeline. Users can zoom with the mousewheel, pan by dragging, and click entries to view Wikipedia articles in an overlay.

## Repository Structure

The project consists of three main components:

- **client/**: Angular 16 frontend application
- **server/**: Flask backend server serving CSV data via REST API
- **fetcher/**: Python scripts for scraping Wikipedia data

## Development Commands

### Client (Angular)
Navigate to the `client/` directory for all Angular commands.

**Development server:**
```bash
cd client
npm install  # First time only
ng serve
```
Access at http://localhost:4200/

**Build:**
```bash
cd client
ng build  # Production build to dist/client
ng build --watch --configuration development  # Development build with watch
```

**Tests:**
```bash
cd client
ng test  # Runs Karma/Jasmine tests
```

### Server (Flask)
Navigate to the `server/` directory.

**Run server:**
```bash
cd server
python timeline_server.py
```

The server reads CSV files on startup and serves data through `/events` and `/allevents` endpoints. The hardcoded host/port is configured in `timeline_server.py:66`.

### Data Fetching
Navigate to the `fetcher/` directory.

**Scrape Wikipedia category:**
```bash
cd fetcher
python category_scraper.py "Category Name"
```

This recursively scrapes Wikipedia categories to extract novel metadata (title, author, publication date, genre, article length, etc.) into CSV files.

## Architecture

### Client Architecture

The Angular application is built around a single main component (`TimelineComponent`) that manages an HTML5 canvas-based visualization:

- **TimelineComponent** (`client/src/app/timeline/timeline.component.ts`): Core component handling all rendering, user interaction (zoom/pan/click), and data fetching. Uses direct canvas manipulation for performance.

- **Label** (`client/src/app/timeline/label.ts`): Represents individual timeline entries. Handles layout calculation, text wrapping, collision detection, and rendering. Calculates font size and importance from article length.

- **TimelineDataService** (`client/src/app/timeline/timeline-data.service.ts`): Angular service for fetching data from Flask backend. Supports both filtered queries (`/events`) and bulk loading (`/allevents`).

**Key rendering logic:**
- Year-based scaling: Maps years to pixel coordinates using `scaleAndShift()` and `rescale()` utilities
- Label placement: Labels sorted by importance (article length), positioned to avoid overlaps using `setY()` methods
- Alternating colored bands mark year intervals (dynamically calculated based on zoom level)
- Labels colored by author hash for visual grouping

**Interaction:**
- Mouse wheel: zoom in/out, centered on cursor position
- Click and drag: pan timeline horizontally and vertically
- Click label: opens Wikipedia article in left overlay iframe
- Touch events supported for mobile (incomplete pinch-to-zoom)

### Server Architecture

Simple Flask server (`server/timeline_server.py`) that:
1. Loads CSV files into memory on startup
2. Serves two endpoints:
   - `/events?start_date=X&end_date=Y&max_events=N`: Filtered events by date range, sorted by article length
   - `/allevents`: Returns all events sorted by article length
3. Uses `fetch_utils.py` from fetcher directory to parse dates

**Data format:** CSV files must contain columns like `title`, `author`, `year`, `day_in_year`, `genre`, `country`, `article_length`, `page_url`.

### Data Fetching Architecture

Wikipedia scraping utilities in `fetcher/`:

- **category_scraper.py**: Main scraper. Recursively traverses Wikipedia categories using MediaWiki API, extracts structured data from article infoboxes, and writes to CSV. Includes article length and inbound link counts as importance metrics.

- **fetch_utils.py**: Shared utilities for date parsing and extraction.

- **data_fetcher.py**: Additional data fetching utilities.

- **process_csv.py**: CSV post-processing scripts.

## Important Configuration Details

**Hardcoded network configuration:**
- Flask server host: `192.168.86.92:5500` (in `server/timeline_server.py:66`)
- Angular service baseUrl: `http://192.168.86.92:5500` (in `client/src/app/timeline/timeline-data.service.ts:30`)

Change these to `localhost` or appropriate addresses for your environment.

**Active data source:**
Currently only loading `BritishAndAmericanNovels.csv` (see `server/timeline_server.py:43`). The `historic_inventions.csv` is commented out.

## Data Flow

1. Wikipedia data scraped by `category_scraper.py` → CSV files in `server/`
2. Flask server loads CSV on startup → in-memory cache
3. Angular client requests data via HTTP → TimelineDataService
4. Data converted to Label objects → positioned and rendered on canvas
5. User interactions update visible year range → triggers re-fetch and re-render

## Known Issues & Incomplete Features

- **Incomplete code:** `LabelTable` and `LabelHeightManager` classes in `timeline.component.ts` are partially implemented but not used
- **Touch gestures:** Pinch-to-zoom handler exists but is not fully implemented
- **Selective loading:** `updateTimelineDataSelective()` method exists but is disabled; currently loads all data upfront via `updateTimelineDataAll()`
- The `fixLabelCoords()` method (line 197) is incomplete

## Current Development Plan

### Completed

- ✅ **Stacking algorithm bug** (Fixed Oct 31, 2025)
  - Implemented virtual window filtering with LabelHeightManager
  - Labels now position consistently regardless of pan direction
  - See `STACKING_ALGORITHM_UPDATE.md` for details

- ✅ **Date parsing bug in scraper** (Fixed Nov 28, 2025)
  - Fixed `category_scraper.py` to correctly extract years from Wikipedia infoboxes
  - Bug: `dateutil.parser` with `fuzzy=True` was misinterpreting years as day numbers
  - Fixed hundreds of books wrongly dated 2024 (e.g., "War of the Worlds" 1898, "A Terrible Temptation" 1871)
  - Created `fix_incorrect_2024_years.py` script to repair existing CSV data

### Short-term Goals

1. **Fix Wikipedia overlay close functionality**
   - Currently no way to close the Wikipedia overlay once opened (besides clicking another label)
   - Need to add a close button or click-outside-to-close behavior

2. **Clean up book database**
   - Wikipedia scraper still produces imperfect data despite recent fixes
   - Specific known issues:
     - "The Grapes of Wrath" showing as 2001 (should be 1939)
     - "Rebecca" by Daphne du Maurier showing as 2001 (should be 1938)
     - "Lives of the Mayfair Witches" appears 3 times (duplicate entries for 1994)
   - Requires ongoing manual review and correction of CSV data
   - May need additional scraper improvements to handle edge cases

3. **Improve color hash algorithm for readability**
   - Current author color hash sometimes produces colors that are too light to read easily
   - Need to adjust algorithm to ensure all generated colors have sufficient contrast
   - Location: `client/src/app/timeline/timeline.component.ts` (color generation logic)

### Long-term Goals

- **Migrate to D3.js** for rendering (instead of raw canvas manipulation)
  - Would simplify axis rendering, zoom/pan behavior, and scale functions
  - Keep custom Label class and stacking logic (D3 allows full control)
  - Most timeline libraries (vis-timeline, TimelineJS) are too opinionated for the custom stacking priority requirements

- **Replace CSV with SQLite or PostgreSQL**
  - Current approach loads all data into memory
  - Database would enable efficient filtering, pagination, and full-text search

- **Move to static site generation**
  - Pre-build JSON files since data doesn't change frequently
  - Eliminate need for Flask server
  - Deploy to Vercel/Netlify for free hosting

### Recommended Order of Work

1. Fix Wikipedia overlay (quick win, unrelated to other work)
2. Fix stacking algorithm (framework-agnostic, needs fixing regardless of D3.js decision)
3. Clean up database (ongoing, can be done in parallel)
4. Evaluate D3.js migration (once stacking is fixed, may realize current approach works well)

**Rationale:** Fix the algorithm bugs first, then decide if framework migration is still necessary. Migrating to D3.js before fixing stacking logic would just mean rewriting buggy code in a new framework.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/markfoskey) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
