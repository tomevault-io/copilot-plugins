## your-calvinist-podcast-qa-database

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FastAPI application that extracts timestamped Q&A content from YourCalvinist Podcast YouTube videos, classifies them using Gemini AI, and serves searchable endpoints. Designed for serverless deployment on Vercel with Neon PostgreSQL.

The app now also exposes an MCP server for LLM clients and a human-facing `POST /v1/ask` endpoint for archive Q&A.

**Live API**: https://keithfoskey.calvinistparrot.com

## Development Commands

### Start Development Server
```bash
# Install dependencies
pip install -r requirements.txt

# Start API with auto-reload
uvicorn app.main:app --reload
```

### Local Video Processing
```bash
# Process all videos from file
python -m app.cli.backfill --input playlist_videos.txt

# Process only unprocessed videos
python -m app.cli.backfill --input playlist_videos.txt --skip-processed

# Skip AI classification (faster, for testing)
python -m app.cli.backfill --input playlist_videos.txt --skip-classification

# Dry run (don't save to database)
python -m app.cli.backfill --input playlist_videos.txt --dry-run

# Process limited number with delay
python -m app.cli.backfill --input playlist_videos.txt --limit 5 --delay 2

# Re-process all videos from stored transcripts (no YouTube API calls)
python -m app.cli.backfill --from-stored

# Re-process with limit
python -m app.cli.backfill --from-stored --limit 5 --delay 2
```

### Manual Timestamp Ingestion
```bash
# For videos with manually extracted timestamps
python -m app.cli.ingest_manual_timestamps
```

## Architecture

### Core Pipeline Flow (`app/ingest/pipeline.py`)

The `process_video()` function is the single source of truth for video processing.
`reprocess_from_stored_transcript()` re-processes from stored DB transcripts without YouTube API calls (for re-classification).

1. **Extract video ID** via `app/youtube/ids.py` - handles both IDs and URLs
2. **Fetch metadata** via YouTube Data API (`app/youtube/metadata.py`)
3. **Parse timestamps** from description (`app/qa/timestamp_parser.py`) - extracts questions with timestamps
4. **Fetch transcript** segments (`app/youtube/transcripts.py`) - gets timestamped text
5. **Store raw transcript** in `transcripts` table as JSONB for re-processing without YouTube API hits
6. **Slice answers** by timestamp windows (`app/qa/answer_slicer.py`) - start-to-next-start windows
7. **Classify with Gemini** (`app/qa/classify.py`) - optional, uses `categories.json` schema
8. **Persist to database** via CRUD operations in `app/db/crud.py`

### Database Schema

**Key tables** (see `app/db/models.py`):
- `videos` - YouTube video metadata and processing status
- `qa_items` - Question-answer pairs with timestamps (unique on `video_id, timestamp_seconds`), includes `passages TEXT[]` for cited Bible passages
- `tags` - Tag names (many-to-many with qa_items)
- `transcripts` - Raw JSONB transcript segments + full text (kept separate for performance)
- `ingest_jobs` - Lightweight queue for video processing (status: pending → processing → done/failed)

**Full-text search**:
- `qa_items.search_tsv` column (tsvector) - declared in ORM and populated by Postgres trigger
- Must apply DDL from `plan.md` Section 3 before search works
- Trigger auto-updates `search_tsv` on insert/update using `to_tsvector('english', question || answer)`

**Schema migrations**:
- Ad-hoc SQL lives in `migrations/`; apply manually against Neon with `psql "$DATABASE_URL" -f migrations/<file>.sql`
- `001_add_passages_column.sql` — adds `qa_items.passages TEXT[]`. Apply before deploying code that writes to this column.

### Database Access Patterns

- **Request scope**: Use `get_db` dependency from `app/dependencies.py` in FastAPI routes
- **Internal jobs**: Use `get_session()` context manager from `app/db/engine.py`
- **Never** keep sessions open across requests - serverless constraint
- CRUD helpers in `app/db/crud.py` handle upserts, job locking, and tag creation

