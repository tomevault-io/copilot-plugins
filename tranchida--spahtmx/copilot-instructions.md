## spahtmx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Development Commands

```bash
make dev          # Run templ watch + Air hot reload in parallel
make build        # Compile to bin/app
make templ        # Watch and regenerate Templ templates
make air          # Run dev server with hot reload (go tool air)
make tailwind     # Build Tailwind CSS (input.css -> static/css/styles.css)

go tool templ generate        # One-time template generation
go test ./...                 # Run tests (testify assertions)
SEED_DB=true go run cmd/server/main.go  # Run with database seeding
```

**Prerequisites:** PostgreSQL must be running (`docker compose up -d` starts it on port 5432).

## Architecture

Clean/Hexagonal Architecture with three layers:

- **Domain** (`internal/domain/`): Entities (`User`, `Prize`, `Laureate`), repository interfaces, and domain errors. No infrastructure imports.
- **App** (`internal/app/`): Service layer (`UserService`, `PrizeService`, `AuthService`) containing business logic. Depends only on domain interfaces.
- **Adapters** (`internal/adapter/`):
  - `database/`: Bun ORM implementations of repository interfaces. Uses internal `*Bun` structs with `bun` tags and `ToDomain()`/`FromDomain()` conversion functions at the boundary.
  - `web/`: Echo HTTP handlers and Templ templates. Static assets (HTMX, CSS) are embedded via `embed.FS`.

Entry point: `cmd/server/main.go` — wires everything together, defines `AuthMiddleware`, creates DB schema, and optionally seeds data.

## HTMX SPA Pattern

The app behaves as an SPA using HTMX without JavaScript frameworks:

- Templates use `hx-get` with `hx-target="#main-content"` and `hx-push-url="true"`
- Handlers check `HX-Request` header: HTMX requests get a fragment (nav + content), regular requests get the full `Base` template
- Auth redirects use `HX-Redirect` response header for HTMX requests, HTTP 303 for regular requests
- Cookie-based sessions (`session` cookie with username, 24h expiry)

## Templ Templates

Templates live in `internal/adapter/web/templates/*.templ` and compile to `*_templ.go` files. **Never edit `*_templ.go` files directly** — always edit the `.templ` source and regenerate.

## Database

PostgreSQL via Bun ORM. DSN is currently hardcoded in `cmd/server/main.go`. Tables are auto-created with `CreateTable().IfNotExists()`. Seed data includes default users (alice/bob/charlie, password: "password") and Nobel Prize data from `nobel-prize.json`.

## Configuration

Environment variables (`internal/config/config.go`):
- `PORT` — server port (default: 8080)
- `SEED_DB` — seed database on startup (default: false)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tranchida)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tranchida)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
