## bobn

> This is a Golang cryptocurrency trading terminal application built with Bubble Tea TUI framework.

# Project Context for Claude

## Project Overview
This is a Golang cryptocurrency trading terminal application built with Bubble Tea TUI framework.

## Tech Stack
- Language: Go
- Build Tool: go build
- Testing: go test
- Linting: golangci-lint (if available)

## Project Structure
```
.
├── cmd/           # Application entrypoints
├── internal/      # Private application code
├── pkg/           # Public libraries
├── api/           # API definitions (proto, openapi, etc)
├── configs/       # Configuration files
├── scripts/       # Build and deployment scripts
├── test/          # Additional test files
└── vendor/        # Vendored dependencies (if using vendor mode)
```

## Development Commands
```bash
# Build the application
go build ./...

# Run tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Format code
go fmt ./...

# Vet code
go vet ./...

# Install dependencies
go mod download

# Tidy dependencies
go mod tidy

# Run linter (if golangci-lint is installed)
golangci-lint run
```

## Testing Strategy
- Unit tests are colocated with source files (e.g., `file.go` and `file_test.go`)
- Use table-driven tests where appropriate
- Mock external dependencies using interfaces
- Aim for high test coverage on business logic

## Code Style Guidelines
- Follow standard Go conventions and idioms
- Use `gofmt` for formatting
- Prefer clear names over comments
- Keep functions small and focused
- Handle errors explicitly
- Use interfaces for abstraction
- Prefer composition over inheritance

## Common Patterns
- Use context.Context for cancellation and timeouts
- Return errors as the last value
- Use defer for cleanup
- Initialize structs with named fields
- Use channels for concurrent communication
- Implement graceful shutdown

## Dependencies
- Manage dependencies with Go modules (go.mod)
- Keep third-party dependencies minimal
- Prefer standard library when possible

## Important Notes
- Always run `go fmt` before committing
- Ensure all tests pass before pushing
- Update go.mod when adding new dependencies
- Follow semantic versioning for releases

---
> Source: [jonasrmichel/bobn](https://github.com/jonasrmichel/bobn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
