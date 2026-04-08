## pixe

> make build                # GoReleaser snapshot → ./pixe (production build)

# AGENTS.md — pixe

## Build & Run

```bash
make build                # GoReleaser snapshot → ./pixe (production build)
make build-debug          # go build with debug symbols (for dlv)
make run ARGS="sort -h"   # build + run with arguments
make clean                # remove binary, dist/, coverage files
```

## Test Commands

```bash
make test                 # unit tests only (excludes integration/)
make test-all             # all tests including integration
make test-integration     # integration tests only

# Single test by name:
go test -race -timeout 120s ./internal/pipeline/ -run TestRun_basicSort -v

# Single package:
go test -race -timeout 120s ./internal/archivedb/...

# With coverage:
make test-cover           # prints coverage summary
make test-cover-html      # opens HTML report in browser
```

Always use `-race` when running tests. The Makefile sets `TEST_FLAGS := -race -timeout 120s`.

## Lint & Format

```bash
make lint                 # golangci-lint run ./...
make vet                  # go vet ./...
make fmt                  # gofmt -w -s .
make fmt-check            # check formatting without modifying (CI gate)
make check                # fmt-check + vet + unit tests (fast pre-commit gate)
```

CI runs `golangci-lint` and `go test -v -short ./...` on every push/PR to main.

## Code Style

### File Header

Every `.go` file starts with the Apache 2.0 copyright block:

```go
// Copyright 2026 Chris Wells <chris@rhza.org>
//
// Licensed under the Apache License, Version 2.0 (the "License");
// ...
```

### Package Comments

Every package has a doc comment on the `package` line or in a `doc.go`. Use `//` style, not `/* */`. Describe what the package does and any key design decisions:

```go
// Package archivedb provides SQLite-backed persistence for the Pixe archive
// database. It replaces the earlier JSON manifest with a cumulative registry
// that tracks all files ever sorted into a destination archive across all runs.
package archivedb
```

### Imports

Three groups separated by blank lines: stdlib, external, internal. Sorted alphabetically within each group. Use named imports only when there's a collision:

```go
import (
    "fmt"
    "os"
    "path/filepath"

    "github.com/spf13/cobra"
    "github.com/spf13/viper"

    "github.com/cwlls/pixe/internal/archivedb"
    copypkg "github.com/cwlls/pixe/internal/copy"    // named: avoids collision with builtin
    jpeghandler "github.com/cwlls/pixe/internal/handler/jpeg"  // named: handler packages share type names
)
```

Handler packages use the convention `<format>handler` for their import alias (e.g., `jpeghandler`, `heichandler`, `dnghandler`).

### Naming

- **Packages:** lowercase, single word when possible (`archivedb`, `discovery`, `pathbuilder`).
- **Exported types:** `PascalCase`. Structs use noun names (`Registry`, `Hasher`, `DB`).
- **Constructors:** `New()` returns the primary type. `NewFoo()` if the package has multiple types.
- **Methods:** receiver name is 1–2 letters derived from the type (`db`, `h`, `r`, `m`, `sw`).
- **Constants:** `PascalCase` for exported, `camelCase` for unexported. Status enums use `Status` prefix: `StatusPending`, `StatusComplete`, `StatusFailed`.
- **Test helpers:** unexported, use `t.Helper()`. Common pattern: `openTestDB(t)`, `copyFixture(t, dir, name)`, `writeFile(t, path, content)`.

### Types & Structs

- Use `*string`, `*time.Time`, `*int64` for nullable DB fields (maps to `sql.NullString` etc. in scan).
- Config structs (`AppConfig`, `SortOptions`) are passed by pointer.
- Domain types live in `internal/domain/` — shared across packages without import cycles.
- The `FileTypeHandler` interface is the extension point for new formats (Section 6 of the architecture).

### Error Handling

