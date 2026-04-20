## nimbus

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Nimbus is a blog recommendation system that aggregates tech blog updates and provides personalized recommendations. The system consists of three main Python services:

1. **blog_fetcher**: Fetches blog updates from RSS/Atom feeds and stores them in BigQuery
2. **blog_preprocessor**: Processes blog entries by generating summaries and embeddings using OpenAI API
3. **blog_recommender**: REST API service for blog recommendations using vector similarity search, includes CLI tool

## Development Commands

All services use `task` (Taskfile) for command orchestration and `uv` for Python dependency management.

### Blog Fetcher Commands
```bash
cd blog_fetcher
task install      # Install dependencies
task test         # Run tests
task fmt          # Format code (ruff + sqruff)
task run          # Run locally with mock BigQuery
task build        # Build Docker image
task push         # Push to Google Container Registry
task deploy       # Deploy to Cloud Run
```

### Blog Preprocessor Commands
```bash
cd blog_preprocessor
task install      # Install dependencies
task test         # Run tests with .env.test
task fmt          # Format code (ruff + sqruff)
task run          # Run locally with .env.local
task build        # Build Docker image
task push         # Push to Google Container Registry
task deploy       # Deploy to Cloud Run
```

### Blog Recommender Commands
```bash
cd blog_recommender
task install      # Install dependencies
task test         # Run tests
task fmt          # Format code (ruff)
task run          # Run API server locally
task build        # Build Docker image
task push         # Push to Google Container Registry
task deploy       # Deploy to Cloud Run
```

### CLI Tool Commands
```bash
# After task install in blog_recommender
uv run nimbus-cli --help          # Show help
uv run nimbus-cli search "query"  # Search blogs
uv run nimbus-cli interactive     # Interactive mode
```

## Architecture

### Blog Fetcher
- Reads feed URLs from `src/blog_fetcher/feeds.yaml`
- Fetches RSS/Atom feeds using feedparser
- Extracts article content with BeautifulSoup
- Stores entries in BigQuery (production) or logs to console (local mode)
- Deployed as Cloud Run job triggered by Cloud Scheduler

### Blog Preprocessor
- Reads blog entries from BigQuery
- Generates summaries using OpenAI API with Jinja2 templates
- Creates embeddings for semantic search using text-embedding-3-small model
- Stores processed data in DuckDB with VSS (Vector Similarity Search) extension
- Enhanced table schema includes: id, title, summary, link, tags, text, embedding, published
- Uses async/await for concurrent processing
- Supports both local and Google Cloud Storage backends for DuckDB persistence

### Blog Recommender
- REST API service using FastAPI for blog recommendations
- Performs vector similarity search using DuckDB VSS with cosine similarity
- Returns complete blog metadata from DuckDB (no BigQuery dependency)
- Includes CLI tool (`nimbus-cli`) for command-line access
- Supports multiple output formats: table, JSON, simple text
- Features interactive mode with search history
- Deployed as Cloud Run service with auto-scaling

## Key Implementation Details

- **Environment modes**: Services support `local`, `test`, and production modes via ENV variable
- **Logging**: Uses Google Cloud Logging in production, text format locally
- **SQL queries**: Stored in `sql/` directories, formatted with sqruff
- **Testing**: pytest with asyncio support for all services
- **Python version**: Requires Python 3.13+
- **Deployment**: 
  - blog_fetcher and blog_preprocessor: Cloud Run jobs
  - blog_recommender: Cloud Run service (for API availability)
  - All deployed to asia-northeast1 region
- **Vector Search**: DuckDB VSS extension with HNSW index for efficient similarity search
- **CLI Tool**: Typer-based CLI with rich formatting and interactive features

## Important Files

- Feed configuration: `blog_fetcher/src/blog_fetcher/feeds.yaml`
- Prompt templates: `blog_preprocessor/src/blog_preprocessor/prompt/*.j2`
- SQL queries: `*/src/*/sql/*.sql`
- Deployment configs: `*/deploy/*.yaml`
- CLI entry point: `blog_recommender/src/blog_recommender/cli/main.py`

## Recent Changes

- **DuckDB Schema Enhancement**: blog_embeddings table now includes full metadata (title, summary, link, tags)
- **BigQuery Dependency Removal**: blog_recommender no longer requires BigQuery access
- **CLI Tool Addition**: New `nimbus-cli` command for searching blogs from terminal
- **ChromaDB Removal**: Removed unused ChromaDB dependency from blog_preprocessor

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uu64) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