### Authentication

Two auth methods handled in `app/dependencies.py`:
- `X-API-Key: {ADMIN_API_KEY}` - for manual ingestion triggers
- `Authorization: Bearer {CRON_SECRET}` - for Vercel cron jobs

Public GET endpoints require no auth.

### Serverless Constraints (Vercel)

- **Bounded execution**: Keep handlers short (<30s), use queue for long operations
- **No long-lived connections**: Close DB sessions immediately
- **Side-effect free**: GET handlers should be idempotent
- **Cron configuration**: `vercel.json` defines two daily cron jobs at 07:00 and 07:05 UTC

### Queue System

Lightweight queue using `ingest_jobs` table:
- `enqueue_video()` - adds job if not already queued
- `get_and_lock_pending_job()` - atomic SELECT FOR UPDATE to claim job
- Failed jobs retry up to 3 times (configurable in cron logic)
- Check `check_recent_videos.ipynb` for requeue utilities

### Ingestion Endpoints (`app/routers/ingest.py`)

Protected by API key or cron secret:
- `GET/POST /v1/ingest/check` - scans playlist for new videos, enqueues unseen ones
- `GET/POST /v1/ingest/run-one` - processes one pending job
- `GET/POST /v1/ingest/run-batch?max_jobs=N` - loops up to N jobs (Vercel cron uses max_jobs=5)

GET variants exist for Vercel cron compatibility.

### Public API (`app/routers/public.py`)

Only serves videos with `status='processed'`:
- `/v1/videos` - list with optional title search
- `/v1/videos/summary` - aggregated categories/subcategories/tags for faceting
- `/v1/videos/{youtube_id}` - single video with description
- `/v1/videos/{youtube_id}/questions` - Q&A for specific video
- `/v1/questions` - browse all questions with category/subcategory/tag filters (AND logic for tags)
- `/v1/questions/search?q=...` - full-text search using `plainto_tsquery` on `search_tsv`
- `/v1/ask` - human-facing archive endpoint with request body `question` plus `mode`
  - Both modes first call Gemini to extract search keywords from the natural-language question (`extract_search_query` in `app/qa/ask.py`), then pass the keywords to `search_archive`. This mirrors how MCP clients naturally distill queries before calling tools.
  - `mode="research"` returns retrieved sources only
  - `mode="answer"` retrieves top matches and generates a grounded answer from the strongest full answers
- `/v1/questions/{id}` - single question with full answer
- `/v1/categories`, `/v1/subcategories`, `/v1/tags` - metadata endpoints

**Filtering**: Tags are comma-separated for AND logic. Example: `?tags=Calvinism,Election`

**Shared retrieval**:
- Shared archive search and citation shaping live in `app/archive.py`
- Search results and answer lookups now include structured citation objects with the direct YouTube timestamp URL

### MCP API (`app/mcp_server.py`)

The app mounts an MCP server for LLM-facing access (no auth required):
- Streamable HTTP at `/mcp/` (preferred production transport)
- SSE compatibility at `/sse` (legacy; mounted at its own path to avoid shadowing FastAPI routes)

All tools carry `ToolAnnotations(readOnlyHint=True, destructiveHint=False, openWorldHint=False)` so MCP clients can auto-approve them.

Tools:
- `search_keith_archive` — full-text search with category/subcategory/tag filters
- `get_keith_answer` — fetch complete answer by question UUID
- `list_keith_topics` — list categories, subcategories, popular tags

Resources:
- `keith://topics` — full topic taxonomy as JSON; clients can read this without a tool call (cheaper than `list_keith_topics` for static data)

Prompts instruct clients to answer only from retrieved Keith Foskey archive material and to cite the returned source URLs.

### Configuration (`app/settings.py`)

