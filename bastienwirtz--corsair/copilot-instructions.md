## corsair

> Corsair is a configurable HTTP CORS proxy server that dynamically exposes endpoints based on YAML configuration. It features structured logging, template-based configuration, and modular architecture.

# AGENTS Instructions

## Project Overview

Corsair is a configurable HTTP CORS proxy server that dynamically exposes endpoints based on YAML configuration. It features structured logging, template-based configuration, and modular architecture.

## Development Commands

### Build and Run

```bash
go build -o corsair ./cmd          # Build the binary
./corsair                          # Run with config.yaml
./corsair -c custom-config.yaml    # Run with custom config file
```

### Testing

```bash
go test ./...                      # Run all tests
go test ./... -v                   # Run with verbose output
go test ./config -v                # Run tests for specific package
go test ./middleware -run TestCORS # Run specific test
```

### Dependencies

```bash
go mod tidy                        # Update dependencies
go mod download                    # Download dependencies
```

## Architecture

### Core Components

**Configuration System**: The `config` package handles YAML parsing, validation, and template processing. It includes a regex-based template engine that substitutes `{{ VARIABLE_NAME }}` patterns with environment variables at runtime.

**Server Layer**: The `server` package provides the HTTP server implementation with structured logging setup and route registration.

**Request Flow**:

1. **Configured endpoints**: Requests to configured paths are proxied to `remote_url` with path preservation, custom headers, and query parameters
2. **Generic forwarding**: The `/forward?url=<target>` endpoint allows ad-hoc proxying with URL validation and security checks
3. **CORS handling**: All requests pass through CORS middleware that applies global rules and handles preflight OPTIONS requests
4. **Logging**: Structured logging using Go's slog package with configurable levels and formats

**Middleware Architecture**: Uses function composition for middleware (no external framework). CORS middleware wraps all handlers.

### Package Structure

- `cmd/`: Application entry point with server lifecycle management
- `config/`: Configuration parsing, validation, template processing, and data structures
- `handlers/`: HTTP handlers for proxy endpoints and generic forwarding
- `middleware/`: CORS handling with origin validation and wildcard support
- `server/`: HTTP server setup, route registration, and structured logging configuration

### Key Design Patterns

**Template Processing**: Runtime environment variable substitution in headers, query parameters, and remote URLs using regex-based template engine with `{{ VAR_NAME }}` syntax.

**Structured Logging**: Configurable logging with multiple formats (json, text, pretty) and levels (debug, info, warn, error) using Go's slog package and the tint library for colored output.

**Modular Server Design**: Server struct encapsulating ServeMux and configuration with clean separation of concerns.

## Configuration Structure

The server expects a `config.yaml` with four main sections:

- `server`: Port configuration (defaults to 8080)
- `logging`: Log level and format configuration (level: debug/info/warn/error, format: json/text/pretty)
- `cors`: Global CORS policy (origins, methods, headers, credentials)
- `endpoints`: Array of proxy endpoint definitions with path, remote_url, headers, and query_params

Template variables use `{{ ENV_VAR }}` syntax and are resolved from environment variables at request time, enabling secure secret injection without storing them in configuration files.

## Dependencies

Key external dependencies:
- `github.com/goccy/go-yaml`: YAML parsing and marshaling
- `github.com/stretchr/testify`: Testing framework with assertions
- `github.com/lmittmann/tint`: Pretty-printed structured logging
- `github.com/mattn/go-isatty`: Terminal detection for colored output

## Testing Strategy

The test suite uses testify and covers:

- Configuration parsing and validation with defaults
- Template variable substitution with various patterns
- CORS origin matching including wildcard patterns
- HTTP request forwarding with proper error handling

When adding new features, follow the existing pattern of comprehensive table-driven tests with both success and failure scenarios.

---
> Source: [bastienwirtz/corsair](https://github.com/bastienwirtz/corsair) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
