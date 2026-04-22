## othrys

> Go-based coordination server enabling multiple developers using different AI coding tools (Claude Code/OmO, Cursor, Copilot, etc.) to collaborate on the same project without conflicts. Provides PM-driven design, LLM-powered task splitting, branch-per-agent isolation, module-level claim/lock coordination, and human-controlled merging.

# Othrys — Multi-Agent Collaboration Platform

## Overview
Go-based coordination server enabling multiple developers using different AI coding tools (Claude Code/OmO, Cursor, Copilot, etc.) to collaborate on the same project without conflicts. Provides PM-driven design, LLM-powered task splitting, branch-per-agent isolation, module-level claim/lock coordination, and human-controlled merging.

## Tech Stack
- **Language:** Go
- **Web Framework:** Fiber (Express-like, WebSocket support)
- **Database:** PostgreSQL (pgx driver)
- **Real-time:** WebSocket (bidirectional command-and-control)
- **Event System:** EventBus interface → PG LISTEN/NOTIFY (swappable to Redis/NATS)
- **CLI Framework:** Cobra
- **Containerization:** Docker + docker-compose
- **LLM Providers:** Anthropic Claude API (default), OpenAI (via provider interface)

## Project Structure
```
othrys/
  cmd/
    server/          — Server entry point (main.go)
    cli/             — CLI client entry point (main.go)
  internal/
    config/          — Configuration loading (env vars)
    server/          — Fiber app setup, routes, middleware
      handlers/      — REST endpoint handlers
    auth/            — API key validation middleware
    models/          — Database models (project, task, agent, claim, event)
    store/           — PostgreSQL repository layer (queries, migrations)
    events/          — EventBus interface + PG LISTEN/NOTIFY impl
    ws/              — WebSocket hub, connections, protocol, EventBus bridge
    coordinator/     — Core coordination logic (claims, conflicts, assignments, merge)
    planner/         — LLM task splitting service (provider interface)
    git/             — GitService interface + LocalGitService (CLI wrapper)
    cli/             — CLI command definitions, HTTP client wrapper
  migrations/        — SQL migration files
  server/            — Docker deployment (Dockerfile, docker-compose.yml, .env.example)
  client/            — Client installer (install.sh)
  claude-plugin/     — Claude Code slash commands + collab_guard hook
  Makefile
  go.mod
```

## Where to Look
- **Claim conflict logic** → `internal/coordinator/claims.go`
- **Task lifecycle** → `internal/coordinator/tasks.go`, `internal/coordinator/approval.go`
- **API endpoints** → `internal/server/handlers/`
- **Database queries** → `internal/store/*_store.go`
- **WebSocket protocol** → `internal/ws/messages.go`, `internal/ws/hub.go`
- **Event publishing** → `internal/events/bus.go` (interface), `internal/events/pgbus.go` (impl)
- **Git operations** → `internal/git/service.go` (interface), `internal/git/local.go` (impl)
- **LLM task splitting** → `internal/planner/`
- **CLI commands** → `internal/cli/cmd_*.go`
- **Claude Code plugin** → `claude-plugin/` (slash commands, collab_guard hook, setup scripts)
- **Server deployment** → `server/` (Dockerfile, docker-compose.yml, .env.example)
- **Client installer** → `client/` (install.sh, README)

## Architecture Decisions
- **Server has zero git dependency.** Git ops are client-side only via `GitService` interface in CLI. Server tracks branch names only. Docker image does NOT need git.
- **EventBus abstraction.** All event publishing goes through `EventBus` interface. Only `pgbus.go` touches PG LISTEN/NOTIFY. Coordinators and stores never call pg_notify directly.
- **Module = directory path prefix.** Claiming `src/auth/` means exclusive access to all files under that path. Path overlap = one path is prefix of the other.
- **Claims cache for collab_guard.** `collab_guard.py` reads local `.othrys/claims_cache.json` — no HTTP per write. Stale cache allows writes (never blocks on stale data).
- **Project-scoped API keys.** No OAuth for v1. Keys generated on project creation.
- **Branch naming:** `othrys/{agent-name}/{task-slug}`

## Conventions
- Use `pgx` for PostgreSQL (not `database/sql` + `lib/pq`)
- Parameterized queries everywhere — no string interpolation in SQL
- Status enums: validate transitions in coordinator layer, not store layer
- HTTP handlers: validate input → call store/coordinator → return JSON
- All REST endpoints require `Authorization: Bearer <api-key>`. Agent endpoints also require `X-Agent-Id` header
- Tests: use real PostgreSQL (testcontainers or docker-compose test service), not mocks for store tests. Mock EventBus for coordinator tests.
- Python hooks (collab_guard): stdlib only, no pip dependencies

## Anti-Patterns
- Do NOT call `pg_notify` outside `internal/events/pgbus.go`
- Do NOT call `os/exec` for git outside `internal/git/local.go`
- Do NOT make HTTP calls to Othrys server from `collab_guard.py` — use local cache only
- Do NOT add git as a server-side dependency
- Do NOT put business logic in HTTP handlers — delegate to coordinators
- Do NOT use direct PG LISTEN/NOTIFY in WebSocket bridge — go through EventBus interface

## Development
```bash
# Build
make build                    # Builds both server and CLI binaries

# Server (Docker)
make server-up                # Starts server + PostgreSQL
make server-down              # Stops everything

# Client install
make client-install           # Builds CLI + installs Claude Code commands

# Test
go test ./... -v -count=1     # Full test suite
go vet ./...                  # Static analysis

# CLI
othrys --help                 # CLI usage
othrys project create --name X --repo Y
othrys task list --status approved
othrys claim request --task X --path Y
othrys merge check
```

## Database
5 tables: `projects`, `tasks`, `agents`, `claims`, `events`
- Migrations in `migrations/` — applied automatically on server startup
- Key indexes: claims (project_id, path, status) WHERE active; tasks (project_id, status); events (project_id, created_at DESC)

## API
- REST: 16 endpoints under `/api/v1/`
- WebSocket: `ws://host/ws?token=<api-key>&agent=<agent-id>`
- See plan at `.claude/omo/plans/2026-03-13-multi-agent-collab-platform.md` for full API spec

## Plan
Full implementation plan with 55 tasks across 9 waves at:
`.claude/omo/plans/2026-03-13-multi-agent-collab-platform.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrzhbr) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
