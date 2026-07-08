## eps

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Is

**EPS** (Epstein Paper System) is a document research pipeline for collecting, OCR'ing, and searching public court filings and government releases related to the Jeffrey Epstein and Ghislaine Maxwell cases.

Three components:

1. **`scan_manager/`** — Python: web crawler + OCR pipeline + SQLite full-text search database
2. **`rest_api/`** — C++17 REST API server backed by MySQL, containerized with Docker

---

## Scan Manager (Python)

### Setup
```bash
cd scan_manager
python -m venv .venv
source .venv/bin/activate
pip install pyyaml pdf2image pytesseract pillow beautifulsoup4 rapidfuzz requests playwright playwright-stealth
playwright install chromium
```

System dependencies: **Tesseract OCR** (`brew install tesseract`), **Poppler** (`brew install poppler`)

### Run Modes

```bash
# --- Document collection ---
# Convenience wrapper (handles venv activation, subcommands): ./crawl.sh help
./crawl.sh prime                                    # ONE-TIME: solve any age gate/queue, save session
./crawl.sh all                                      # Crawl DOJ + CourtListener + DocumentCloud
./crawl.sh doj                                      # DOJ disclosure tree only
./crawl.sh cl                                       # CourtListener RECAP archive only
./crawl.sh dc                                       # DocumentCloud only
./crawl.sh reset                                    # Clear progress + saved session

# Or call the module directly:
python -m src.web.crawler --prime
python -m src.web.crawler
python -m src.web.crawler --source doj
python -m src.web.crawler --headless false
python -m src.web.crawler --reset

# --- Ingest & search ---
python src/main.py --mode ingest                   # OCR all PDFs → SQLite (parallel, resumable)
python src/main.py --mode ingest --workers 2       # Limit parallel OCR workers
python src/main.py --mode search --query "grand jury"          # Full-text search
python src/main.py --mode search --query "flight logs" --limit 50

# --- Legacy token pipeline ---
python src/main.py --mode scan                     # Full pipeline: OCR → tokenize → CSV
python src/main.py --mode scan_exclude             # OCR only files NOT already in CSV
python src/main.py --mode scan_include             # OCR only files already in CSV
python src/main.py --mode round_trip               # Re-import CSV corrections into JSON
```

### Data Flow

**Crawler** (collection):
```
DOJ / CourtListener / DocumentCloud  →  data/input/ (PDFs)
```

**Ingest** (processing):
```
data/input/ (PDFs)  →  OCR (Tesseract, 4 parallel workers)  →  data/epstein.db (SQLite + FTS5)
```

**Legacy pipeline**:
```
data/input/ (PDF/HTML) → data/output/ (raw JSON) → data/json/ (enriched JSON) → data/csv/merged.csv
```

### Database Schema (data/epstein.db)

SQLite with FTS5 full-text search:

- **`documents`** — `document_id`, `filename` (unique), `source` (cl/dc/doj), `page_count`, `full_text`, `created_at`
- **`pages`** — `page_id`, `document_id`, `page_number`, `page_text`
- **`documents_fts`** — FTS5 virtual table over `full_text`, auto-synced via triggers

### Architecture

#### Web Crawler (`src/web/`)
- **`src/web/crawler.py`** — Multi-source document crawler
  - **DOJ source**: Walks `justice.gov/epstein/doj-disclosures` tree with Playwright. Uses a persisted `data/doj_session_state.json` (from `--prime`), `playwright-stealth` fingerprint patches, and an init script that pads `navigator.{webdriver,languages,plugins,platform}`. Warmup retries with back-off and detects challenge pages by title/body markers before crawling.
  - **CourtListener source**: Searches RECAP archive REST API for each subject in `search_subjects.yaml`
  - **DocumentCloud source**: Searches public document API for Epstein/Maxwell-related terms
  - Downloads via Playwright `expect_download` (DOJ, handles age gates) or direct `requests` (CL/DC)
  - DOJ visited-section tracking is process-local — every run retraverses the tree so newly-published sections are picked up. Download dedup is persistent via `data/crawl_pdfs.txt`. DOJ session in `data/doj_session_state.json`.
- **`src/web/queries.py`** — Search query generator (combinatorial product of terms, doc types, people, places, dates, sites)

