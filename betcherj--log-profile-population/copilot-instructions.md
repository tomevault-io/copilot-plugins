## log-profile-population

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Purpose

A standalone Python microservice (Dockerized) that ingests an iPhone camera roll or exported Photos folder and returns a structured JSON response of **cards** — each card representing one discrete real-world experience (e.g., a meal, hike, museum visit, bar). Each card groups related photos with metadata: text description and location.

This service is consumed by a larger application over HTTP. Card-building logic is downstream of this service; this repo handles ingestion only.

## Commands

```bash
# Install dependencies (dev)
pip install -r requirements-dev.txt

# Run the service locally
uvicorn app.main:app --reload --port 8000

# Run all tests
pytest

# Run a single test file
pytest tests/unit/test_normalizer.py -v

# Run a single test by name
pytest tests/unit/test_normalizer.py::test_normalize_heic_fixture -v

# Lint (must be clean before committing)
ruff check .

# Format
ruff format .

# Run DB migrations
alembic upgrade head

# Generate a new migration after model changes
alembic revision --autogenerate -m "description"

# Build Docker image
docker build -t log-profile-population .

# Run with docker-compose (includes volume mounts for storage)
docker-compose up

# Ingest an exported iPhone Photos folder (dev/testing)
python -m cli.ingest_folder --path ~/Desktop/iPhone_Export --output /tmp/manifests
```

## Architecture

### Core Concepts

**Card**: The primary output unit (downstream, not yet implemented). Groups photos representing one discrete experience + text description + location.

**Asset**: The primary storage unit in this service. One DB row per unique file, keyed by SHA-256 content hash.

**Ingestion pipeline** (per file):
`receive → hash → dedup_check → store_raw → exiftool+ffprobe (parallel) → normalize → upsert_db`

### Module Layout

```
app/
  main.py                    # FastAPI app factory + lifespan
  config.py                  # pydantic-settings (Settings class, env-driven)
  dependencies.py            # FastAPI Depends() providers
  models/                    # SQLAlchemy 2.x ORM models (import app.models to register all)
    asset.py                 # Asset, FileType, IngestionStatus
    raw_metadata.py          # RawMetadata (verbatim exiftool/ffprobe JSON blobs)
    normalized_metadata.py   # NormalizedMetadata (40+ mapped fields)
    asset_relationship.py    # AssetRelationship (Live Photo, AAE, burst)
  api/v1/
    ingest.py                # POST /ingest/upload, POST /ingest/folder, GET /ingest/health
  services/
    ingestion/
      pipeline.py            # Single-file orchestrator — THE central seam
      hasher.py              # SHA-256
      storage.py             # Local FS sharded by hash prefix (swap body for S3)
      classifier.py          # file_type + mime_type from extension
      relationships.py       # Live Photo pairing, AAE linking, burst detection
    metadata/
      exiftool.py            # subprocess wrapper → (dict, error)
      ffprobe.py             # subprocess wrapper → (dict, error)
      normalizer.py          # raw JSON → NormalizedMetadata fields (tag priority maps)
  workers/thread_pool.py     # Shared ThreadPoolExecutor for blocking subprocess calls
  db/session.py              # Async SQLAlchemy engine + session factory
cli/
  ingest_folder.py           # CLI runner for testing with macOS Photos exports
alembic/                     # DB migrations
tests/
  conftest.py                # Fixtures: in-memory SQLite, mock subprocess, file stubs
  fixtures/                  # sample_exiftool_heic.json, sample_ffprobe_mov.json, etc.
  unit/                      # test_hasher, test_classifier, test_normalizer, test_relationships, test_exiftool_wrapper
  integration/               # test_ingest_pipeline, test_ingest_api, test_cli
```

### Key Design Decisions

- **SHA-256 as identity key**: Stable across renames; enables dedup + idempotency. `assets.sha256` has a `UNIQUE` constraint.
- **Raw JSON stored verbatim**: `raw_metadata.raw_exiftool_json` + `raw_ffprobe_json` stored as-is. Re-normalization never requires re-running tools.
- **Subprocess concurrency**: exiftool and ffprobe run in `ThreadPoolExecutor`, wrapped with `asyncio.run_in_executor`. Both run in parallel via `asyncio.gather` for the same file.
- **Per-file sessions in CLI**: Each file in the CLI runner gets its own DB session to avoid concurrent session conflicts.
- **Storage sharding**: `STORAGE_ROOT/{sha256[:2]}/{sha256[2:4]}/{sha256}{.ext}` — mirrors Git object store. `storage_path` is stored relative to `STORAGE_ROOT` for portability.
- **Model import**: Always `import app.models` (the `__init__`) before instantiating any ORM object to ensure the mapper registry is fully populated.

### Database (SQLAlchemy 2.x, SQLite dev / Postgres prod)

Four tables:
- `assets` — one row per file; `sha256` unique index; `ingestion_status` state machine
- `raw_metadata` — verbatim exiftool JSON + ffprobe JSON; `JSONB` on Postgres
- `normalized_metadata` — 40+ mapped fields (GPS, time, camera, iPhone-specific, video); composite GPS index
- `asset_relationships` — Live Photo pairs (confidence 1.0 by ContentIdentifier, 0.7 by stem+timestamp), AAE sidecars, burst groups

### Metadata Extraction

```bash
# All files:
exiftool -json -G -a -u -ee <file>

# Video files additionally:
ffprobe -v error -print_format json -show_format -show_streams -show_chapters <file>
```

Install tools: `brew install exiftool ffmpeg`

### Testing Approach

- All subprocess calls mocked via `monkeypatch` / `patch("subprocess.run", ...)`
- DB uses `sqlite+aiosqlite:///:memory:` per test
- CLI integration tests use `cli_env` fixture to redirect storage + DB to `tmp_path`
- API tests swap the engine in `app.db.session` before `TestClient` starts (so the lifespan uses the test engine)

### HTTP API

```
POST /api/v1/ingest/upload   — multipart file upload; X-Content-SHA256 header for idempotency hint
POST /api/v1/ingest/folder   — { folder_path, recursive, skip_if_exists }
GET  /api/v1/ingest/health   — DB + exiftool + ffprobe status
```

### iPhone-Native Path (future)

What CANNOT be captured without an iOS client: album membership, favorites/hidden flags, face clusters, user captions, edit history, Memories, Shared Albums.

Future extension: iOS app exports `<stem>.logpop.json` sidecars alongside assets. Backend adds `.logpop.json` to `SIDECAR_EXTENSIONS` + one migration for `raw_sidecar_json` column. No architectural change needed.

### Docker

Stateless container; assets and DB stored on volume mounts (`/data/assets`, `/data`). Dockerfile installs `libimage-exiftool-perl` + `ffmpeg` via apt.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/betcherj)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/betcherj)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
