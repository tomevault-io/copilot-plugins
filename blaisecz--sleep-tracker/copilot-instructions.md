## sleep-tracker

> Go REST API for sleep tracking with PostgreSQL/GORM.

# Sleep Tracker API

Go REST API for sleep tracking with PostgreSQL/GORM.

## Stack
- Go 1.22, chi/v5, GORM, validator/v10, swaggo

## Structure
- `cmd/api/` - entrypoint
- `internal/api/` - handlers, middleware, router
- `internal/domain/` - entities, DTOs, errors
- `internal/repository/` - database layer
- `internal/service/` - business logic
- `pkg/` - shared utilities

## Commands
- `make run` - start server
- `make test` - run tests
- `make docker-up` - start containers

## Conventions
- UTC timestamps, RFC 9457 errors, cursor pagination

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BlaiseCz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