#### Ingest Pipeline (`src/core/`)
- **`src/core/ingest.py`** — Parallel OCR orchestration using `multiprocessing.Pool` + `imap_unordered`. Each worker runs Tesseract via pdf2image, results stream back to main process for DB insert. Idempotent (skips already-ingested files).
- **`src/core/database.py`** — SQLite schema init, document insert, FTS5 search queries. WAL mode for concurrent reads.
- **`src/core/search.py`** — CLI search interface over the FTS5 index with snippet highlighting.

#### Legacy OCR Pipeline (`src/core/`)
- **`src/core/doc_serializer.py`** — OCR engine: PDF via pdf2image+Tesseract, HTML via BeautifulSoup
- **`src/core/token_finder.py`** — Field extraction using KMP or regex, driven by `config/doc_template.yaml`
- **`src/core/round_trip.py`** — Bidirectional CSV↔JSON conversion
- **`src/plugin/scan_manager.py`** — High-level orchestration for legacy pipeline

#### Config (`config/`)
- **`config/search_subjects.yaml`** — Search subjects for the crawler: primary targets (Epstein, Maxwell), politicians, legal figures, business associates, entertainment, inner circle, public victims, and keyword queries. All names sourced from unsealed federal court filings and public records.
- **`config/doc_template.yaml`** — Token pattern definitions for legacy field extraction

### Data Directory Structure
```
scan_manager/data/
├── input/              # Downloaded PDFs (cl_*, dc_*, or bare names for DOJ)
├── output/             # Raw OCR JSON (legacy pipeline)
├── json/               # Enriched JSON (legacy pipeline)
├── csv/                # Merged CSV (legacy pipeline)
├── epstein.db          # SQLite database with FTS5 full-text search
├── crawl_pdfs.txt      # Crawler progress: downloaded document URLs (persistent dedup)
└── doj_session_state.json  # Playwright storage_state for DOJ (saved by --prime)
```

### File naming conventions in data/input/
- `cl_*.pdf` — CourtListener RECAP documents (e.g., `cl_gov.uscourts.flsd.123456.78.0.pdf`)
- `dc_*.pdf` — DocumentCloud documents (e.g., `dc_Ghislaine-Maxwell-Indictment.pdf`)
- No prefix — DOJ disclosure documents (e.g., `EFTA02732399.pdf`)

---

## REST API (C++)

### Prerequisites
Docker and Docker Compose. All build/run commands use the helper script.

### Setup & Build
```bash
cd rest_api
source bash_scripts/helper_script.sh

rest_api_build_dev   # Build Docker image
rest_api_run_dev     # Run container
```

### Build modes (run inside container)
```bash
re          # Light build: Wall, Wextra, Werror, address sanitizer
re_full     # Full debug build: g3, verbose warnings, address sanitizer
go_tests    # Build with Google Test
```

### Testing
```bash
# Unit tests (GTest) — run inside container after go_tests
./run_tests

# Integration tests (bash/curl) — run from host
bash tests/external_ExampleUser_route_tests.sh

# Manual test client
python mock_rest_client.py
```

### API (default port 3004)
```
GET    /exampleusers?limit=10&offset=0
GET    /exampleusers/:id
POST   /exampleusers   {"name":"...","email":"..."}
PUT    /exampleusers/:id
DELETE /exampleusers/:id
```

### Architecture
- **`src/core/rest_api.hpp`** — Crow HTTP framework routing and middleware
- **`src/core/mysql_conn_pool.hpp`** — Thread-safe MySQL connection pool (acquire/release with idle timeout and auto-reconnect)
- **`src/db_repository/example_repository.hpp`** — Repository pattern template; use this as the model for new tables
  - `IExampleUsersRepository` — virtual interface
  - `MySQLExampleUsersRepository` — production implementation using MySQL++ SSQLS
  - `FakeExampleUsersRepository` — in-memory fake for unit tests (enabled via `REPO_FAKE_ONLY` macro)
- **`src/main.cpp`** — Entry point; initializes MySQL credentials and connection pool
- **`bash_scripts/helper_script.sh`** — 20+ helper functions for Docker, build, and test operations
- **`bash_scripts/generate_mysqlpp_table.sh`** — Generates SSQLS struct definitions from SQL DDL

### Adding a new table/entity
1. Write SQL DDL, run `generate_mysqlpp_table.sh` to get the SSQLS struct
2. Copy `example_repository.hpp` pattern, implement the interface for the new struct
3. Add routes in `rest_api.hpp`
4. Add GTest tests mirroring `tests/test_example_repo.cc`
5. Add bash integration tests mirroring `tests/external_ExampleUser_route_tests.sh`

---
> Source: [Lbelus/eps](https://github.com/Lbelus/eps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
