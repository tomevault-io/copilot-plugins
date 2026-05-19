## quiqupgo

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Quiqupgo is a collection of reusable [uber/fx](https://github.com/uber-go/fx) modules for Go microservices. Each module follows a consistent pattern: Config interface + Module() function + testutil helpers.

## Common Commands

```bash
# Install tools (via asdf)
asdf install

# Run all tests
task test:unit

# Run a specific module's tests
task test:module MODULE=tracing

# Lint
task tools:lint

# Lint with auto-fix
task tools:lint-fix

# Format
task tools:fmt

# Full verification (tidy, fmt, lint, test, build)
task verify

# Integration tests (requires docker:up first)
task docker:up
task docker:status    # wait for healthy
task test:integration
task docker:down
```

## Architecture

### Module Pattern

Each fx module follows this structure:
- `config.go` - Config interface + StandardConfig implementation
- `module.go` - `Module()` returns `fx.Option` with providers and lifecycle hooks
- `*_test.go` - Unit tests
- `testutil/` - Test helpers (NoopModule, MockModule, BufferLogger, etc.)
- `doc.go` - Package documentation

### Module Dependencies

```
tracing.Module() → provides: TracerProvider, Tracer, MeterProvider, Meter
    ↓
logger.Module() → provides: *zap.Logger, Logger interface
    ↓
temporal.Module() → requires: *zap.Logger, Tracer → provides: client.Client
gormfx.Module()   → requires: TracerProvider → provides: *gorm.DB
kafka.Module()   → requires: *zap.Logger, Tracer → provides: Producer, Consumer
```

### Config Interface Pattern

Each module defines a Config interface (e.g., `tracing.Config`, `logger.Config`). Applications implement this interface or use the provided `StandardConfig`. Use `fx.Annotate` with `fx.As` to adapt app configs:

```go
fx.Provide(fx.Annotate(newTracingConfig, fx.As(new(tracing.Config))))
```

### Testing Pattern

Use `testutil` packages for testing:
```go
tracingtest.NoopModule()   // No-op tracing
loggertest.BufferModule()  // Captures log entries
temporaltest.MockModule()  // Mock temporal client
```

## Git Flow

- **Feature branches** (`feat/*`): Update `CHANGELOG.md` under the `[Unreleased]` section before merging.
- **Release branches** (`rel/*`): Promote `[Unreleased]` changes in `CHANGELOG.md` to a versioned section (e.g., `[v1.2.0]`) before closing.
- **Merge method**: Always use `rebase` when merging PRs.

## Code Style

- Uses golangci-lint with revive, gocritic, gosec, and other linters
- goimports for import ordering
- Pre-commit hook available via `task tools:hooks`
- Tests use `testify/assert` and `testify/require`

---
> Source: [quiqupltd/quiqupgo](https://github.com/quiqupltd/quiqupgo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
