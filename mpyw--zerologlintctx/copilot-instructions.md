## zerologlintctx

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**zerologlintctx** is a Go linter that enforces context propagation in [zerolog](https://pkg.go.dev/github.com/rs/zerolog) logging chains. It detects:

1. Event chains missing `.Ctx(ctx)` when context is available
2. Direct logging calls (`Print`, `Printf`, `Println`) that bypass the Event chain

See [Architecture](./docs/ARCHITECTURE.md) for internal design details.

## Development Commands

```bash
# Run ALL tests (ALWAYS use this before committing)
./test_all.sh

# Run tests
go test ./...

# Run tests with verbose output
go test -v ./...

# Build CLI
go build -o bin/zerologlintctx ./cmd/zerologlintctx

# Run linter on itself
go vet -vettool=./bin/zerologlintctx ./...

# Run golangci-lint
golangci-lint run ./...
```

> [!IMPORTANT]
> Always use `./test_all.sh` before committing. This runs all tests including linting.

## Testing Strategy

- Use `analysistest` for all analyzer tests
- Test fixtures use `// want` comments for expected diagnostics
- Test structure:
  - `===== SHOULD REPORT =====` - Cases that should trigger warnings
  - `===== SHOULD NOT REPORT =====` - Negative cases
  - `===== EDGE CASES =====` - Corner cases

### Testdata Organization

```
testdata/src/zerolog/
├── basic.go           # Simple good/bad cases, ignore directives
├── evil.go            # General edge cases (nesting, closures, conditionals)
├── evil_ssa.go        # SSA-specific patterns (IIFE, Phi, channels)
├── evil_logger.go     # Logger transformation patterns, direct logging
└── with_logger.go     # WithLogger-specific tests
```

## Code Style

- Follow standard Go conventions
- Use `go/analysis` framework
- Prefer type-based checks over name-based (see Architecture)
- Unexported types by default; only export what's needed

### Comment Guidelines

**Comments should inform newcomers, not document history.**

- Bad: `// moved from evil.go`
- Bad: `// refactored in session 5`
- Good: `// LIMITATION: cross-function tracking not supported`

## Key Design Decisions

1. **Type-safe analysis**: Uses return types (`returnsEvent`, `returnsLogger`, etc.) instead of method name hardcoding
2. **SSA tracing**: Strategy Pattern with three tracers (Event, Logger, Context)
3. **Zero false positives**: Prefer missing issues over false alarms

## Known Limitations

- **Helper function returns**: Can't track through interprocedural analysis (IIFE is supported)
- **Channel send/receive**: Can't trace through channels
- **Closure-modified capture**: Closure writes to outer variable

These are documented in test cases with `// LIMITATION` comments.

## Related Documentation

- [README.md](./README.md) - User documentation
- [Architecture](./docs/ARCHITECTURE.md) - Internal design and detection logic

---
> Source: [mpyw/zerologlintctx](https://github.com/mpyw/zerologlintctx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
