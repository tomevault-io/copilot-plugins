## hyprflow

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Test Commands

```bash
cargo build --release              # Release binary (~2 MB)
cargo test                         # All 52 tests (47 unit + 5 integration)
cargo test --lib                   # 47 unit tests only
cargo test --test cli_test         # 5 integration tests only
cargo test <test_name>             # Single test by name
cargo clippy --all-targets         # Lint (known warnings: 5x assert_cmd::cargo_bin deprecated, 1x too-many-arguments)
cargo install --path .             # Install to ~/.cargo/bin/
```

## Architecture

Hyprflow is a Rust CLI that captures and restores Hyprland window sessions via `hyprctl` IPC.

### Core Modules

- **main.rs** — clap-derived CLI with subcommands: `save`, `restore`, `list`, `delete`, `config`
- **capture.rs** — Queries `hyprctl clients/monitors`, resolves CWD/last-command per window, builds `Session`
- **restore.rs** — Groups clients by workspace, spawns binaries sequentially, polls for new window address, positions via `hyprctl dispatch` (movetoworkspacesilent → resizewindowpixel exact → movewindowpixel exact)
- **session.rs** — `Session`/`SessionClient`/`LaunchInfo` data model, JSON file I/O under `$XDG_DATA_HOME/hyprflow/sessions/`
- **config.rs** — TOML config at `$XDG_CONFIG_HOME/hyprflow/config.toml`, per-app capture settings, ignore-class filters
- **hyprctl.rs** — `HyprctlClient` trait + `RealHyprctl` (shells out to `hyprctl`) + `MockHyprctl` for tests
- **process.rs** — `ProcessInfoProvider` trait + `RealProcessInfo` (reads `/proc`) + `MockProcessInfo` for tests
- **brave.rs** — Reads Brave `Local State` JSON, extracts profile info, filters by config

### Key Design Pattern

Trait-based dependency injection (`HyprctlClient`, `ProcessInfoProvider`) enables full unit testing without a running Hyprland session. Mocks record dispatches and return fixture data.

### Restore Flow Detail

Each window is restored sequentially: spawn → poll for new address (100ms intervals, configurable timeout) → position via hyprctl dispatch commands. The address-diff approach detects which new window belongs to which spawn.

### Test Fixtures

`tests/fixtures/` contains real `hyprctl` JSON output (3 windows on 2 monitors) used by both unit and integration tests.

## Active Development

Branch `feat/v0.2` is active development for v0.2.0. All P0 bugs from v0.1.0 are resolved.

---
> Source: [isorensen/hyprflow](https://github.com/isorensen/hyprflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
