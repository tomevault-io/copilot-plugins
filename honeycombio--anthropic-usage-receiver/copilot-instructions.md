## anthropic-usage-receiver

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Common Development Commands

### Building
```bash
# Build the custom collector binary with Anthropic receiver
cd example && ./build.sh

# Build the receiver module itself
cd anthropicusagereceiver && go build -v ./...

# Build using Makefile
make build
```

### Testing
```bash
# Run all tests with race detection
make test
# or
go test -v -race ./...

# Run tests with coverage
make test-coverage
# or
go test -v -race -coverprofile=coverage.txt -covermode=atomic ./...

# Run a single test
cd anthropicusagereceiver && go test -v -run TestSpecificTest ./...
```

### Linting
```bash
# Run linter (golangci-lint v2.3.0 required)
make lint
# or
cd anthropicusagereceiver && golangci-lint run --config=../.golangci.yml ./...

# Format code
make fmt
# or
go fmt ./... && gofumpt -w .
```

### Running the Collector
```bash
# After building with example/build.sh
export ANTHROPIC_ADMIN_API_KEY="sk-ant-admin-..."
export OTLP_ENDPOINT="your-backend:4317"
./example/otelcol-anthropic/otelcol-anthropic --config example/otel-config.yaml

# Or using Makefile
make run
```

### Metadata Generation
```bash
# Generate code from metadata.yaml (after modifying it)
cd anthropicusagereceiver
go generate ./...
```

## High-Level Architecture

### Repository Structure
- **anthropicusagereceiver/**: Core receiver implementation as an OpenTelemetry Collector component
  - Uses mdatagen pattern with `metadata.yaml` defining metrics, logs, and resource attributes
  - Implements both metrics and logs pipelines for usage and cost data
- **example/**: Build tooling for creating a custom collector distribution
  - Uses OpenTelemetry Collector Builder (OCB) to bundle the receiver into a collector binary
  - Contains sample configurations

### Key Components

#### Receiver Factory Pattern
The receiver follows OpenTelemetry's factory pattern:
- `factory.go`: Creates receiver instances with default configurations
- `config.go`: Defines configuration structure and validation
- Supports both metrics and logs receivers from the same factory

#### Data Collection Architecture
Two distinct scrapers handle different data types:
- **Metrics Scraper** (`scraper.go`): 
  - Collects usage metrics (tokens, web searches) via Anthropic Admin API `/v1/admin/usage`
  - Aggregates by model, workspace, API key, and service tier
  - Uses configurable time buckets for aggregation
- **Logs Scraper** (`logs_scraper.go`):
  - Fetches cost data via `/v1/admin/costs` endpoint
  - Emits structured logs with cost breakdowns by workspace
  - Runs on independent schedule from metrics

#### Client Layer
- `internal/client/client.go`: HTTP client with authentication and retry logic
- Implements exponential backoff for resilience
- Handles Anthropic Admin API authentication

#### Code Generation
- Uses OpenTelemetry's mdatagen tool
- `metadata.yaml` defines all metrics, logs, and attributes
- Generated code in `internal/metadata/` provides type-safe metric builders
- Run `make generate` after modifying metadata.yaml to regenerate metadata files
- Note: `make generate` requires access to mdatagen tool from opentelemetry-collector-contrib

### Configuration Patterns
The receiver uses nested configuration with validation:
- Top-level admin API key and endpoint
- Separate `usage` and `cost` sections with independent settings
- Each section has `enabled`, `bucket_width`, `collection_interval`, and `group_by` options
- Configuration is validated at startup

### Testing Approach
- Unit tests for all major components
- Mock HTTP client for API testing
- Test data in `testdata/` directory
- GitHub Actions runs tests on Go 1.22 and 1.23

---
> Source: [honeycombio/anthropic-usage-receiver](https://github.com/honeycombio/anthropic-usage-receiver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
