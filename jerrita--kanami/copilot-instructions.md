## kanami

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CLAUDE-specific rules

- Claude MUST NOT read config.rs under any circumstances.

## Common Development Commands

- **Build**: `cargo build --release`
- **Run (no TLS)**: `cargo run --release`
- **Run (with TLS, for wss:// endpoint)**: `cargo run --release --features=tls`
- **Set log level**: `LOG=debug cargo run --release`

Run the above from the project root. No special test commands or just/makefiles detected.

## High-Level Architecture

- **Purpose**: Kanami is a suckless-styled QQ bot client compatible with NapCat. It is written in Rust for efficiency and minimal resource usage.
- **Entry point**: `src/main.rs` sets up logging and launches the protocol adapter (async).
- **Protocol/Core**: `src/protocol/adapter.rs` manages WebSocket connections, dispatch, event loop, reconnection, and bridging to Application logic.
- **Application Layer**: `src/application/mod.rs` registers bot modules (apps), exposes traits for loading and processing protocol events, and orchestrates calls to per-app logic via async trait objects.
- **Apps**: Each file under `src/application/` implements a specific bot module (e.g., chat, ping, cat, gscore, builtin, cron). They conform to the central `Application` trait.
- **Configuration**: User config is copied from `src/config.def.rs` to `src/config.rs` and edited by user.
- **Extensibility**: Add new functionality by implementing the `Application` trait and registering the new module in `src/application/mod.rs`.

## Other Notes
- The protocol uses both a message/event system and provides support for NapCat extensions (see `src/protocol/extension/`).
- No built-in unit tests detected. Add tests in the `tests/` dir or as #[cfg(test)] in modules per standard Rust practice.
- Large/complex logic: See comments in `src/protocol/adapter.rs` and focus on that file for core event handling.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jerrita)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jerrita)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
