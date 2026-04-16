## common

> This file provides Gemini with a comprehensive guide to the `github.com/patdeg/common` repository. Adhere to the principles and guidelines outlined here when modifying the codebase.

# GEMINI.md - Development Guide for the 'common' Go Library

This file provides Gemini with a comprehensive guide to the `github.com/patdeg/common` repository. Adhere to the principles and guidelines outlined here when modifying the codebase.

## ⚠️ CRITICAL SECURITY NOTICE

**THIS IS A PUBLIC REPOSITORY.** You must never commit:
- API keys, tokens, or secrets. Use environment variables.
- Personal information (PII), including email addresses (except for `example.com`).
- Credentials of any kind.
- Customer data or real user information.
- Internal URLs or sensitive endpoints.

Always use placeholders for sensitive data and ensure it is loaded from the environment.

## Project Overview

The `common` package is a shared Go library providing a collection of reusable utility packages for building scalable web applications. It serves as a centralized repository for common functionality to avoid code duplication and ensure consistency across multiple projects.

### Core Principles
1.  **Security First**: This is a public repository; no sensitive data should ever be committed.
2.  **Environment-Aware**: The code should adapt to the environment (development vs. production), using local fakes or cloud services accordingly.
3.  **Backward Compatibility**: Changes should not break existing APIs.
4.  **Comprehensive Testing**: All packages must have robust tests.
5.  **Clear Documentation**: All exported functions and types must be documented.

## Package Organization

The library is organized into packages based on functionality:

*   **Core Utilities (`/`):** `common`, `convert`, `crypt`, `file`, `slice`, `url`. These provide foundational helpers for logging, type conversion, and more.
*   **Web & API (`/`):** `web`, `cookie`, `csrf`. These handle HTTP-level concerns like middleware, security, and bot detection.
*   **GCP Integration (`/gcp`, `/datastore`, `/bigquery`):** A repository pattern for Datastore, BigQuery utilities, and other Google Cloud Platform helpers.
*   **Application Features (`/auth`, `/email`, `/payment`, `/search`, `/monitor`):** Higher-level features like authentication, email, payments, search, and monitoring.
*   **Analytics (`/ga`, `/track`):** Google Analytics and custom event tracking.

## Building and Running

### Installation

As a library, it can be added to a Go project using:
```bash
go get github.com/patdeg/common
```

### Testing Commands

Use the following commands to test the codebase:

```bash
# Run all tests
go test ./...

# Run tests with race detection (important for concurrent code)
go test -race ./...

# Run tests with code coverage
go test -cover ./...

# Generate a coverage report
go test -coverprofile=coverage.out ./...
go tool cover -html=coverage.out
```

### Running Examples

The `examples/` directory contains runnable examples:
```bash
# Example for basic usage
cd examples/basic-usage && go run main.go

# Example for a full App Engine application
cd examples/appengine && go run main.go
```

## Key Development Guidelines

### Code Style
- Follow standard Go formatting (`gofmt`).
- Code should be self-documenting, with clear names for variables, functions, and types.
- All exported symbols must have doc comments.

### Error Handling
- Always return errors; do not panic.
- Wrap errors with `fmt.Errorf("context: %w", err)` to provide context.
- Use `errors.Is` and `errors.As` for checking specific error types.
- Log errors using the PII-safe logging functions.

### Environment Variables
Never hardcode configuration or secrets. Load them from the environment.

```go
// Good: Load from environment with a clear error if missing.
apiKey := os.Getenv("SENDGRID_API_KEY")
if apiKey == "" {
    return nil, errors.New("SENDGRID_API_KEY is not set")
}

// Bad: Hardcoded secret. NEVER do this.
apiKey := "sk_live_abc123"
```

### Dependencies
- Minimize external dependencies.
- Group imports into three blocks: standard library, third-party, and internal packages.
- Keep the dependency graph clean. Lower-level packages (`convert`, `slice`) should not depend on higher-level packages (`auth`, `payment`).

## Security Checklist

Before committing any changes, verify:
- [ ] No hardcoded credentials, keys, or secrets are present.
- [ ] No real user data or PII is included.
- [ ] All sensitive data is loaded via environment variables.
- [ ] Input validation is performed where necessary.
- [ ] Security best practices (like those in the `web` package's middleware) are followed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/patdeg) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
