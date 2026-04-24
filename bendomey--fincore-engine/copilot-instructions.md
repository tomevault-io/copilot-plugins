## fincore-engine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
# Run dev server (hot reload via reflex)
make run-dev

# Run production server
make run

# Build binary
make build-server

# Database migrations
make setup-db    # drop and recreate (init)
make update-db   # apply incremental migrations

# Lint (check only)
make lint

# Lint (auto-fix)
make lint-fix

# Install dev tools (gofumpt, golines, swag, reflex)
make install-tools

# Deploy to staging
make deploy-staging
```

There are no automated tests. Build verification: `go build ./...`

## Architecture

The app follows a strict layered architecture wired together through `pkg.AppContext`:

```
cmd/fincore-engine/main.go
  └── AppContext { DB, Config, Repository, Services, Handlers, Validator }
        └── internal/router/router.go  (chi routes)
              └── internal/handlers/   (HTTP layer — decode, validate, call service, encode)
                    └── internal/services/  (business logic)
                          └── internal/repository/  (GORM DB queries)
                                └── internal/models/  (GORM models)
```

Each layer only calls the layer directly below it. Handlers never touch the repository; services never touch HTTP.

**Transformations** (`internal/transformations/`) map GORM models to REST response maps. Always use these — never serialize a model struct directly to JSON.

**`lib` package** (`internal/lib/`) provides three cross-cutting helpers:
- `ValidateRequest` — runs go-playground/validator on any struct and writes a 422 response on failure
- `GenerateQuery` / `FilterQuery` — parses pagination/sort/search/populate URL params into a typed struct used by all list endpoints
- `ClientFromContext` / `WithClient` — store/retrieve the authenticated client from `context.Context`

## Request/Response Conventions

- All responses are JSON wrapped in `{"data": ...}` for single resources or `{"data": [...], "meta": {...}}` for lists
- Errors: `{"errors": {"message": "..."}}` for single errors or `{"errors": {"field": "rule..."}}` for validation errors
- Authentication: `VerifyAuthMiddleware` runs globally and attaches a client to context if both `X-FinCore-Client-Id` and `X-FinCore-Client-Secret` headers are present. `CheckForAuthPresenceMiddleware` is applied to protected route groups to reject unauthenticated requests.
- `POST /api/v1/clients` is the only public endpoint (no auth required)

## Adding a New Resource

1. **Model** — add to `internal/models/`, embed `BaseModel` or `BaseModelSoftDelete`
2. **Repository** — define interface + GORM implementation in `internal/repository/`; register in `repository.Repository` struct (`internal/repository/main.go`)
3. **Service** — define interface + implementation in `internal/services/`; register in `services.Services` struct (`internal/services/main.go`)
4. **Transformation** — add a `DBXxxToRestXxx` function in `internal/transformations/`
5. **Handler** — add handler struct + methods in `internal/handlers/`; register in `handlers.Handlers` struct (`internal/handlers/main.go`)
6. **Router** — add a new router file in `internal/router/` and mount it in `router.go`
7. **Migration** — add model to `AutoMigrate` call in `init/migration/main.go`
8. **OpenAPI spec** — add path to `api/service-specs/index.yaml`, schemas to `api/service-specs/schemas/`
9. **AI skill files** — update `api/llms.txt`, `api/llms-full.txt`, and `api/skill.md` (see below)

## Key Business Rules

- **Double-entry**: `validateLines()` in `internal/services/journal-entry.go` enforces that total debits == total credits across all lines of a journal entry
- **POSTED entries are immutable**: update and delete operations check `entry.Status == "POSTED"` and return an error
- **Group accounts**: cannot be deleted if they have child accounts; cannot be used as journal entry line accounts
- **Amounts**: stored as `int64` (integer cents/smallest currency unit)
- **`transaction_date`**: expected in RFC3339 format (`2006-01-02T15:04:05Z07:00`) — note the OpenAPI spec shows `date` format but the handler validates against RFC3339

## AI Skill Files

**Always update these three files whenever API changes are made** — new endpoints, changed fields, new business rules, or removed features:

- `api/llms.txt` — lightweight index (resource list, auth summary, links)
- `api/llms-full.txt` — full reference with every endpoint, field table, curl examples, and workflows
- `api/skill.md` — Claude Code slash command file (same depth as `llms-full.txt`, formatted as a command prompt)

These are served by the running API at `/llms.txt`, `/llms-full.txt`, and `/skill.md` and are the primary way API consumers' AI assistants discover and use FinCore. Keeping them stale undermines their purpose.

## Code Style

- Formatter: `gofumpt` (stricter than `gofmt`)
- Line length: max 120 characters (`golines`)
- Run `make lint-fix` before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bendomey) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
