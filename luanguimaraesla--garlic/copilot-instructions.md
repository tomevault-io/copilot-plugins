## garlic

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Garlic is an internal Go framework providing standardized utilities for building Web APIs, libraries, workers, and service integrations. It is a **library** (no main entry point) — consumed by other Go projects.

## Common Commands

```bash
make test                        # Run all unit tests with coverage
make GOTESTRUN=TestName test     # Run a specific test by name
make lint                        # Run golangci-lint
make fix                         # Format code (goimports) + tidy/vendor modules
make build                       # Build binaries from cmd/ to build/
make cover                       # Show text coverage report (run after test)
make cover/html                  # Open HTML coverage report
make image                       # Build dev Docker image
```

## Architecture

The framework is organized as independent packages, each providing a specific concern:

- **errors/** — Rich error type (`ErrorT`) with kinds (User, Validation, System), propagation chains, reverse traces, stack traces, and HTTP status code mapping. Central to the framework — used by most other packages.
- **rest/** — Chi-based HTTP server with response helpers that integrate with the error system to produce appropriate HTTP responses.
- **middleware/** — HTTP middleware stack: logging, tracing (request/session IDs), OpenTelemetry monitoring, CORS, content-type enforcement, context cancellation.
- **request/** — Request parsing helpers (UUID/int from path, query params, JSON body) with error handling that produces user-friendly hints.
- **database/** — PostgreSQL abstraction over sqlx/pgx with connection management, transactions (rollback/commit), query filtering, patching, and mock support.
- **logging/** — Singleton Zap-based structured logger, initialized via config, injected into context by middleware.
- **monitoring/** — OpenTelemetry HTTP metrics recorded via the global MeterProvider: `http.server.requests`, `http.server.active_requests`, `http.server.request.duration`.
- **observability/** — Installs a MeterProvider that pushes metrics to an OTLP/gRPC collector (configured via OTEL env vars or `Config` overrides); `Shutdown()` flushes on exit.
- **validator/** — Singleton go-playground/validator with custom field validators and JSON tag-based field naming.
- **httpclient/** — HTTP client wrapper with backoff retry strategy for inter-service communication.
- **crypto/** — Encryption/decryption utilities with mock support.
- **tracing/** — Request/session ID generation and context propagation.
- **worker/** — Background worker pool with task queue.
- **test/** — Builder-pattern HTTP test case utilities using httptest and chi router context.

## Key Conventions

- **Build tags:** Unit tests use `//go:build unit`. The `make test` target passes `-tags=unit` automatically.
- **Singleton patterns:** logging, validator, and rest server use singleton/multiton initialization — call `Init()` or `GetServer()` before use.
- **Error propagation:** Use `errors.Propagate(err)` or `errors.PropagateAs(err, kind)` rather than wrapping errors manually. Error kinds map to HTTP status codes automatically.
- **Context injection:** Logger, tracing IDs, and other state flow through `context.Context` via middleware.
- **Module mode:** Build uses `-mod=mod` (set in GOFLAGS). Dependencies are vendored (`vendor/` directory).

## Agent Guidance

For comprehensive usage rules and code examples when using garlic in downstream
projects, see the skill at `.claude/skills/garlic-conventions/SKILL.md`.

---
> Source: [luanguimaraesla/garlic](https://github.com/luanguimaraesla/garlic) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
