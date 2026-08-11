## mcp-tg-client

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`mcp-telegram-bridge` is a Rust application that acts as a bridge between Telegram and ChatGPT-5 through the Model Context Protocol (MCP). It allows users to interact with ChatGPT-5 via a Telegram bot interface.

## Development Commands

```bash
# Build the project
cargo build

# Run in development mode
cargo run

# Run tests
cargo test

# Check code without building
cargo check

# Format code
cargo fmt

# Run clippy for linting
cargo clippy
```

## Project Architecture

### Core Modules

- **`src/main.rs`** - Application entry point, initializes all components
- **`src/config/`** - Configuration management using environment variables
- **`src/mcp/`** - MCP client implementation for ChatGPT-5 communication
- **`src/bot/`** - Telegram bot handler and message processing

### Key Components

1. **Config System** (`src/config/mod.rs`)
   - Loads configuration from environment variables
   - Manages Telegram bot token, MCP endpoint, and API keys

2. **MCP Client** (`src/mcp/mod.rs`)
   - Implements Model Context Protocol for ChatGPT-5 communication
   - Handles async request/response with proper error handling
   - Uses trait-based design for testability

3. **Telegram Bot** (`src/bot/mod.rs`)
   - Uses `teloxide` crate for Telegram Bot API
   - Implements command handling (`/start`, `/help`, `/clear`, `/status`)
   - Manages concurrent user sessions with semaphore-based rate limiting

### Async Architecture

- Built on `tokio` async runtime
- Concurrent message processing with configurable limits
- Non-blocking I/O for both Telegram and MCP communications

## Environment Setup

Copy `.env.example` to `.env` and configure:
- `TELEGRAM_BOT_TOKEN` - Your Telegram bot token from @BotFather
- `MCP_API_KEY` - API key for MCP/ChatGPT-5 access
- `MCP_ENDPOINT` - MCP server endpoint (defaults to localhost:3000)
- `MCP_MODEL` - Model name (defaults to "chatgpt-5")

## Dependencies

Key external crates:
- `tokio` - Async runtime
- `teloxide` - Telegram Bot API client
- `reqwest` - HTTP client for MCP communication
- `serde` - Serialization/deserialization
- `tracing` - Structured logging
- `anyhow` - Error handling

---
> Source: [rshirochenko/mcp-tg-client](https://github.com/rshirochenko/mcp-tg-client) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-11 -->
