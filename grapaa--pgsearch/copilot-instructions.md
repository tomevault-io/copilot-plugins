## pgsearch

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

pgsearch fetches building case documents (byggesaker) from Drammen municipality's public innsyn system, indexes them into PostgreSQL with pgvector, and provides hybrid search over the content.

## Build & Run

```bash
# Install (editable mode)
pip install -e .

# Run the CLI (interactive REPL menu)
pgsearch
```

No tests, linting, or CI are configured.

## Menu

```
1. Sett opp database        — creates tables, indexes, pgvector extension
2. Hybrid-søk               — search indexed documents
3. Vis statistikk           — document/chunk counts
4. Indekser byggesaker fra Innsyn — scrape, download, and index for a date or range
5. Avslutt
```

## Architecture

**Innsyn pipeline** (option 4): `cli.py` → `innsyn/pipeline.py` → `innsyn/scraper.py` + `innsyn/downloader.py` → `extractor.py` → `chunker.py` → `embedding.py` → `database.py`

1. Scrapes Drammen's GraphQL API for journal entries on a given date
2. Saves case metadata (`byggesaker` table) for all entries
3. Downloads PDFs to `data/raw/YYYY-MM-DD/{saksnr}/`
4. Skips image-only PDFs (no text layer detected via `fitz.get_text()`)
5. Extracts text: native PyMuPDF first, EasyOCR (GPU) fallback for blank pages
6. Splits into 1000-char chunks with 200-char overlap, paragraph-boundary aware
7. Embeds via Azure OpenAI (`text-embedding-3-small`, 1536 dimensions)
8. Upserts chunks into `document_chunks` table (atomic per document)

**Search** (option 2): `cli.py` → `searcher.py` → `embedding.py` + `database.py`
- Query is embedded, then hybrid search combines vector cosine similarity + Norwegian full-text search + metadata search
- Results are ranked using Reciprocal Rank Fusion (RRF), weights 0.4 / 0.2 / 0.4

## Key Design Details

- **Database schema:** Two tables — `byggesaker` (saksnr + jsonb metadata) and `document_chunks` (text chunks with HNSW vector index, GIN tsvector index, GIN jsonb index). The tsvector column is `GENERATED ALWAYS` using the `'norwegian'` dictionary.
- **Document identity:** `document_id` = `{saksnr_folder}/{filename}` for uniqueness across cases. Unique constraint on `(document_id, chunk_index)` with upsert on conflict.
- **Transactional writes:** `insert_chunks` uses `autocommit=False` — all chunks for a document commit atomically, preventing partial indexing on crash.
- **Config:** All settings loaded from `.env` via `python-dotenv`. Required vars: `PGSEARCH_DB_CONNECTION`, `AZURE_OPENAI_ENDPOINT`, `AZURE_OPENAI_API_KEY`. Optional: `AZURE_OPENAI_DEPLOYMENT`.
- **Connection string format:** Standard libpq key=value pairs (e.g., `host=localhost dbname=pgsearch user=postgres password=postgres`).
- **Logging:** Rotates daily to `logs/pgsearch.log.YYYY-MM-DD`, retained 90 days. Logs pipeline events at INFO, errors with full tracebacks at ERROR.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/grapaa)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/grapaa)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
