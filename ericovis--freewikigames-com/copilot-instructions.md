## freewikigames-com

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**freewikigames.com** is a game platform that scrapes Wikipedia, stores raw HTML in PostgreSQL, and uses AI to generate trivia questions and other game content. Players can play solo or multiplayer. The backend is entirely Go.

Go module: `github.com/ericovis/freewikigames.com`

## Common Commands

```bash
# Build all binaries
go build ./...

# Run all tests
go test ./...

# Run a single test
go test ./path/to/package -run TestName

# Run tests with verbose output
go test -v ./...

# Run linter (golangci-lint expected)
golangci-lint run ./...

# Format code
goimports -w .
```

## Architecture

The project follows a monorepo layout with distinct binaries and internal shared packages:

```
cmd/
  api/        — HTTP API server (game sessions, user-facing endpoints)
  migrate/    — Migration CLI (up, status)
  scraper/    — Wikipedia scraping worker
  worker/     — Background jobs (AI question generation, dedup analysis)
internal/
  db/         — Database access layer (PostgreSQL via pgx)
    migrations/ — Embedded SQL migration files
  scraper/    — Wikipedia fetch + HTML storage logic
  questions/  — AI-driven question generation and similarity/dedup logic
  game/       — Game session, lobby, and multiplayer state management
  ai/         — Ollama client integration for content generation (Gemma4)
```

### Key Architectural Decisions

**Scraper pipeline**: Scrapers fetch Wikipedia article HTML and store the full raw HTML in PostgreSQL (not just parsed content). Downstream workers read from this raw store to extract and analyze content.

**AI question generation**: A worker reads scraped articles and calls a local Ollama instance (Gemma4 model) to produce trivia questions. Before inserting new questions, a similarity check (via pgvector embeddings stored in Postgres) deduplicates against existing content.

**Multiplayer**: Real-time game sessions are managed server-side; the API binary handles both REST endpoints and WebSocket connections for live gameplay.

**Database**: PostgreSQL is the single store for raw HTML, structured game data, embeddings (pgvector), and user/session state.

## Database Package

The `internal/db` package implements connection management, DAO-style data access, and migration execution. See [`internal/db/DB.md`](internal/db/DB.md) for the full developer guide, including how to add DAOs, write migrations, and create cross-table services.

```bash
# Apply pending migrations
go run ./cmd/migrate up

# Check migration status
go run ./cmd/migrate status
```

## Scraper Package

The `internal/scraper` package implements all Wikipedia fetch logic. See [`internal/scraper/SCRAPER.md`](internal/scraper/SCRAPER.md) for the full developer guide, including how to add new crawler modes, the worker pool and rate-limiter design, and testing conventions.

## Local Development

Infrastructure runs via Docker Compose; the Go API is run directly on the host.

```bash
# Start infrastructure
docker compose up -d

# Stop infrastructure
docker compose down
```

Expected `.env` variables (never committed):

```
DATABASE_URL=postgres://freewiki:freewiki@localhost:5432/freewiki?sslmode=disable
OLLAMA_HOST=http://localhost:11434
OLLAMA_MODEL=gemma4:e2b
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ericovis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-16 -->
