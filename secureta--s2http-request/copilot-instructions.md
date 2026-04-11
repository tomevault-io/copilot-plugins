## s2http-request

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Commands

All project tasks must be executed using `go tool mage`:

- `go tool mage build` - Build s2req and s2req-schema binaries
- `go tool mage test` - Run all tests  
- `go tool mage testCoverage` - Run tests with HTML coverage report
- `go tool mage lint` - Run golangci-lint
- `go tool mage fmt` - Format all Go code
- `go tool mage security` - Run gosec security scanner
- `go tool mage schema` - Generate request-schema.json file
- `go tool mage install` - Install binaries to GOPATH/bin
- `go tool mage clean` - Remove build artifacts

To run a single test: `go test -v -run TestFunctionName ./path/to/package`

## Architecture Overview

**s2http-request** is a WAF testing tool that sends HTTP requests defined in pure JSON/YAML files with dynamic value generation via `$function` tags.

### Key Components

1. **cmd/s2req** - Main HTTP request dispatcher with subcommand architecture
   - Default mode: Execute HTTP requests and display responses
   - `generate` subcommand: Generate request files with variable expansion
   - `validate` subcommand: Validate request definition files
2. **cmd/s2req-schema** - JSON schema generator for request validation
3. **internal/parser** - Parses and processes request definitions with function evaluation
4. **internal/http** - HTTP client that builds and sends requests
5. **pkg/functions/registry.go** - Central registry for all dynamic functions

### Subcommand Architecture

The tool uses a subcommand-based architecture:

- **Main command** (default): Execute HTTP requests from files
  ```bash
  s2req [options] <request-file>...
  ```

- **Generate subcommand**: Process request files with variable overrides and array expansion
  ```bash
  s2req generate [options] <request-file>...
  s2req generate --var 'ids=[1,2,3]' --format yaml request.json
  ```

- **Validate subcommand**: Validate request definition files
  ```bash
  s2req validate [options] <request-file>...
  echo '{"method":"GET","path":"/"}' | s2req validate
  ```

### Variable Override System

The `generate` subcommand supports variable overrides via `--var` flags:

- **CLI variables override file variables** with higher priority
- **Array expansion**: Arrays in variables create Cartesian product combinations
- **Type detection**: Automatically parses JSON values, numbers, booleans, and strings
- **Multiple formats**: Output in YAML (default), JSON, or JSONL

Example:
```bash
# Single variable override
s2req generate --var user_id=123 request.yaml

# Array expansion (generates multiple requests)
s2req generate --var 'ids=[1,2,3]' --var 'types=["admin","user"]' request.yaml

# JSON object variables
s2req generate --var 'config={"enabled":true,"timeout":30}' request.yaml
```

### Function System

Dynamic functions use `$function_name` tags in JSON/YAML:
- Variable: `$var`, `$concat`, `$join`
- Encoding: `$url_encode`, `$base64_encode`, `$html_encode`, `$hex_encode`
- Decoding: `$url_decode`, `$base64_decode`, `$html_decode`
- Random: `$random`, `$random_string`, `$uuid`
- Time: `$timestamp`, `$date`, `$time`
- File: `$file`
- Arrays: `$concat_arrays`
- Body formatting: `$form`, `$json`, `$multipart`

New functions must:
1. Be implemented in `pkg/functions/`
2. Be registered in `pkg/functions/registry.go`
3. Have comprehensive tests following TDD

## Development Requirements

### Test-Driven Development (TDD)

**MANDATORY**: Follow Takuto Wada's TDD methodology for all changes:

1. **Red** - Write a failing test first
2. **Green** - Write minimal code to pass
3. **Refactor** - Improve design with tests as safety net

Test naming: `Test<FunctionName>_<Condition>_<ExpectedBehavior>`

### Critical Rules

1. **No DSLs or templating** - Keep request definitions pure JSON/YAML
2. **Update README.md** when modifying magefile.go commands
3. **All responses in English** - Think in English regardless of instruction language
4. **Function registry** - All new functions must be registered
5. **Use mage** - Never use raw `go` commands for project tasks
6. **Subcommand separation** - Each subcommand has its own flag.FlagSet and argument parsing

### Request Definition Structure

Requests support two formats:

1. **Map-based** (simple key-value):
```json
{
    "method": "GET",
    "path": "/api/test",
    "query": {"key": "value"},
    "headers": {"Content-Type": "application/json"},
    "params": {"id": "123"}
}
```

2. **Array-based** (duplicate keys):
```json
{
    "query": [
        {"key": "q", "value": "v1"},
        {"key": "q", "value": "v2"}
    ]
}
```

Variables and functions can be used anywhere in the request definition.

### Key Implementation Details

- **Variable Context**: Parser uses `context.WithValue(ctx, "variables", vars)` to pass variables
- **Priority System**: CLI variables (--var) override file-defined variables
- **Array Expansion**: `generateVariableCombinations()` creates Cartesian product of array variables
- **Format Detection**: `detectFormat()` auto-detects JSON/YAML/JSONL input format
- **Path Field Type**: Changed from `string` to `interface{}` to support function calls

## Testing Guidelines

- When executing command tests, use Docker's httpbin for consistent and controlled HTTP testing
- Test both main command and subcommands separately
- Verify variable override priority (CLI > file variables)
- Test array expansion logic with multiple combinations
- Validate error handling for invalid input formats

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/secureta)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/secureta)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
