## zsp

> Go conventions for all Zapstore Go projects (zsp, server, zindex, zapstore-cli)


# Go Conventions

## Style

- Standard library style. Run `gofmt`. No third-party linters beyond `go vet`.
- Use `internal/` for private packages. Public API surface should be minimal.
- Prefer flat package structure within `internal/` — one package per concern, not deep nesting.
- Reference existing patterns in the same project before inventing new ones.

## Error Handling

- Wrap errors with `fmt.Errorf("context: %w", err)` — always add context.
- Return errors; don't panic. Panics are only acceptable for programmer bugs (unreachable code).
- Use `errors.Is` / `errors.As` for sentinel and typed error checks.
- Define sentinel errors as package-level `var ErrFoo = errors.New("foo")`.

## Testing

- Table-driven tests. Name subtests clearly.
- Test files live next to the code they test (`foo_test.go` beside `foo.go`).
- Use `testdata/` for fixtures.
- No test frameworks — standard `testing` package only.

## Dependencies

- Prefer the standard library. Add a dependency only when it saves significant complexity.
- All projects use `github.com/nbd-wtf/go-nostr` for Nostr operations.
- Pin dependency versions via `go.sum`. Run `go mod tidy` after changes.

## Concurrency

- Use `context.Context` for cancellation. Pass it as the first parameter.
- Prefer `sync.WaitGroup` or `errgroup.Group` over bare goroutines.
- No goroutine leaks — every goroutine must have a clear shutdown path.
- Use channels for communication, mutexes for state protection. Don't mix.

## Naming

- Short, clear names. `src` not `sourceManager`. `cfg` not `configuration`.
- Interfaces describe behavior: `Signer`, `Publisher`, not `ISignerInterface`.
- Acronyms are all-caps: `URL`, `HTTP`, `APK`, `ID`.

## Project Layout

- `main.go` — entry point, minimal logic, delegates to `internal/`.
- `internal/` — all business logic, one package per domain.
- `testdata/` — test fixtures, config examples.
- `Makefile` — build commands where applicable.

## Build

- Use `-ldflags` for version injection at build time.
- Support `go install module@latest` with embedded build info fallback.
- CGo is acceptable where needed (e.g. SQLite) but prefer pure Go when possible.
- `make` (default `build` target) produces a single binary named after the project at the repo root.
- `make all` cross-compiles for all supported platforms into `dist/` as `<binary>-<os>-<arch>`.
- Always pass `-trimpath -ldflags '-s -w'` for reproducible, stripped binaries.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zapstore) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
