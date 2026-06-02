## cove

> SQLite-backed TTL key-value cache library for Go. Module: `github.com/modfin/cove`.

# AGENTS.md - Cove (lite-cache)

SQLite-backed TTL key-value cache library for Go. Module: `github.com/modfin/cove`.

## Build & Test Commands

```bash
# Run all tests
go test ./...

# Run all tests (verbose)
go test -v ./...

# Run a single test by name (regex match)
go test -v -run="^TestCacheSetAndGet$" ./...

# Run tests matching a prefix
go test -v -run="^TestCache" ./...

# Run a specific subtest
go test -v -run="^TestCacheSetAndGetSize/__1_kB$" ./...

# Run benchmarks
go test -bench=. -benchmem ./...

# Run a single benchmark
go test -bench="^BenchmarkSetParallel$" -benchmem ./...

# Run fuzz tests (30 seconds)
go test -v -fuzz=FuzzGetSet -run="^$" -parallel=1 -fuzztime=30s ./...

# Build check (no output binary, just verify compilation)
go build ./...

# Vet (static analysis)
go vet ./...
```

There is no Makefile, no linter config, and no CI pipeline. The project relies on
standard `go test`, `go build`, and `go vet`.

## Dependencies

- **Go 1.23.0** (uses `iter.Seq` / `iter.Seq2` from Go 1.23)
- **CGo required** (`github.com/mattn/go-sqlite3` is a CGo SQLite driver)
- **Test dependency:** `github.com/stretchr/testify` v1.9.0

## Project Structure

All source files live in the root package `cove` (no `src/` directory):

```
cache.go          Core Cache struct, New(), functional options (Op), CRUD, batch, range, iterators
db.go             Low-level SQL operations (get, set, evict, vacuum, batch, iterators)
facad.go          TypedCache[V] generic facade using encoding/gob
lock.go           keyedMutex (per-key locking for thundering herd protection)
log.go            discardLogger (no-op slog.Handler)
models.go         Constants, KV[T] struct, Zip/Unzip helpers, NotFound sentinel error
utils.go          URITemp(), URIFromPath(), Hit(), Miss(), Vacuum()
examples/         Standalone example programs (each file has its own main)
```

Test files: `cache_test.go`, `facad_test.go`, `lock_test.go`, `x_benchmark_test.go`, `x_fuzz_test.go`.

## Code Style Guidelines

### Formatting

- Standard `gofmt` formatting (tabs for indentation)
- Backtick raw strings for SQL queries and pragma strings
- No strict line length limit, but keep lines reasonable

### Imports

- Single `import (...)` block per file
- Alphabetical sort within groups; stdlib and third-party are intermixed in the
  same group (not separated by blank lines) in production code
- In test files, `testify` and `go-sqlite3` may be separated into a second group
- Side-effect import for SQLite driver: `_ "github.com/mattn/go-sqlite3"`

### Naming Conventions

- **Package:** `cove` (lowercase, single word)
- **Files:** lowercase, underscores for tests. Benchmark/fuzz files prefixed `x_`
- **Constants:** `UPPER_SNAKE_CASE` (e.g., `NS_DEFAULT`, `MAX_PARAMS`, `NO_TTL`)
  -- this is a project convention, not standard Go style
- **Exported types/funcs:** `PascalCase` (`Cache`, `TypedCache`, `KV`, `Op`)
- **Unexported types/funcs:** `camelCase` (`query`, `keyedMutex`, `get`, `setTTL`)
- **Struct fields:** unexported `camelCase` for `Cache` internals; exported single
  uppercase letters for `KV[T]` (`K`, `V`)
- **Receiver names:** single letter matching type (`c` for `*Cache`, `t` for
  `*TypedCache`, `km` for `*keyedMutex`)
- **Abbreviations stay uppercase:** `NS`, `TTL`, `DB`, `KV`, `Itr`, `URI`
- **Sentinel error:** `NotFound` (not `ErrNotFound` -- project convention)

### Types & Generics

- Functional options pattern: `type Op func(*Cache) error`
- Heavy use of Go generics: `KV[T any]`, `TypedCache[V any]`, `Of[V any]`
- One small interface: `query` in `db.go` (abstracts `*sql.DB` and `*sql.Tx`)
- Prefer structs over interfaces; keep interfaces minimal

### Error Handling

- Standard `(value, error)` return pattern
- Wrap errors with `fmt.Errorf("context, %w", err)` -- lowercase messages, no period
- Use `errors.Is(err, NotFound)` for sentinel error checks
- Use `errors.Join(...)` to combine multiple errors (e.g., pragma setup, file cleanup)
- `Hit(err)` / `Miss(err)` helpers separate not-found from real errors
- Silence non-critical cleanup errors with `_ = expr` (e.g., `_ = tx.Rollback()`)
- Panic only for programming errors (e.g., unlock of unlocked mutex)

### Nil Handling

- Explicit nil checks before use (e.g., `if log == nil { ... }`)
- nil values coerced to empty byte slices: `if value == nil { value = []byte{} }`
- Generic zero values: `var z V` as return placeholder on error paths
- nil slices returned (not empty slices) when range/batch queries have no results

### Functions & Methods

- Always pointer receivers for `Cache` and `TypedCache`
- Closures used extensively: option functions, vacuum goroutines, iterators, tx callbacks
- Transaction pattern: `c.tx(func(tx *sql.Tx) error { ... })`

### Comments

- GoDoc-style comments on all exported functions and types
- `// WARNING` blocks in GoDoc for iterator methods (errors are silently dropped)
- Log messages prefixed with `[cove]`: `c.log.Debug("[cove] creating schema")`
- Inline comments are sparse; used only for non-obvious context

## Test Conventions

- **Framework:** Go `testing` + `github.com/stretchr/testify/assert`
- **Never use `require`** -- only `assert` (tests continue after failure)
- **Test packages:** `cove_test` (external/black-box) for most files;
  `cove` (internal) only for `lock_test.go` (accesses unexported `keyedMu`)
- **Test isolation:** each test creates its own cache via
  `cove.New(cove.URITemp(), cove.DBRemoveOnClose())` with `defer cache.Close()`
- **No shared fixtures, no TestMain, no mocks** -- all tests use real SQLite DBs
- **Subtests:** heavy use of `t.Run()` with `do` closures for parameterized tests:
  ```go
  do := func(size int) func(t *testing.T) {
      return func(t *testing.T) { /* test logic */ }
  }
  t.Run("__1_B", do(1))
  t.Run("__1_kB", do(1_000))
  ```
- **Test factory helpers** in `facad_test.go` (e.g., `TypedCache(t)`, `TypedComplexCache(t)`)
- **Naming:** mixed `TestCacheMethod` and `TestCache_Method` styles
- **Benchmarks:** use `b.RunParallel`, `b.ResetTimer`, `b.ReportMetric`
- **Fuzz tests:** `f.Add` for seed corpus, `f.Fuzz` for the fuzz function;
  seed corpus includes SQL injection attempts and non-UTF-8 sequences

## Architecture Notes

- SQLite in WAL mode with configurable pragmas
- Namespaces are separate tables in the same SQLite database
- Per-key locking in `GetOr()` prevents thundering herd
- Background vacuum goroutine handles TTL-based expiration
- `TypedCache[V]` wraps `Cache` with `encoding/gob` serialization
- Batch operations are chunked to respect SQLite's 999-parameter limit (`MAX_PARAMS`)
- Eviction callbacks are fired in goroutines: `go c.onEvict(kv.K, kv.V)`

---
> Source: [modfin/cove](https://github.com/modfin/cove) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-02 -->
