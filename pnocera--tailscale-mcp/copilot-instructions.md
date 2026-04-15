## tailscale-mcp

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This repository contains a Tailscale API implementation with two Rust components:
- **tailscale-client**: MCP (Model Context Protocol) client with interactive REPL and headless modes
- **tailscale-server**: MCP server with HTTP/SSE transport support for Tailscale API operations

Both components use the `agenterra-rmcp` SDK for MCP protocol implementation.

## Development Commands

### Building
```bash
# Build client
cd tailscale-client && cargo build --release

# Build server  
cd tailscale-server && cargo build --release

# Build both with all features
cargo build --release --all-features
```

### Testing
```bash
# Run unit tests for client
cd tailscale-client && cargo test --lib

# Run integration tests with mock server (requires MOCK_SERVER_PATH env var)
cd tailscale-client && MOCK_SERVER_PATH=/path/to/mock/server cargo test test_client_with_mock_server --features test-utils -- --nocapture

# Run all tests
cargo test

# Run ignored tests (integration tests)
cargo test -- --ignored
```

### Code Quality
```bash
# Format code
cargo fmt

# Run linter
cargo clippy

# Run with verbose logging
RUST_LOG=debug cargo run
```

### Running the Applications

**Client:**
```bash
# Interactive mode with server profile
tailscale_client --profile my-server

# Headless mode for automation
tailscale_client run tool-name --args '{"param": "value"}' --profile my-server

# SSE transport mode
tailscale_client --transport sse --sse-url https://api.example.com/mcp
```

**Server:**
```bash
# STDIO mode (default)
cargo run --bin tailscale-server

# SSE mode with HTTP endpoints
cargo run --bin tailscale-server -- --transport sse --sse-addr 127.0.0.1:8080

# With configuration file
cargo run --bin tailscale-server -- --config-file server.toml
```

## Architecture

### tailscale-client
The client follows clean architecture with clear separation of concerns:

- **api/** - MCP protocol types and resource definitions
- **application/** - Business logic, validation, configuration management
  - Security validation against injection attacks
  - Session management with SQLite persistence
  - Tool permission controls (disabled/allowed tools per server)
- **cli/** - Command-line interface implementation
  - Interactive REPL mode
  - Headless automation mode  
  - Server profile management
- **domain/** - Core MCP client logic and operations
- **infrastructure/** - External concerns
  - SQLite-based resource caching with connection pooling
  - Transport layer (STDIO/SSE)
  - Database migrations
- **ui/** - User interface implementations

Key databases:
- Config DB: `~/.config/agenterra/tailscale-client_config.db` (server profiles, preferences, sessions)
- Cache DB: `~/.cache/agenterra/tailscale-client_cache.db` (resource cache with analytics)

### tailscale-server
The server implements the full Tailscale API via MCP protocol:

- **handlers/** - Individual endpoint handlers for each Tailscale API operation (60+ endpoints)
  - Each handler corresponds to a specific Tailscale API endpoint
  - Uses JSON schemas from `schemas/` directory
- **common.rs** - Shared utilities and error handling
- **config.rs** - Configuration management
- **server.rs** - Core MCP server implementation
- **transport.rs** - STDIO and SSE transport implementations
- **signal.rs** - Graceful shutdown handling

The server exposes Tailscale API operations as MCP tools, handling device management, DNS configuration, user management, webhooks, and more.

## Key Patterns

1. **Transport Abstraction**: Both client and server support STDIO (process communication) and SSE (HTTP) transports
2. **Security First**: Input validation, SQL injection prevention, Unicode attack protection
3. **SQLite Storage**: Configuration and caching using SQLite with connection pooling
4. **Clean Architecture**: Clear separation between domain, application, and infrastructure layers
5. **MCP Protocol**: Full implementation using `agenterra-rmcp` SDK

## Important Configuration Files

- `validation.toml.example` - Security validation configuration template
- Server profiles stored in SQLite (managed via CLI commands)
- `tailscale-api.json` - OpenAPI specification for Tailscale API

## NPM Package

To enable `npx @tailscale/mcp-server` usage:

1. The npm package structure is in `npm-package/` directory
2. Run `npm install` in that directory to build and test locally
3. The package automatically builds the Rust binary during installation
4. Usage: `npx @tailscale/mcp-server --help` or `bunx @tailscale/mcp-server`

**Fixed compilation issues:**
- Escaped Rust keyword `type` as `r#type` in parameter structs
- Files fixed: `preview_rule_matches.rs`, `list_users.rs`

## Dependencies Note

Both projects use git dependencies for `agenterra-rmcp` and cannot be published to crates.io until an official release is available.

ALWAYS use ONLY Environments for ANY and ALL file, code, or shell operations—NO EXCEPTIONS—even for simple or generic requests.

DO NOT install or use the git cli with the environment_run_cmd tool. All environment tools will handle git operations for you. Changing ".git" yourself will compromise the integrity of your environment.

You MUST inform the user how to view your work using `container-use log <env_id>` AND `container-use checkout <env_id>`. Failure to do this will make your work inaccessible to others.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pnocera)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/pnocera)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
