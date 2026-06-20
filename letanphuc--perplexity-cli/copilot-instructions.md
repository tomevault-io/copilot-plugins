## perplexity-cli

> A Rust CLI tool for Perplexity search/chat via OpenRouter API (`https://openrouter.ai/api/v1`).

# AGENTS.md — Perplexity CLI

## Project

A Rust CLI tool for Perplexity search/chat via OpenRouter API (`https://openrouter.ai/api/v1`).

## Build

```bash
cargo build --release
cargo run -- ask "test query"
```

## Run

```bash
# One-shot commands
./target/release/pc ask "question"
./target/release/pc search "query"
./target/release/pc reason "complex problem"
./target/release/pc research "deep dive topic"

# TUI chat
./target/release/pc chat

# With options
./target/release/pc ask -m perplexity/sonar-reasoning-pro --temperature 0.3 "query"
```

## Env

- `OPENROUTER_API_KEY` — required for API calls
- `RUST_LOG` — controls log verbosity

## Check

```bash
cargo fmt --check
cargo clippy -- -D warnings
cargo test
```

## Code Conventions

- `src/main.rs` — entry point, tokio runtime
- `src/cli.rs` — clap derive structs
- `src/api.rs` — `OpenRouterClient` with `chat()` and `chat_stream()` methods
- `src/tui.rs` — ratatui app with `App` struct holding state
- `src/config.rs` — `Config` struct from env/file/cli merge
- Use `anyhow::Result` for error handling
- Use `tracing` for logging (info, debug, error)
- Follow standard Rust idioms (no unsafe, use `?` for error propagation)
- No comments unless the logic is non-obvious

---
> Source: [letanphuc/perplexity-cli](https://github.com/letanphuc/perplexity-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-20 -->
