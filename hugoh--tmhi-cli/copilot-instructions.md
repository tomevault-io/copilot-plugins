## tmhi-cli

> This project uses `mise` for task management. Use the following Mise tasks for all build, test, and lint operations:

# Development Instructions

## Build & Development Tasks

This project uses `mise` for task management. Use the following Mise tasks for all build, test, and lint operations:

### Building

```bash
mise build    # Build application binary into dist/
mise dev      # Run in development mode (go run .)
```

### Testing

```bash
mise test     # Run tests with coverage using gotestsum
mise coverage # Check coverage meets threshold
```

Individual tasks (`mise lint`, `mise format`, `mise fix`, `hk check --all`) are all wrapped by `mise full-check` / `mise ci`.

### Module Maintenance

```bash
mise tidy  # Tidy Go module (go mod tidy -v)
mise depup # Upgrade dependencies
mise clean # Clean build artifacts
```

### CI

```bash
mise full-check # Run full pre-commit validation (format, fix, core CI including tests) during development
mise ci         # Full CI pipeline (lint, tests, coverage, build) run in the pipeline
hk check        # Run hooks
```

## Agent Workflow

When making changes to this codebase:

1. **Before editing**: Run `mise full-check` to establish a clean baseline
2. **After editing**: Run `mise full-check` — auto-formats, auto-fixes lint, runs tests+coverage+integration
3. **Before completion**: Run `mise ci` for the full CI pipeline (fresh lint from scratch)

Always use `mise` tasks rather than calling tools directly

## Test Coverage

Test coverage threshold is configured in `.testcoverage.yml`.

---
> Source: [hugoh/tmhi-cli](https://github.com/hugoh/tmhi-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
