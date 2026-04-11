## bskt-fter

> This project, named **IMP**, is a Go-based service designed to collect basketball match statistics from various external providers and persist them in a PostgreSQL database.

# Gemini CLI Context: IMP (Basketball Statistics Collector)

This project, named **IMP**, is a Go-based service designed to collect basketball match statistics from various external providers and persist them in a PostgreSQL database.

## Project Overview

*   **Purpose:** Collects games, team stats, and player stats from providers like `API_NBA`, `INFOBASKET`, and `SPORTOTEKA`.
*   **Architecture:** Follows **Hexagonal Architecture (Ports and Adapters)**:
    *   `app/internal/core`: Domain entities and models (e.g., `games`, `players`, `tournament_poll_logs`).
    *   `app/internal/ports`: Interfaces (contracts) for repositories and providers.
    *   `app/internal/adapters`: Implementations of ports (GORM for DB, specific provider logic).
    *   `app/internal/service`: Business logic, orchestration (scheduler, processor, persistence).
    *   `app/internal/infra`: Low-level HTTP clients, transformers for external APIs, and infrastructure factories.
*   **Main Components:**
    *   **Scheduler:** A background worker (`app/cmd/scheduler`) that manages distributed workers for each tournament with staggered start times.
    *   **Debug Server:** A small HTTP server (`app/cmd/debug-server`) for manual triggers and debugging.
    *   **Persistence:** Uses GORM to interact with PostgreSQL.
    *   **Poll Logging:** Every poll cycle is recorded in `tournament_poll_logs` for audit and determining the next poll interval.

## Technologies

*   **Language:** Go 1.25
*   **Database:** PostgreSQL
*   **ORM:** GORM
*   **Logging:** [composite_logger](https://github.com/Consolushka/golang.composite_logger) (Supports Console, File, and Telegram)
*   **Testing:** Testify, GoMock
*   **Rate Limiting:** golang.org/x/time/rate
*   **Migrations:** Goose
*   **Linting:** golangci-lint

## Building and Running

The project uses a `Makefile` for common tasks:

*   **Setup:** `make setup` (copies `.example.env` to `.env`).
*   **Infrastructure:**
    *   `make start`: Run PostgreSQL in background (Docker).
    *   `make stop`: Stop PostgreSQL.
*   **Running:**
    *   `make run-scheduler`: Start the background worker.
    *   `make run-debug`: Start the debug HTTP server (default port `:8080`).
*   **Database:**
    *   `make migrate`: Apply database migrations.
    *   `make create-migration name=<name>`: Create a new migration file.
*   **Quality:**
    *   `make test`: Run all tests quickly.
    *   `make test-with-coverage`: Run tests and generate coverage report (mocks excluded).
    *   `make lint`: Run `golangci-lint`.

## Development Conventions

*   **Strict Layering:** Never import `adapters` or `infra` directly into `core` or `service`. Use `ports` (interfaces).
*   **Pure Transformers:** Transformers in `app/internal/infra/<provider>` MUST be pure functions. They only map data and NEVER make network calls.
    *   **Data Normalization:** All percentage-based statistics (e.g., Field Goals Percentage) MUST be normalized to the `0.0 - 1.0` range (where 1.0 represents 100%) before being mapped to core entities, ensuring consistency across different providers.
*   **Player Processing (Discovery & Ingestion):**
    *   **Discovery**: Check if player exists in local DB.
    *   **Ingestion**: Fetch bio from provider only if missing or player is new.
*   **Naming:**
    *   Repositories should have interfaces in `ports` and implementations in `adapters/<name>_repo/gorm.go`.
    *   Constructors should follow the `NewGormRepo(...)` pattern.
    *   Receiver names in GORM adapters should be `g`.
*   **Resilience:** All long-running goroutines (scheduler workers) MUST use `defer composite_logger.Recover(ctx)` to handle panics.
*   **Testing Strategy:** 
    *   Use SQLite in-memory for repository testing (`app/pkg/dbtest`).
    *   Follow transactional pattern (`Begin`/`Rollback`) in `testify/suite` for isolation and speed.
    *   All Stats Providers MUST be covered by unit tests using `MockClientInterface`.
*   **Mocking:** Use `gomock` for testing. Mocks should be updated whenever an interface in `ports` changes.
*   **Documentation:** Always update `AGENTS.md` when architectural or significant logic changes are made.
*   **Environment:** Use `.env` for configuration. Mandatory variables include `DB_*` settings, provider API keys, rate limits, and logger settings.

## Directory Structure Highlights

*   `app/cmd/`: Entry points for the application.
*   `app/database/migrations/`: SQL migration files managed by Goose.
*   `app/internal/adapters/`: Database and external provider implementations.
*   `app/internal/core/`: Domain models.
*   `app/internal/ports/`: Interface definitions.
*   `app/internal/service/`: Core business logic and orchestrators.
*   `app/internal/infra/logger/`: Factory for configuring the external `composite_logger`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Consolushka)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Consolushka)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
