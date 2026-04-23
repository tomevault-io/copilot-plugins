## commute-tracker

> Commute Tracker is a Telegram bot that monitors UK train routes via the National Rail Darwin OpenLDBWS SOAP API and sends real-time push notifications about delays, cancellations, and platform changes. See `docs/DFA.md` for the full design document.

# CLAUDE.md

## Project Overview

Commute Tracker is a Telegram bot that monitors UK train routes via the National Rail Darwin OpenLDBWS SOAP API and sends real-time push notifications about delays, cancellations, and platform changes. See `docs/DFA.md` for the full design document.

## Tech Stack

- **Language:** Go 1.26.1
- **Database:** PostgreSQL 18 (via sqlc for query generation, golang-migrate for migrations)
- **Cache/Queue:** Redis (caching now, Celery-style task queue in the future)
- **Bot Framework:** telebot v4 (`gopkg.in/telebot.v4`)
- **Train API:** National Rail Darwin OpenLDBWS (SOAP/XML)
- **Deployment:** Docker Compose

## Project Structure

```
cmd/bot/main.go          — entrypoint
internal/bot/            — Telegram bot setup, handlers, state machine, inline button UI
internal/config/         — configuration loading from environment
internal/darwin/         — National Rail SOAP client (implements TrainClient interface)
internal/db/             — sqlc generated code + DB connection helpers
internal/domain/         — domain models (TrainStatus, NearestTrains, UserState) + interfaces (TrainClient, Notifier)
internal/redis/          — Redis client wrapper (cache + future queue)
internal/station/        — UK CRS station codes with fuzzy search (embedded JSON)
internal/timezone/       — UK timezone helper (Europe/London init, Now())
internal/tracker/        — Daily Planner (cron), Tracker Daemon (polling), Circuit Breaker
migrations/              — golang-migrate SQL files (000001_create_users, 000002_create_routes)
queries/                 — sqlc SQL query definitions
.github/workflows/pr-check.yml — GitHub Actions PR checks (runs on PRs to main)
docs/DFA.md              — design document
docs/PLAN.md             — implementation plan
```

## Key Architecture Decisions

- **Interfaces in `domain/`**: `TrainClient` and `Notifier` are defined in the domain package. Implementations live in their own packages (`darwin/`, `bot/`). This enables easy swapping and testing with mocks.
- **Redis is dual-purpose**: used for caching (service IDs, LastKnownState) now, and will support task queue functionality later. Keep the `internal/redis/` package generic.
- **Flat package layout**: prefer `internal/darwin/` over `internal/provider/rail/`. Each external integration gets its own top-level internal package.
- **Circuit Breaker lives in `tracker/`**: it's part of the monitoring logic, not a standalone utility.

## Dev Tools

| Purpose     | Tool                             |
|-------------|----------------------------------|
| Formatting  | `gofmt`                          |
| Linting     | `golangci-lint`                  |
| Testing     | `go test -race`                  |
| Mocks       | `mockgen` (`go.uber.org/mock`)   |
| SQL codegen | `sqlc`                           |
| Task runner | `Makefile`                       |

## Commands

All commands are available via `make`:

```bash
make run            # Run the bot
make build          # Build binary to bin/bot
make test           # Run tests with race detector
make test-cover     # Run tests with coverage report
make lint           # Run golangci-lint
make fmt            # Format code (gofmt + goimports)
make generate       # Run all code generation (sqlc + mockgen)
make sqlc           # Generate sqlc code
make mock           # Generate mocks (go generate)
make migrate-up     # Apply all migrations
make migrate-down   # Rollback last migration
make migrate-create # Create a new migration file
make docker-up      # Start infrastructure (postgres + redis)
make docker-down    # Stop infrastructure
make docker-logs    # Tail infrastructure logs
make test-v         # Run tests with verbose output
```

## Code Style & Conventions

- Use `slog` for structured logging (not `log` or third-party loggers).
- Follow standard Go project layout: `cmd/`, `internal/`, no `pkg/`.
- Keep packages small and focused — one responsibility per package.
- Interfaces are defined where they are consumed (in `domain/`), not where they are implemented.
- Error handling: wrap errors with `fmt.Errorf("context: %w", err)` for traceability.
- Context propagation: pass `context.Context` as the first parameter in all functions that do I/O.
- No global state. Dependencies are injected via constructors.
- No comments in code. Code must be readable and self-documenting through clear naming and structure.
- Always use established design patterns (e.g., constructor functions, functional options, repository pattern, strategy pattern) rather than ad-hoc solutions.

## Workflow

- After every code change, verify that all documentation is up to date. If not — update it immediately. This includes:
  - `docs/PLAN.md` — check off finished tasks, mark phases as done, add new phases for new features.
  - `docs/DFA.md` — update feature descriptions, sequence diagrams, actions, and test coverage requirements.
  - `CLAUDE.md` — update project structure, commands, or conventions if they changed.

## Testing

- Mock external interfaces (`TrainClient`, `Notifier`) in tests.
- Bot state machine transitions must have full unit test coverage.
- Circuit Breaker state changes (`API_UP` <-> `API_DOWN`) must be fully tested.
- Use table-driven tests where applicable.
- Run `go test ./...` before committing.

## Environment Variables

Defined in `.env` (never committed). Required:

- `TELEGRAM_BOT_TOKEN` — from @BotFather
- `NATIONAL_RAIL_TOKEN` — Darwin OpenLDBWS API key
- `DATABASE_URL` — PostgreSQL connection string
- `REDIS_URL` — Redis connection string

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yurii-merker) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
