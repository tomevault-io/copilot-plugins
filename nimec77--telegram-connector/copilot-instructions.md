## telegram-connector

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Telegram MCP Connector — a Model Context Protocol (MCP) service that enables Claude to search Russian-language Telegram channels and messages in real-time. Built in Rust using the `rmcp` SDK and `grammers` Telegram client.

## Build & Test Commands

```bash
cargo build                            # Debug build
cargo build --release                  # Release build
cargo test                             # All tests (some ignored)
cargo test <module>                    # e.g. cargo test mcp, cargo test types
cargo test <test_fn_name>              # Run a single test by name
cargo test config -- --test-threads=1  # Config tests MUST run serial (env var mutation)
cargo test -- --nocapture              # Show println!/dbg! output during tests
cargo fmt --check                      # Check formatting
cargo clippy -- -D warnings           # Lint (warnings = errors)
cargo run --bin telegram-mcp           # Run the binary

# Pre-commit (all must pass)
cargo fmt --check && cargo clippy -- -D warnings && cargo test
```

## Toolchain & Dependencies

- **Rust nightly** (2024 edition) — required for `let chains` and other nightly features; no `rust-toolchain.toml`, nightly is implied by `edition = "2024"`
- **`grammers` from git master** (not crates.io) — API can change between builds; check `grammers-client` docs if compilation fails after update
- **`schemars` v1** (not v0.8) — different derive API; uses `#[derive(JsonSchema)]` from `schemars::JsonSchema`
- **`rmcp` v0.15** — MCP server SDK; `#[tool_router]` and `#[tool(...)]` proc macros
- **`secrecy`** — `SecretString` wraps sensitive config fields (`api_hash`, `phone_number`); access via `.expose_secret()`

## Architecture

```
MCP Client (Claude) ──JSON-RPC/stdio──► MCP Server Layer (rmcp)
                                        │  src/mcp/server.rs (8 tools)
                                        │  src/mcp/tools/ (types, helpers)
                                        ▼
                                      Application Layer
                                        │  config, logging, rate_limiter, link, error, cli
                                        ▼
                                      Telegram Layer (grammers)
                                        │  client.rs, trait_def.rs, converters.rs, auth.rs
                                        │  types/ (ids, names, media, entities, params)
                                        ▼
                                      Telegram Cloud API (MTProto)
```

### Key Patterns

**Generic MCP server with trait-based DI:** `McpServer<T: TelegramClientTrait, R: RateLimiterTrait>` takes `Arc<T>` and `Arc<R>`. In production, T=`TelegramClient`, R=`RateLimiter`. In tests, mockall-generated `MockTelegramClientTrait` and `MockRateLimiterTrait`.

**rmcp tool macros:** All 8 tools live in `server.rs` inside a `#[tool_router] impl` block. Each tool method has a `#[tool(...)` attribute. Tools cannot be split to separate files due to macro constraints. All tools return `Result<String, String>` and serialize responses to JSON — this is an rmcp constraint.

**Library + Binary split:** `lib.rs` exports all public types/modules. `main.rs` is the CLI entry point with signal handling and setup mode.

**No `mod.rs` files:** File-as-module pattern throughout. `src/mcp.rs` declares submodules, `src/telegram.rs` declares submodules.

**Type-safe domain model (DDD):** Newtype wrappers `ChannelId(i64)`, `MessageId(i64)`, `UserId(i64)`, `Username(String)`, `ChannelName(String)` prevent accidental misuse. JSON schemas via `schemars` derives.

**Config resolution:** `TELEGRAM_MCP_CONFIG` env var → `~/.config/telegram-connector/config.toml`. Supports `${VAR}` env var expansion in TOML values (pure-integer values are auto-unquoted for TOML type compatibility).

### Test Organization

- Unit tests: `#[cfg(test)]` blocks inline in each module, or in separate files via `#[path = "..."]` attribute (e.g., `config.rs` → `config/tests.rs`)
- MCP tool tests: `src/mcp/tests/{server_core,status,channels,links,search,history,message_by_link}.rs`
- Telegram client tests: `src/telegram/tests/client_tests.rs` (mock-based)
- Test fixtures: `src/test_helpers.rs` — `create_test_message()`, `create_test_channel()`, etc.
- Config tests require `--test-threads=1` due to `env::set_var()` race conditions

## Coding Conventions

Full detail in `docs/conventions.md`. Key rules:

**Error handling:**
- Library code uses `thiserror` (typed errors in `src/error.rs`); application code uses `anyhow` (context + propagation)
- **Never `unwrap()`** in production code — use `?` or `.context("...")`
- `expect()` is allowed only in tests or truly impossible situations

**Logging:** Use `tracing`. Never log phone numbers, API hashes, passwords, or session tokens.

**Style:** Line length 100 chars. Run `cargo fmt --all` after every code change (not just `--check`).

**TDD:** Write the failing test first; no production code without a preceding test.

## Critical Rules

1. **NEVER create git commits** — The user manages all git operations. Only write code and documentation.
2. **Always use LOCAL memory file** `docs/memory.md`, NOT global Claude memory.
3. **Wait for user approval** before implementing any proposed changes.

## Workflow

See `docs/workflow.md` for the full iteration cycle: PROPOSE → AGREE → IMPLEMENT → VERIFY → UPDATE PROGRESS → UPDATE MEMORY

**Session start:** Read `docs/tasklist.md`, report current phase and next task, ask "Continue from [task]?"

Progress tracked in:
- `docs/tasklist.md` — phase checklist and task details
- `docs/memory.md` — patterns, decisions, and lessons learned

---
> Source: [nimec77/telegram-connector](https://github.com/nimec77/telegram-connector) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
