## ninja-linter

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Agents.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Ninja Linter is a Rust CLI tool that automatically runs PHP CS Fixer (and optionally PHPStan) on git-modified `.php` files inside a Docker container named `ninja_symfony`.

## Commands

```bash
# Build
cargo build           # Debug build
cargo build --release # Optimized release build

# development
cargo clippy          # Lint

# Test
cargo test                        # Run all tests
cargo test <test_name_substring>  # Run a specific test

# Run (after build)
./target/debug/ninja-linter            # Basic: lint modified PHP files
./target/debug/ninja-linter --test     # Run configured test command first
./target/debug/ninja-linter --stan     # Also run PHPStan after linting
```

## Architecture

The tool has five modules:

- **`src/main.rs`** — CLI definition (`clap::Parser`) and top-level orchestration. Reads git branch via `shadow-rs` build metadata; exits with error if not in a git repo.
- **`src/command.rs`** — All external process execution: `git status`, `docker exec ninja_symfony composer cs:fix <file>`, `docker exec ninja_symfony composer stan`, and arbitrary test commands. Returns `io::Result<bool>` where `true` = success.
- **`src/file.rs`** — Parses `git status --short` output. Filters for `.php` files only. Normalizes paths: strips `back/` prefix but preserves `src/` and `tests/` prefixes for Docker container compatibility.
- **`src/config.rs`** — Reads/writes `.ninja-linter.json` in the working directory. Stores `test_command` (e.g., `"docker exec ninja_symfony bin/phpunit"`). Created interactively on first `--test` use.
- **`src/parallel.rs`** — Spawns worker threads for concurrent task execution (Tests, CS Fixer, PHPStan). Owns the `TaskStatus`, `TaskState`, and `TaskUpdate` types used by the TUI.
- **`src/tui.rs`** — Ratatui-powered live dashboard displayed in the terminal's alternate screen during `--parallel` runs. The single public entry-point is `run_dashboard(tasks, php_files, rx)`.

### Execution Flow

1. Validate git branch exists (via `shadow-rs` embedded build info)
2. Optionally run test command (`--test` flag → load/prompt config → execute)
3. Get modified `.php` files via `git status --short`
4. For each file: `docker exec ninja_symfony composer cs:fix <file>`
5. Optionally run `docker exec ninja_symfony composer stan` (`--stan` flag)

#### `--parallel` flow

When `--parallel` is passed, steps 2–5 run concurrently:

1. `parallel::run_parallel` spawns up to three threads (tests, CS Fixer, PHPStan)
2. Each thread sends `TaskUpdate::Started(i)` / `TaskUpdate::Finished(i, ok)` over an `mpsc` channel
3. `tui::run_dashboard` enters the alternate screen and drives a 100 ms tick loop:
   - Drains all pending `TaskUpdate` messages via `try_recv`
   - Redraws the ratatui layout (outer border → task list → file list → hint line)
   - Exits automatically when every task reaches `Done` or `Failed`
   - User can also press `q` / `Esc` to exit early

### Key Constraints

- **Docker dependency**: The tool assumes a running Docker container named `ninja_symfony`. No fallback exists.
- **PHP only**: File filtering is hardcoded to `.php` extension.
- **Path normalization**: `src/file.rs:clean_modified_file()` strips the `back/` directory prefix — this is specific to the project's monorepo structure where PHP lives under `back/`.
- **Exit codes**: `0` = success, `1` = any failure (linting failure, docker error, test failure).
- **TUI terminal restore**: `tui::run_dashboard` always restores raw mode and leaves the alternate screen before returning, even on error.

## Testing

Tests live inline in each module under `#[cfg(test)]`. Key test areas:
- `file.rs`: `parse_git_status()` and `clean_modified_file()` path normalization
- `command.rs`: `build_cs_fix_args()` and `CommandStatus` enum values
- `parallel.rs`: `build_initial_tasks()` initial state, success/failure detection
- `tui.rs`: `App` state transitions (`apply`, `is_complete`, `has_failure`), `task_style` coverage

Tests do **not** require Docker or a real terminal — docker-dependent and TUI-dependent functions have unit tests only on their pure logic.

---
> Source: [IgnacioToledoDev/ninja-linter](https://github.com/IgnacioToledoDev/ninja-linter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
