## orchids2cc

> Rust TUI proxy server that proxies orchids.app AI model service to Claude Code CLI. Provides both TUI and headless modes for configuration and operation.

## Project Summary
Rust TUI proxy server that proxies orchids.app AI model service to Claude Code CLI. Provides both TUI and headless modes for configuration and operation.

## Tech Stack
- Language: Rust (Edition 2021)
- Runtime: Tokio async runtime
- HTTP Server: Axum 0.7
- HTTP Client: Reqwest 0.12
- TUI Framework: Ratatui 0.29 + Crossterm 0.28
- Logging: Tracing + tracing-subscriber

## Architecture
- `src/main.rs`: Application entry, config/state management, mode selection
- `src/proxy.rs`: Axum HTTP proxy server, handles /v1/messages endpoint with SSE streaming
- `src/tui.rs`: Terminal UI for configuration and monitoring
- Config via environment variables: ORCHIDS_URL, ORCHIDS_TOKEN, LISTEN_PORT

## User Preferences

## Project Guidelines
- Claude Code compatible: Set ANTHROPIC_BASE_URL=http://localhost:8082

## Common Patterns
- State management via Arc<Mutex<T>> for shared config/stats
- SSE streaming via tokio channels + ReceiverStream

---
> Source: [shijianzhong/orchids2cc](https://github.com/shijianzhong/orchids2cc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
