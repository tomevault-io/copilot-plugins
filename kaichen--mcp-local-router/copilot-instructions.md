## mcp-local-router

> This is an MCP (Model Context Protocol) local router written in Rust that serves as an aggregation proxy for MCP servers. It connects to multiple upstream MCP servers and aggregates their functionalities into a single interface for downstream clients.

# AGENTS.md

## Project Overview

This is an MCP (Model Context Protocol) local router written in Rust that serves as an aggregation proxy for MCP servers. It connects to multiple upstream MCP servers and aggregates their functionalities into a single interface for downstream clients.

## Development Commands

### Build and Run
```bash
# Build the project
cargo build --release

# Run with stdio transport (default)
cargo run -- --config mcp.json --transport stdio

# Run with SSE transport
cargo run -- --config mcp.json --transport sse --address 127.0.0.1:8080
```

### Development Build
```bash
cargo build
cargo run -- --config mcp.json --transport stdio
```

### Testing and Quality
```bash
# Run tests
cargo test

# Run clippy for linting
cargo clippy

# Format code
cargo fmt
```

## Architecture

### Core Components

- **AggregatorService**: The main service that manages multiple upstream MCP server connections and aggregates their tools
- **ServerConfig**: Configuration structure for defining upstream MCP servers with command, args, and environment variables
- **Transport Layer**: Supports both stdio and SSE (Server-Sent Events) transports

### Key Design Patterns

- **Async Architecture**: Built on tokio async runtime with async/await throughout
- **Client Management**: Uses `Arc<RwLock<HashMap<>>>` for thread-safe client storage with `tokio::sync::Mutex` for individual client access
- **Tool Discovery**: Asynchronous tool discovery with timeout handling (30s timeout) to avoid blocking initialization
- **Error Handling**: Comprehensive error handling with anyhow for error context and rmcp ErrorData for MCP protocol errors

### Configuration

Configuration is JSON-based with the following structure:
- `mcpServers`: Object mapping server names to server configurations
- Each server config contains `command`, `args` array, and optional `env` object for environment variables

Example configurations are provided in `mcp.example.json` and `mcp.json`.

### Dependencies

- **rmcp**: Core MCP SDK (crates.io `0.2.0`) with features for client, server, macros, child-process transport, and SSE server
- **tokio**: Async runtime with full feature set
- **tracing**: Structured logging with file appender support
- **clap**: Command-line argument parsing
- **serde/serde_json**: Configuration serialization

## Important Implementation Details

- Tool discovery happens asynchronously after service initialization to prevent blocking
- Client connections are protected by mutexes and can be safely shut down
- The aggregator routes tool calls to appropriate upstream servers based on tool ownership
- Logging is configured to stderr with DEBUG level filtering via environment variables
- Graceful shutdown is implemented for SSE transport with Ctrl+C handling

---
> Source: [kaichen/mcp-local-router](https://github.com/kaichen/mcp-local-router) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
