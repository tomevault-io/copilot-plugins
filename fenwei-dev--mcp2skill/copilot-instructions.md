## mcp2skill

> Instructions for AI coding assistants working in this repository.

# AGENTS.md

Instructions for AI coding assistants working in this repository.

## Project Overview

mcp2skill is a Go CLI tool that converts MCP (Model Context Protocol) servers into agent skills. It generates skill packages with SKILL.md documentation from MCP server metadata.

## Build, Lint, and Test Commands

```bash
# Build
go build ./cmd/mcp2skill                              # Standard build
CGO_ENABLED=0 go build -ldflags="-s -w" ./cmd/mcp2skill  # Static binary for distribution

# Run all tests
go test ./...

# Run tests with coverage
go test -cover ./...

# Run a single test file
go test -v ./internal/config

# Run a single test function
go test -v -run TestFunctionName ./internal/config

# Run tests in specific package with timeout
go test -timeout 30s ./internal/mcp

# Lint (if golangci-lint is installed)
golangci-lint run ./...
golangci-lint run --new-from-rev HEAD~1  # Lint only changes
```

## Code Style Guidelines

### Formatting
- Use `gofmt` (or `go fmt`) to format code before committing
- Run `go vet` to catch common mistakes
- Configure editor to format on save with gofmt

### Imports
- Use standard Go import organization with `go fmt`/`goimports`
- Group imports: standard library first, then third-party, then internal
- Use dot imports sparingly (only in test files when helpful)

### Types and Interfaces
- Use struct types for data containers with JSON tags for config files
- Define interfaces for external dependencies (e.g., MCP client) to enable testing
- Prefer concrete types over interfaces unless mocking is needed
- Use type aliases for domain-specific strings (e.g., `type TransportType string`)

### Naming Conventions
- **Packages**: lowercase, short, descriptive (e.g., `config`, `mcp`, `cliapp`)
- **Exported identifiers**: PascalCase (e.g., `LoadEffective`, `ServerConfig`)
- **Unexported identifiers**: camelCase (e.g., `loadEffective`, `serverCfg`)
- **Variables**: concise but descriptive; avoid single letters except loop indices
- **Constants**: SNAKE_CASE for grouped constants, PascalCase for exported enums
- **Acronyms**: Use consistent casing (e.g., `ServeMux`, not `ServeMIX` or `ServeMuxID`)

### Error Handling
- Return errors early with `fmt.Errorf("context: %w", err)` for wrapping
- Handle errors at the appropriate level (don't ignore unless explicitly documented)
- Use `os.IsNotExist(err)` for file-not-found checks
- Prefix error messages with lowercase for consistency
- Use `%+v` in debug contexts to show struct fields

### Functions and Methods
- Keep functions focused: single responsibility, <50 lines preferred
- Use receiver methods for operations on config structs
- Document exported functions with doc comments (will appear in godoc)
- Prefer named return values for documentation clarity

### Concurrency
- Pass context as first parameter: `func(ctx context.Context, ...)`
- Use `context.Context` for cancellation and timeouts
- Handle goroutine leaks: ensure all goroutines can exit

### CLI Conventions (urfave/cli/v3)
- Use `&cli.Command` for subcommands with kebab-case names
- Use `&cli.StringFlag`, `&cli.BoolFlag` for options
- Define required flags with `Required: true`
- Provide `Usage` field for help text

### Project Structure
```
cmd/mcp2skill/           # Main entry point
internal/
  cliapp/                # CLI command implementations
  config/                # Configuration loading/merging
  mcp/                   # MCP client wrapper
  skill/                 # Skill generation logic
```

## OpenSpec Workflow

For new features, breaking changes, or architecture work:

1. Read `openspec/AGENTS.md` for the full spec-driven development workflow
2. Create a change proposal under `openspec/changes/<change-id>/`
3. Use the three-stage workflow: Create → Implement → Archive
4. Run `openspec validate <change-id> --strict` before requesting approval

Skip proposals for: bug fixes, typos, dependency updates, or tests for existing behavior.

## Testing Conventions

- Place tests in same package with `_test.go` suffix
- Use table-driven tests for functions with multiple test cases
- Name test functions: `TestFunctionName_ExpectedBehavior`
- Mock external dependencies via interfaces
- Use `t.Cleanup()` for cleanup logic

## Documentation

- Document public APIs with doc comments
- Use `// TODO:` comments for technical debt (with issue reference if exists)
- Update README.md when adding new commands or changing behavior
- Keep SKILL.md generation logic in sync with CLI help text

## Git Conventions

- Use conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`, `test:`
- Squash commits before merging PRs
- Write meaningful commit messages (not "wip" or "fix")
- Run tests before committing: `go test ./...`

## Useful References

- Go effective Go: https://go.dev/doc/effective_go
- Go naming conventions: https://golang.org/doc/naming
- urfave/cli v3 docs: https://github.com/urfave/cli/tree/main/v3

---
> Source: [fenwei-dev/mcp2skill](https://github.com/fenwei-dev/mcp2skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
