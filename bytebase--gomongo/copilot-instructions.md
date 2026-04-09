## gomongo

> This file provides guidance to AI coding assistants when working with code in this repository.

This file provides guidance to AI coding assistants when working with code in this repository.

## Project Overview

gomongo is a Go library that parses MongoDB shell syntax and executes commands using the native Go MongoDB driver. It uses the ANTLR-based parser from `github.com/bytebase/parser/mongodb`.

## Project Structure

```
gomongo/
├── client.go              # Public API: Client, NewClient, Execute
├── executor.go            # Parse → Translate → Execute pipeline
├── translator.go          # Walk ANTLR parse tree, build driver operations
├── method_registry.go     # Registry of MongoDB methods with status and hints
├── helper_functions.go    # Convert ObjectId(), ISODate(), etc. to BSON
├── errors.go              # Error types (ParseError, UnsupportedOperationError, DeprecatedOperationError)
├── executor_test.go       # Integration tests with testcontainers
└── go.mod
```

## Development Workflow

**ALWAYS follow these steps after making code changes:**

1. **Format** — Run `gofmt -w` on modified files
2. **Lint** — Run `golangci-lint run --allow-parallel-runners` to catch issues
   - Run repeatedly until there are no issues (linter has max-issues limit)
3. **Auto-fix** — Use `golangci-lint run --fix --allow-parallel-runners` to fix issues automatically
4. **Test** — Run `go test -v ./...` before committing
5. **Build** — Run `go build ./...` to verify compilation

## Build/Test Commands

```bash
# Build
go build ./...

# Run all tests
go test -v ./...

# Run single test
go test -v -count=1 -run ^TestFunctionName$

# Run tests with race detection
go test -race -v ./...

# Lint
golangci-lint run --allow-parallel-runners

# Format
gofmt -w .
```

## Code Style

### General

- Follow [Google Go Style Guide](https://google.github.io/styleguide/go/)
- Write clean, minimal code; fewer lines is better
- Prioritize simplicity for effective and maintainable software
- Only include comments that are essential to understanding functionality

### Go

- Use standard Go error handling with detailed error messages
- Always use `defer` for resource cleanup like `cursor.Close()`
- Avoid using `defer` inside loops — use IIFE or scope properly
- Use `any` instead of `interface{}` (Go 1.18+)

### Naming

- Use American English
- Avoid plurals like "xxxList"

### Imports

- Use organized imports (sorted by import path)
- Group: stdlib, external, internal

### Error Handling

- Be explicit but concise about error cases
- Wrap errors with context using `errors.Wrap()` or `fmt.Errorf("...: %w", err)`

## Common Go Lint Rules

- **Unused Parameters** — Prefix unused parameters with underscore (e.g., `func foo(_ *Bar)`)
- **Modern Go Conventions** — Use `any` instead of `interface{}`
- **Confusing Naming** — Avoid similar names that differ only by capitalization
- **Identical Branches** — Don't use if-else branches that contain identical code
- **Function Receivers** — Don't create unnecessary function receivers
- **Export Rules** — Only export functions and types that need to be used outside the package

## Testing

### Unit Tests

- Test translator logic with mock parse trees
- Test helper function conversions (ObjectId, ISODate, etc.)
- Test error message formatting

### Integration Tests

Use testcontainers for MongoDB:

```go
func TestFind(t *testing.T) {
    // Start MongoDB container
    // Insert test documents
    // Execute query via gomongo
    // Verify results
}
```

## Dependencies

- `go.mongodb.org/mongo-driver/v2` — Official MongoDB Go driver
- `github.com/bytebase/parser` — ANTLR-based MongoDB shell parser
- `github.com/antlr4-go/antlr/v4` — ANTLR runtime for Go

## Output Format

All query results are returned as Extended JSON (Relaxed) format using `bson.MarshalExtJSONIndent()`. This ensures:
- Valid JSON that can be parsed by `JSON.parse()`
- Type information preserved for BSON types
- Consistent output format

## Pull Request Guidelines

- **Description** — Clearly describe what the PR changes and why
- **Testing** — Include information about how the changes were tested
- **Breaking Changes** — Clearly mark any breaking API changes

## Adding New Method Support

When adding support for a new MongoDB method:

1. **Update `method_registry.go`** — Remove the method entry or change status to `statusSupported`
2. **Update `translator.go`** — Add handler in `visitMethodCall()` for the new method
3. **Add tests** — Create integration tests in `executor_test.go`
4. **Update README** — Add the method to the supported methods list

### Method Registry

The `method_registry.go` file maintains metadata about MongoDB methods:

- **statusSupported** — Method is implemented and working
- **statusDeprecated** — Method is deprecated; error includes alternative suggestion
- **statusUnsupported** — Method is recognized but not yet implemented

When implementing a previously unsupported method, remove its entry from the registry (supported methods don't need registry entries).

### Error Types

- **UnsupportedOperationError** — For methods not yet implemented (includes hint)
- **DeprecatedOperationError** — For deprecated methods (includes alternative)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bytebase)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bytebase)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
