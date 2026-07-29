## bmad-automated

> **Analysis Date:** 2026-01-08

# Coding Conventions

**Analysis Date:** 2026-01-08

## Naming Patterns

**Files:**

- snake_case for all Go files (`create_story.go`, `client_test.go`)
- `*_test.go` co-located with source files
- `types.go` for type definitions within a package
- `errors.go` for error types within a package

**Functions:**

- PascalCase for exported functions (`NewExecutor`, `RunSingle`)
- camelCase for unexported functions (`writeln`, `handleEvent`)
- Constructors: `New*` prefix (`NewParser`, `NewPrinter`)

**Variables:**

- camelCase for local variables
- PascalCase for exported package-level vars
- Short receiver names (`e` for executor, `p` for printer)

**Types:**

- PascalCase for all types (`StreamEvent`, `Executor`)
- No prefix for interfaces (not `IExecutor`, just `Executor`)
- `*Config` suffix for configuration structs
- `*Result` suffix for result types

**Constants:**

- PascalCase for exported constants
- Event type constants in `types.go` (`EventTypeSystem`, `EventTypeAssistant`)

## Code Style

**Formatting:**

- `gofmt` with simplification (`gofmt -s`)
- `goimports` for import organization
- Tab indentation (Go standard)
- Config: `.editorconfig` and `.golangci.yml`

**Linting:**

- golangci-lint with `.golangci.yml` config
- Timeout: 5 minutes
- Enabled linters: errcheck, gosimple, govet, ineffassign, staticcheck, unused, gofmt, goimports, misspell, unconvert, unparam, revive
- Run: `just lint`

## Import Organization

**Order:**

1. Standard library (`context`, `fmt`, `io`)
2. External packages (`github.com/spf13/cobra`)
3. Internal packages (`bmad-automate/internal/claude`)

**Grouping:**

- Blank line between groups
- `goimports` handles sorting automatically

**Path Aliases:**

- None - use full import paths

## Error Handling

**Patterns:**

- Return errors, don't panic (except truly unrecoverable)
- Wrap errors with context: `fmt.Errorf("failed to X: %w", err)`
- CLI uses `ExitError` type for testable exit codes

**Error Types:**

- Custom `ExitError` in `internal/cli/errors.go`
- Standard `error` interface elsewhere
- Check with `if err != nil` immediately after call

**Exit Codes:**

- 0: Success
- 1: General error (config, unknown command)
- Non-zero from Claude subprocess passed through

## Logging

**Framework:**

- Console output via `Printer` interface
- No structured logging (CLI tool)

**Patterns:**

- User-facing output through `Printer` methods
- Debug output not currently implemented
- Stderr passed through from Claude subprocess

## Comments

**When to Comment:**

- Package-level documentation (required by linter)
- Exported functions and types (required by linter)
- Non-obvious implementation decisions

**JSDoc/TSDoc:**

- N/A (Go uses standard doc comments)

**Format:**

```go
// Package claude provides types and functionality for interacting with the Claude CLI.
package claude

// Executor runs Claude CLI and returns streaming events.
type Executor interface {
    // Execute runs Claude with the given prompt and returns a channel of events.
    Execute(ctx context.Context, prompt string) (<-chan Event, error)
}
```

**TODO Comments:**

- Format: `// TODO: description`
- Not currently used in codebase

## Function Design

**Size:**

- Functions generally under 50 lines
- Extract helpers for complex logic

**Parameters:**

- Use `context.Context` as first parameter where applicable
- Use options struct for 4+ parameters (`ExecutorConfig`)
- Interfaces for dependencies (enables mocking)

**Return Values:**

- Return error as last value
- Use named returns sparingly
- Return early for error cases

## Module Design

**Exports:**

- Named exports only (Go doesn't have default exports)
- Export interfaces, not implementations when possible
- Keep internal helpers unexported

**Package Structure:**

- One clear purpose per package
- `internal/` for private packages
- Avoid circular dependencies

**Interface Location:**

- Interfaces defined in the package that uses them
- `Executor` interface in `internal/claude/client.go`
- `Printer` interface in `internal/output/printer.go`

## Testing Conventions

**Test Files:**

- Co-located: `foo.go` → `foo_test.go`
- Same package (white-box testing)

**Test Naming:**

- `Test<Type>_<Method>` or `Test<Function>`
- Example: `TestMockExecutor_Execute`, `TestDefaultParser_Parse`

**Assertions:**

- Use `github.com/stretchr/testify/assert`
- Use `require` for fatal assertions

**Mocking:**

- Mock implementations in production code (not separate file)
- `MockExecutor` in `internal/claude/client.go`
- `NewPrinterWithWriter()` for output capture

---

_Convention analysis: 2026-01-08_
_Update when patterns change_

---
> Source: [robertguss/bmad_automated](https://github.com/robertguss/bmad_automated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
