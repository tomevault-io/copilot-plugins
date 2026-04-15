## linebased

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build and Test Commands

```bash
go test ./...                    # Run all tests
go test -v ./...                 # Verbose output
go test -run TestName ./...      # Run specific test
go test -run FuzzExpressions -fuzz FuzzExpressions .  # Fuzz the tokenizer
go vet ./...                     # Static analysis
```

## Architecture

The `linebased` package parses line-based scripts where each expression has a command name and body. Key types:

- **Decoder** (`decoder.go`): Low-level parser that reads `Expression` values from input. Returns syntax errors for malformed input (lines starting with space, etc.).

- **ExpandingDecoder** (`expander.go`): High-level parser that wraps Decoder and handles `define` templates and `include` directives. Returns `Expanded` values with template substitution applied.

- **Expression** (`decoder.go`): Raw parsed expression with Name, Body, Comment, and Line fields. Use `ParseArgs(n)` to split the body into arguments.

- **Expanded** (`expander.go`): Expression after template expansion, includes source file tracking for error messages.

The LSP server (`cmd/linebased/`) is a single-file implementation providing diagnostics, hover, and references for `.linebased` files.

## Code Style

- All sources must be `gofmt` clean
- Package-level errors use `errFoo` pattern

### Error handling

Use `errors.As` instead of type assertions for checking error types:

```go
// Good
var e *SyntaxError
if errors.As(err, &e) {
    // handle e
}

// Bad
if e, ok := err.(*SyntaxError); ok {
    // ...
}
```

### Avoid map[string]any

Use struct types instead of `map[string]any` for structured data. For static JSON responses with trusted input, a string literal is acceptable:

```go
// Good - struct types
type response struct {
    ID     int    `json:"id"`
    Result string `json:"result"`
}

// Good - string literal for static/mostly-static JSON
const initResponse = `{"capabilities": {"hoverProvider": true}}`

// Bad
resp := map[string]any{"id": 1, "result": "ok"}
```

### Error handling in Go

Use `errors.Is` or `errors.As` for error comparisons instead of direct equality checks.
This ensures compatibility with wrapped errors.

```go
// Good
if errors.Is(err, io.EOF) {
    // handle EOF
}
// Good
if errors.Is(err, sql.ErrNoRows) {
    // handle no rows
}

// Bad
if err == sql.ErrNoRows {
    // ...
}
// Bad
if err == io.EOF {
    // ...
}
```




# The gopls MCP server

These instructions describe how to efficiently work in the Go programming language using the gopls MCP server. You can load this file directly into a session where the gopls MCP server is connected.

## Detecting a Go workspace

At the start of every session, you MUST use the `go_workspace` tool to learn about the Go workspace. The rest of these instructions apply whenever that tool indicates that the user is in a Go workspace.

## Go programming workflows

These guidelines MUST be followed whenever working in a Go workspace. There are two workflows described below: the 'Read Workflow' must be followed when the user asks a question about a Go workspace. The 'Edit Workflow' must be followed when the user edits a Go workspace.

You may re-do parts of each workflow as necessary to recover from errors. However, you must not skip any steps.

### Read workflow

The goal of the read workflow is to understand the codebase.

1. **Understand the workspace layout**: Start by using `go_workspace` to understand the overall structure of the workspace, such as whether it's a module, a workspace, or a GOPATH project.

2. **Find relevant symbols**: If you're looking for a specific type, function, or variable, use `go_search`. This is a fuzzy search that will help you locate symbols even if you don't know the exact name or location.
   EXAMPLE: search for the 'Server' type: `go_search({"query":"server"})`

3. **Understand a file and its intra-package dependencies**: When you have a file path and want to understand its contents and how it connects to other files *in the same package*, use `go_file_context`. This tool will show you a summary of the declarations from other files in the same package that are used by the current file. `go_file_context` MUST be used immediately after reading any Go file for the first time, and MAY be re-used if dependencies have changed.
   EXAMPLE: to understand `server.go`'s dependencies on other files in its package: `go_file_context({"file":"/path/to/server.go"})`

4. **Understand a package's public API**: When you need to understand what a package provides to external code (i.e., its public API), use `go_package_api`. This is especially useful for understanding third-party dependencies or other packages in the same monorepo.
   EXAMPLE: to see the API of the `storage` package: `go_package_api({"packagePaths":["example.com/internal/storage"]})`

### Editing workflow

The editing workflow is iterative. You should cycle through these steps until the task is complete.

1. **Read first**: Before making any edits, follow the Read Workflow to understand the user's request and the relevant code.

2. **Find references**: Before modifying the definition of any symbol, use the `go_symbol_references` tool to find all references to that identifier. This is critical for understanding the impact of your change. Read the files containing references to evaluate if any further edits are required.
   EXAMPLE: `go_symbol_references({"file":"/path/to/server.go","symbol":"Server.Run"})`

3. **Make edits**: Make the required edits, including edits to references you identified in the previous step. Don't proceed to the next step until all planned edits are complete.

4. **Check for errors**: After every code modification, you MUST call the `go_diagnostics` tool. Pass the paths of the files you have edited. This tool will report any build or analysis errors.
   EXAMPLE: `go_diagnostics({"files":["/path/to/server.go"]})`

5. **Fix errors**: If `go_diagnostics` (using severity "hint" or higher) reports any errors, fix them. The tool may provide suggested quick fixes in the form of diffs. You should review these diffs and apply them if they are correct. Once you've applied a fix, re-run `go_diagnostics` to confirm that the issue is resolved. It is OK to ignore 'hint' or 'info' diagnostics if they are not relevant to the current task. Note that Go diagnostic messages may contain a summary of the source code, which may not match its exact text.

6. **Run tests**: Once `go_diagnostics` reports no errors (and ONLY once there are no errors), run the tests for the packages you have changed. You can do this with `go test [packagePath...]`. Don't run `go test ./...` unless the user explicitly requests it, as doing so may slow down the iteration loop.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bmizerany)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/bmizerany)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
