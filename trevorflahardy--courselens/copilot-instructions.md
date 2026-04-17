## courselens

> AI-powered course audit system for EGN 3000L (Foundations of Engineering Lab) at USF.

# Course Audit System — Claude Code Instructions

## Project Overview

AI-powered course audit system for EGN 3000L (Foundations of Engineering Lab) at USF.
See APP.md for vision, ARCHITECTURE.md for technical design, PLAN.md for implementation phases.

## Tech Stack

- **Backend**: FastAPI + aiosqlite + Pydantic v2 (strict mode)
- **Frontend**: Next.js 15+ (App Router) + Tailwind CSS v4 + shadcn/ui + Zustand
- **AI Engine**: Claude Code CLI (Max plan, subprocess spawned by FastAPI)
- **MCP Servers**: Canvas MCP (pre-existing), Chroma MCP (official), Audit MCP (custom FastMCP)
- **Database**: SQLite (`data/audit.db`) — WAL mode, foreign keys enforced
- **Graph**: NetworkX (in-memory, loaded from SQLite edges on demand)

## Directory Layout

```
backend/          Python backend (FastAPI + services + models)
frontend/         Next.js dashboard
audit_mcp/        Custom Audit MCP server (FastMCP) — named to avoid shadowing pip `mcp` package
data/             SQLite DB, ChromaDB, downloaded files
scripts/          Setup, seed, and utility scripts
tests/            pytest (backend), vitest (frontend), playwright (e2e)
.claude/commands/ Slash commands for audit operations
```

## Key Conventions

### Python
- All models use Pydantic v2 strict mode (`model_config = {"strict": True}`)
- When hydrating models from SQLite rows, use `Model.model_validate(dict(row), strict=False)` to allow str→StrEnum/datetime coercion
- Async everywhere: aiosqlite for DB, async FastAPI routes
- Line length: 100 chars (ruff)
- Type hints required on all public functions

### Frontend
- App Router (no pages/ directory)
- TypeScript strict mode
- Tailwind v4 with `@theme inline` directive
- shadcn/ui for all UI primitives
- Zustand for client state
- Native `fetch` + `EventSource` for API calls (no axios)

### Database
- All structured data in SQLite — no JSON files for mutable state
- `content_hash` (SHA-256 prefix) on nodes for change detection
- Finding lifecycle: active → stale → resolved/superseded/confirmed
- Parameterized queries only — never interpolate user input into SQL

## MCP Tool Namespaces

### Audit MCP (`audit_mcp/audit_mcp.py`)
- `nodes_*` — Course node CRUD (read, write/upsert, list, read_many, link, get_stale)
- `emit_*` — Finding emission (emit_finding, emit_resolve_stale)

> Graph logic lives in `backend/services/graph_service.py` — called internally by FastAPI routes and the audit engine, not exposed as MCP tools.

### Canvas MCP (external)
- Use `get_course_structure` as entry point for ingestion
- `get_assignment_details`, `get_page_content`, `get_rubric_details` for full content
- `download_course_file` for PDFs/DOCXs

### Chroma MCP (official)
- Collection: `course_nodes`
- Metadata filtering by `week` and `type`
- Batch upserts of 20

## Audit Principles

1. Every finding must quote specific text from the course content as evidence
2. Never say "could be clearer" without explaining exactly what is ambiguous
3. Findings must be actionable — an instructor should know what to fix
4. Severity levels: `gap` (must fix), `warn` (should review), `info` (observation), `ok` (verified correct)
5. Three audit passes: (1) standalone clarity, (2) backward dependencies, (3) forward impact

## Common Commands

```bash
make setup    # First-time setup (venv, deps, DB)
make dev      # Start backend + frontend dev servers
make seed     # Populate demo data
make test     # Run all tests
make lint     # Run ruff + eslint
make check    # Run mypy + tsc
make clean    # Remove generated data
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/trevorflahardy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