- Wrap errors with `fmt.Errorf("context: %w", err)` — always include the package or function as context prefix.
- Use `errors.Is()` for sentinel checks (e.g., `errors.Is(err, os.ErrNotExist)`).
- DB scan errors: return `(nil, nil)` for `sql.ErrNoRows` when "not found" is a valid result.
- Non-fatal errors (e.g., tag failure): log a warning to `out` and continue. Do not return an error.
- Fatal errors: return the error up the call chain. The CLI layer (`cmd/`) prints it and exits.

```go
// Pattern: wrap with package prefix
return nil, fmt.Errorf("archivedb: insert run: %w", err)

// Pattern: nil-nil for not-found
if errors.Is(err, sql.ErrNoRows) {
    return nil, nil
}
```

### Testing

- Tests use the stdlib `testing` package only — no testify or other assertion libraries.
- Test files are in the same package (white-box testing): `package pipeline` not `package pipeline_test`.
- Use `t.TempDir()` for filesystem tests — auto-cleaned.
- Use `t.Helper()` on all helper functions.
- Test names: `TestTypeName_methodOrBehavior` (e.g., `TestRun_basicSort`, `TestCheckDuplicate_notFound`).
- Table-driven tests use anonymous struct slices with `want`/`got` pattern.
- Integration tests live in `internal/integration/` and are excluded from `make test`.
- Locale-sensitive tests pin locale via `TestMain`: `pathbuilder.SetLocaleForTesting(language.English)`.

### SQL

- SQL strings use backtick-quoted `const` blocks with the query name as the const name.
- Use `?` placeholders (SQLite positional params).
- All DDL uses `IF NOT EXISTS` for idempotency.
- Schema migrations are additive (`ALTER TABLE ADD COLUMN`), never destructive.

### Concurrency

- Worker pool pattern: coordinator goroutine owns DB writes; workers own I/O.
- `syncWriter` wraps `io.Writer` with a mutex for concurrent stdout access.
- Per-worker assignment channels (buffered 1) for coordinator→worker communication.
- `context.Context` threaded through concurrent paths for cancellation.

## Project Layout

```
cmd/                    CLI commands (Cobra). No business logic here.
internal/
  archivedb/            SQLite database (schema, CRUD, queries)
  benchmark/            Centralized benchmark suite (hash, copy, db, discovery, pathbuilder)
  cli/                  Bubble Tea progress bar model and Lip Gloss styles
  config/               AppConfig struct (populated by cmd/, read by pipeline/)
  copy/                 File copy + verify
  dblocator/            Database path resolution (local vs network mount)
  discovery/            File walking + handler registry
  docgen/               Development-time doc generation tool (marker-based injection)
  domain/               Shared types (FileStatus, Ledger, Manifest, FileTypeHandler)
  fileutil/             Shared file-path utilities (extension normalization)
  handler/              FileTypeHandler implementations (jpeg/, heic/, mp4/, tiffraw/, dng/, ...)
  hash/                 Configurable hasher (MD5, SHA-1, SHA-256, BLAKE3, xxHash)
  ignore/               Glob-based file ignore matching with .pixeignore support
  integration/          End-to-end tests (excluded from `make test`)
  manifest/             JSON ledger/manifest persistence
  migrate/              Legacy JSON manifest → SQLite migration
  pathbuilder/          Destination path construction (date-based, locale-aware)
  pipeline/             Sort orchestrator (sequential + concurrent)
  progress/             Pipeline event bus (typed events, non-blocking channel, plain text writer)
  tagging/              Metadata tag injection (dispatch via handler capability)
  verify/               Post-sort verification
  xmp/                  XMP sidecar file generation (Adobe-compatible)
```

## Key Architectural Rules

1. **`dirA` is read-only.** Only `.pixe_ledger.json` is written there. Never modify source files.
2. **No external binaries.** No `os/exec`. All parsing is pure Go.
3. **Copy-then-verify.** Every file is re-hashed after copy. Mismatches are flagged, never silent.
4. **No Viper below `cmd/`.** Only `cmd/` reads Viper. Everything else receives `*config.AppConfig`.
5. **Version from git tags.** No version literals in source. Injected via ldflags at build time.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tholent)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tholent)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
