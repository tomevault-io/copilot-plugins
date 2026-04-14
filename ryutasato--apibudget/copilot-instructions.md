## apibudget

> apibudget is a Go library for unified rate limiting and credit/token budget management of external APIs. It follows the `Allow/Reserve/Wait` pattern from `golang.org/x/time/rate`.

# AGENTS.md — AI Coding Assistant Guide

## Project Overview

apibudget is a Go library for unified rate limiting and credit/token budget management of external APIs. It follows the `Allow/Reserve/Wait` pattern from `golang.org/x/time/rate`.

## Architecture

- Single Go module: `github.com/ryutasato/apibudget`
- All core types are in the root package
- Server entry point: `cmd/apibudget-server/main.go`
- No internal packages — flat structure

## Key Types

| Type | File | Description |
|---|---|---|
| `Credit` | `credit.go` | `*big.Rat` wrapper for precise arithmetic |
| `BudgetManager` | `manager.go` | Main entry point — Allow/Wait/Reserve |
| `Reservation` | `reservation.go` | Reserve + Confirm/Cancel pattern |
| `Store` | `store.go` | Interface for counter/credit persistence |
| `memoryStore` | `memory_store.go` | In-memory Store with `sync.Mutex` |
| `redisStore` | `redis_store.go` | Redis Store with Lua scripts |
| `Config` | `config.go` | YAML config parser |
| `Server` | `server.go` | HTTP API server |
| `Logger` | `logger.go` | Logger interface + default slog impl |

## Conventions

- Error variables: `Err*` prefix in `errors.go`
- Constructors: `New*` prefix
- Options: `With*` prefix for functional options
- Tests: `*_test.go` co-located with source
- Property tests use `testing/quick`

## Building & Testing

```bash
# Build
go build ./...

# Test (all)
go test ./...

# Test with race detection
go test -race ./...

# Lint
golangci-lint run

# Build Docker image
docker build -t apibudget-server .

# Run with docker-compose
docker compose up -d
```

## Configuration

- YAML files define APIs and credit pools only
- Environment variables control server, store, and log settings
- See `apibudget.example.yaml` for a sample config
- See `docs/docs/configuration.md` for full reference

## Important Patterns

### Allow/Wait/Reserve

```go
// Non-blocking check
allowed, next := mgr.Allow("api-name")

// Blocking wait
err := mgr.Wait(ctx, "api-name")

// Reserve + confirm
r := mgr.Reserve("api-name")
if r.OK() {
    err := r.Confirm(actualCost)
}
```

### Credit is big.Rat based

Always use `NewCredit("1.5")` or `MustNewCredit("1.5")` — never float literals.

### Store interface

All counter and credit operations go through the `Store` interface. In-memory is default; Redis is for distributed setups.

## File Layout

```
├── credit.go / credit_test.go
├── config.go / config_test.go
├── manager.go / manager_test.go
├── reservation.go
├── store.go
├── memory_store.go / memory_store_test.go
├── redis_store.go / redis_store_test.go
├── window.go / window_test.go
├── logger.go / logger_test.go
├── server.go / server_test.go
├── errors.go
├── cmd/apibudget-server/main.go
├── Dockerfile
├── docker-compose.yaml
├── Makefile
└── docs/
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ryutasato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
