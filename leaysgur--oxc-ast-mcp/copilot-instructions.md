## oxc-ast-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an MCP (Model Context Protocol) server that exposes OXC parser functionality. It allows AI assistants to parse JavaScript/TypeScript code, inspect AST structures, and check code for syntactic and semantic errors.

The server implements three main tools:
- `parse`: Parses JS(X) or TS(X) code into an OXC AST
- `docs`: Shows documentation for OXC AST nodes with regex filtering
- `check`: Validates code for syntactic and semantic errors

## Build and Test Commands

### Initial Setup

Before building for the first time, generate the AST node documentation:

```sh
node generate-oxc_ast-nodes.mjs > ast-nodes.generated.json
```

### Build

```sh
# Development build
cargo build

# Release build
cargo build --release
```

### Run Tests

```sh
cargo test
```

### Run Specific Test

```sh
# Run a specific test by name
cargo test test_name

# Run tests in a specific module
cargo test docs::tests
```

### Generate AST Documentation

To regenerate the AST node documentation from the OXC library:

```sh
node generate-oxc_ast-nodes.mjs > ast-nodes.generated.json
```

This requires nightly Rust toolchain as it uses unstable rustdoc JSON output features.

## Architecture

### MCP Server Structure

The server is built using the `rust-mcp-sdk` crate and implements the standard MCP server pattern:

- `src/main.rs`: Entry point that sets up the MCP server with stdio transport
- `src/lib.rs`: Minimal library entry point
- `src/tools/mod.rs`: Tool registry using the `tool_box!` macro to combine all tools
- Individual tool implementations in `src/tools/`:
  - `parse.rs`: AST parsing tool
  - `check.rs`: Code validation tool
  - `docs.rs`: AST documentation tool

### Tool Implementation Pattern

Each tool follows this pattern:

1. Decorated with `#[mcp_tool(...)]` macro with metadata (name, title, description)
2. Implements `MyTool` trait with a `call()` method
3. Returns `Result<CallToolResult, CallToolError>`
4. Uses OXC libraries for parsing/validation

The `tool_box!` macro in `src/tools/mod.rs` combines all tools into a `MyTools` enum that handles dispatch.

### OXC Integration

The tools use these OXC crates:
- `oxc_allocator`: Arena allocator for AST nodes
- `oxc_parser`: JavaScript/TypeScript parser
- `oxc_semantic`: Semantic analysis and error checking
- `oxc_span`: Source location tracking

Supported file extensions: `js`, `mjs`, `cjs`, `jsx`, `ts`, `mts`, `cts`, `tsx`

### AST Documentation Generation

The `generate-oxc_ast-nodes.mjs` script:
1. Runs `cargo rustdoc` with JSON output to extract type information
2. Parses the generated JSON documentation
3. Extracts all public structs and enums from `oxc_ast`
4. Filters out internal types (AstBuilder, AstKind, etc.)
5. Generates simplified Rust-style definitions with field types and docs
6. Outputs to `ast-nodes.generated.json`

This JSON file is embedded into the `docs` tool at compile time using `include_str!`.

### Error Handling

Custom `StringError` type in `src/tools/mod.rs` provides simple string-based error messages that convert to `CallToolError` for MCP responses.

## Development Notes

### Adding a New Tool

1. Create a new file in `src/tools/`
2. Define a struct with `#[mcp_tool(...)]` attribute
3. Implement the `MyTool` trait
4. Add the tool to the `tool_box!` macro in `src/tools/mod.rs`
5. Import the tool in `src/tools/mod.rs`

### Updating OXC Version

When updating OXC dependencies in `Cargo.toml`:
1. Update all `oxc_*` crates to the same version
2. Regenerate AST documentation: `node generate-oxc_ast-nodes.mjs > ast-nodes.generated.json`
3. Test all tools to ensure compatibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leaysgur)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/leaysgur)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
