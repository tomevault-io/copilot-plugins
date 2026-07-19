## unofficial-pddikti-api

> Project-scoped rules for AI coding agents working on this repository.

# AGENTS.md

Project-scoped rules for AI coding agents working on this repository.

## Project

Unofficial reverse-engineered Go API for **PDDIKTI** (Indonesian Higher Education Database). Proxies requests through **FlareSolverr** to bypass Cloudflare, with in-memory caching and AES decryption.

- **Language:** Go 1.25 · **Module:** `pddikti-api`
- **Framework:** Gin (HTTP) · Zap (logging) · Swaggo (API docs)
- **External dependency:** FlareSolverr (headless browser, port 8191)

## Commands

```bash
# Run
make run              # Docker production
make dev              # Docker dev (Air hot-reload)
go run ./cmd/main.go  # Local (requires FlareSolverr running)

# Test
make test                                                # All tests
go test ./internal/... -cover -count=1 -timeout 120s     # With coverage
go test -run TestName ./internal/api/...                  # Single test

# Swagger
make swagger          # Regenerate docs/ from godoc annotations
```

## Architecture

```
Client → Gin Router → ZapLogger → ZapRecovery → BearerAuth → Handler → Cache → Scraper → FlareSolverr → PDDIKTI
```

- Handlers in `internal/api/handler.go` are thin pass-throughs — they map a route to an upstream path via `scraper.Get` or `scraper.GetBinary`.
- Scraper maintains a persistent FlareSolverr session with auto-recovery on stale sessions.
- `scraper.Fetcher` interface enables unit testing with mocks.
- AES-128-CBC key/IV are hardcoded (reverse-engineered from upstream), not configurable.

## Conventions

- **Swagger annotations:** Every handler must have full `swag` godoc annotations (`Summary`, `Description`, `Tags`, `Params`, `Success`/`Failure`, `Security BearerAuth`, `Router`). Run `make swagger` after changes.
- **Models:** DTOs live in `internal/models/models.go` with JSON tags and Swagger `example` annotations.
- **Logging:** Use `middleware.Logger` (Zap). Never use `fmt.Println` or `log` for application logging.
- **Config:** `internal/config/config.go` loads from `.env` and env vars. Note: `CacheTTLMinutes` and `CFRefreshMinutes` are hardcoded to 10 — they are not read from env.
- **Tests:** Use the `scraper.Fetcher` interface for mocking. Cache sweep interval is configurable in tests via `newWithSweepInterval`.
- **Route registration:** All routes are registered in `cmd/main.go` via `registerRoutes()`.

## Code Style

- Keep handlers simple — proxy pattern, no business logic unless necessary.
- Preserve all existing comments and docstrings unrelated to your changes.
- Every exported type and function must have a godoc comment.
- Error responses use `models.ErrorResponse` with appropriate HTTP status codes.

---
> Source: [0xtbug/unofficial-pddikti-api](https://github.com/0xtbug/unofficial-pddikti-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-19 -->
