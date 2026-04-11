## go-bus

> This repository is a Go publish/subscribe bus built on PostgreSQL. Use this file

# Agent Guide (go-bus)

This repository is a Go publish/subscribe bus built on PostgreSQL. Use this file
to orient automated agents to the build, test, and style conventions used here.

## Build, Lint, Test

### Go version

- Go toolchain version is defined in `go.mod` (Go 1.24).

### Lint

- CI runs golangci-lint (v1.64.8):
  - `golangci-lint run --timeout=5m`

### Build

- `go build -v ./...`

### Test (full suite)

- CI uses:
  - `go test -p=1 -count=1 -failfast -coverprofile=coverage.txt -coverpkg=./... ./...`

### Run a single package’s tests

- `go test ./...` (all packages)
- `go test ./pgbus` (PostgreSQL implementation)
- `go test ./quex` (worker utilities)
- `go test ./example` (integration tests)

### Run a single test

- By name across all packages:
  - `go test ./... -run TestSubscriptionDrain`
- In a specific package:
  - `go test ./pgbus -run TestSubscriptionDrain`

### Run subtests

- `go test ./... -run TestSubscriptionDrain/BasicDrainFunctionality`

### Notes for tests

- Tests use `github.com/theplant/testenv` with `DBEnable(true)`.
- Expect a PostgreSQL test environment (often via Docker) to be available.
- `TestMain` in `bus_test.go` runs migrations and truncates tables.

## Repo Map

- Root package (`bus`) contains core interfaces and types (types.go, bus.go, etc.).
- `pgbus/` PostgreSQL implementation and dialect.
- `quex/` worker utilities.
- `example/` integration-style tests and sample flows.
- `AI_USAGE_GUIDE.md` provides domain-level usage examples.

## Code Style Guidelines

### Formatting

- Use `gofmt` for all Go files.
- Keep line lengths readable, but do not manually wrap comments if gofmt will undo.

### Imports

- Use standard Go import grouping:
  - Standard library
  - Third-party packages
  - Local module packages
- Keep imports sorted and minimal.

### Package structure

- Root package is `bus` and defines the public API.
- `pgbus` is the implementation; keep cross-package dependencies clean.
- Prefer keeping public interfaces in `bus` and implementation details in `pgbus`.

### Naming

- Exported identifiers use PascalCase; unexported use camelCase.
- Prefer clear, domain-oriented names (`Subscription`, `Publish`, `Outbound`).
- Avoid abbreviations unless they are standard in Go (`ctx`, `db`, `tx`).

### Types and API design

- Use functional options for configuration (`WithPlanConfig`, `WithTTL`, etc.).
- Use `time.Duration` for time intervals and TTLs.
- Prefer strongly typed configuration structs (e.g., `PlanConfig`, `WorkerConfig`).

### Context usage

- Accept `context.Context` as the first parameter when needed.
- Do not store contexts inside structs.

### Errors

- Use `github.com/pkg/errors` for wrapping:
  - `errors.Wrap(err, "message")`
  - `errors.Wrapf(err, "format", ...)`
- Return sentinel errors from `errors.go` where appropriate.
- Preserve error causes when adding context.

### Logging

- Prefer `log/slog` where logging is needed.
- Do not log in hot paths unless necessary.

### JSON encoding

- The code uses explicit JSON encoding in `message.go` to preserve header keys.
- When encoding messages, use `json.NewEncoder` with `SetEscapeHTML(false)`.

### Headers

- `Header` is an alias for `http.Header`.
- Normalize header keys using `http.CanonicalHeaderKey` when persisting.
- When querying JSONB in SQL, use canonicalized header keys.

### Tests

- Use `testify/assert` and `testify/require`.
- Keep tests deterministic; avoid global state leaks.
- Prefer table-driven tests for coverage across patterns.
- Integration tests rely on DB setup; keep them isolated and clean up data.

### SQL / Dialect

- The PostgreSQL dialect lives in `pgbus/`.
- Keep SQL statements explicit and parameterized.
- For JSONB queries, match the stored schema exactly.

### Concurrency

- Avoid data races; protect shared state and use `sync` primitives where needed.
- Prefer explicit worker configs and bounded concurrency.

## CI / Workflow Notes

- GitHub Actions runs lint, build, and tests on `main` PRs.
- Linting uses golangci-lint with a 5 minute timeout.
- Tests run with `-p=1` for serial execution and `-failfast`.

## No Extra Rules

- No Cursor rules in `.cursor/rules/` or `.cursorrules`.
- No Copilot instructions in `.github/copilot-instructions.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qor5)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/qor5)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
