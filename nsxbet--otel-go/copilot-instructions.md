## otel-go

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

otel-go is an opinionated OpenTelemetry SDK wrapper for Go applications. The main package is `otelx` which provides simple configuration for traces, metrics, and logs with production-ready defaults.

## Common Commands

```bash
# Run tests
make test

# Run tests with verbose output
go test ./otelx/... -v

# Run a single test
go test ./otelx/... -run TestFunctionName -v

# Run linting
make lint

# Run tests and lint together
go test ./otelx/... && make lint

# View test coverage
make coverage-html
```

## Architecture

### Core Package Structure (`otelx/`)

- **otelx.go** - Main entry point with `ConfigureOpentelemetry()`, `Shutdown()`, `ForceFlush()`, and provider accessors
- **config.go** - Configuration struct and all `With*` option functions (functional options pattern)
- **tracing.go** - TracerProvider setup, sampler configuration, propagator setup
- **metrics.go** - MeterProvider setup with OTLP exporter and runtime metrics
- **logging.go** - LoggerProvider setup with OTLP exporter
- **exporter.go** - OTLP exporter creation for gRPC and HTTP protocols (traces, metrics, logs)
- **propagator_istio.go** - Custom propagator for Istio `x-request-id` header
- **client.go** - Internal client struct holding provider references

### Key Design Patterns

1. **Functional Options**: All configuration uses the `With*` pattern (e.g., `WithEndpoint()`, `WithSampleRatio()`)

2. **Atomic Client**: Global state managed via `atomic.Value` for thread-safe provider access

3. **Signal Independence**: Each signal (traces, metrics, logs) can be enabled/disabled independently

4. **Environment Variable Support**: Standard OTEL env vars are respected (OTEL_SERVICE_NAME, OTEL_EXPORTER_OTLP_ENDPOINT, etc.)

### Default Behaviors

- Protocol: gRPC
- Tracing: Enabled with ParentBased(TraceIDRatioBased(1.0)) sampler
- Metrics: Enabled with 15s export interval
- Logging: Disabled
- Compression: Enabled (gzip)
- Retry: Enabled with exponential backoff
- Istio propagation: Enabled (B3 + x-request-id)

## Testing

Tests use testify for assertions. Mock OTLP servers are created in `exporter_test.go` for integration testing. Use `resetGlobalState(t)` helper in tests that modify global OpenTelemetry state.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/NSXBet) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