Environment variables (via `.env`):
- `DATABASE_URL` - Neon PostgreSQL connection string (required)
- `YOUTUBE_API_KEY` - YouTube Data API key (required).
- `GEMINI_API_KEY` - Gemini classification (optional, skip with `skip_classification=True`)
- `ADMIN_API_KEY` - protects ingestion endpoints
- `CRON_SECRET` - Vercel cron authentication
- `PLAYLIST_ID` - default: YourCalvinist Live Q&A playlist
- `ANSWER_PREVIEW_LENGTH` - default: 500 chars (for list views of 2-hour podcasts)

`settings.validate()` warns if DATABASE_URL or YOUTUBE_API_KEY missing.

`GEMINI_API_KEY` is also required for `/v1/ask` (both modes use it for keyword extraction; answer mode additionally uses it for answer synthesis). Without it, `/v1/ask` falls back to raw question text for search, and answer mode returns 503.

### CORS (`app/main.py`)

Hardcoded allowlist:
```python
allowed_origins = [
    "http://localhost:3000",
    "http://localhost:8000",
    "https://keithfoskey.com",
    "https://www.keithfoskey.com",
]
```

Update when adding new frontends.

## Classification System

Gemini 3 Flash classifies Q&A into structured taxonomy from `categories.json`:
- Category (e.g., "Theology")
- Subcategory (e.g., "Soteriology")
- Tags (e.g., ["Calvinism", "Election"])
- Passages (e.g., ["Romans 9:10-13", "Genesis 3"]) — Bible passages cited or discussed

Classification is optional - set `GEMINI_API_KEY` or use `skip_classification=True` to bypass.

## Common Issues

### Failed Videos Not Retrying
If videos stuck at `status='failed'`:
- Cron now retries failed videos up to 3 attempts
- Manually requeue using `check_recent_videos.ipynb` → `requeue_failed_videos()`

### Private/Deleted Videos
Videos with error "Failed to fetch video metadata" are likely:
- Made private by creator
- Deleted
- Region/age restricted

These cannot be processed automatically.

### Search Not Working
If full-text search returns no results:
- Apply DDL from `plan.md` Section 3
- Run trigger creation BEFORE inserting data
- Backfill existing rows if trigger was added late:
  ```sql
  UPDATE qa_items
  SET search_tsv = to_tsvector('english', coalesce(question,'') || ' ' || coalesce(answer,''))
  WHERE search_tsv IS NULL;
  ```

## Monitoring

- **Check recent videos**: Run `check_recent_videos.ipynb`
- **Check Q&A counts**: Run `video_qa_counts.ipynb` to find videos without timestamps
- **API health**: `GET /health`
- **Interactive docs**: `/docs` (FastAPI auto-generated)

## API Response Building

**YouTube links with timestamps**:
```
https://www.youtube.com/watch?v={youtube_id}&t={timestamp_seconds}
```

**Answer previews**: First 500 chars (configurable) for list views. Full answer available at `/v1/questions/{id}`.

**Archive citations**:
- REST `/v1/ask` and MCP archive retrieval return structured citations with question id, video title, timestamp metadata, excerpt, and the final timestamped YouTube link.

## Development Notes

- No test suite present - use `/docs` interactive API or manual curl
- Keep answer_preview for 2+ hour podcasts (full answers can be 10-20 minutes of text)
- Re-processing is safe due to `(video_id, timestamp_seconds)` uniqueness constraint
- Transcripts stored as JSONB to avoid YouTube API rate limits on re-runs
- Classification can be re-run without re-fetching transcripts
- Summary endpoint uses SQL aggregation - keep DB column names aligned with ORM changes
- MCP is the LLM-facing interface; `/v1/ask` is the human-facing interface.
- `/v1/questions/search` remains the low-level search endpoint (raw `plainto_tsquery`); `/v1/ask` uses Gemini to extract keywords first, then searches.
- Keyword extraction (`app/qa/ask.py:extract_search_query`) uses Gemini Flash with `temperature=0` and `max_output_tokens=60` for fast, deterministic results. Falls back to raw question text on failure.

---
> Source: [Jegama/Your-Calvinist-Podcast-QA-Database](https://github.com/Jegama/Your-Calvinist-Podcast-QA-Database) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
