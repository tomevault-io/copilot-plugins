## gitmop

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Core Commands

```bash
# Build the project
cargo build --release

# Run the application (dry-run mode by default)
cargo run

# Run with execute mode (performs actual operations)
cargo run -- --execute

# Run tests
cargo test

# Format and lint code
cargo fmt --all && cargo clippy --all-targets -- -D warnings

# Run a single test
cargo test test_name

# Check compilation without building
cargo check
```

## Architecture Overview

GitMop is a TUI application for managing GitHub repositories with bulk operations (archive, delete, download). The codebase follows a modular architecture:

### Key Components

- **`main.rs`**: Entry point that initializes the CLI parser, loads config, creates auth manager and GitHub client, then launches the TUI app
- **`lib.rs`**: Library root exposing all public modules for potential reuse
- **`auth/`**: GitHub OAuth device flow authentication with token persistence
  - `AuthManager` handles device flow authentication and stores tokens securely in `~/.config/gitmop/auth.json`
- **`github.rs`**: `GitHubClient` wrapper around octocrab for GitHub API interactions
- **`config.rs`**: `Config` struct for user preferences, persisted to `~/.config/gitmop/config.json`
- **`domain.rs`**: Core domain types including `RepoAction`, `AppState`, `AppMode`, and download-related structs
- **`ui/app.rs`**: Main TUI application using ratatui, handles user input and orchestrates operations
- **`download.rs`**: Repository download functionality with progress tracking

### Data Flow

1. Authentication via GitHub OAuth device flow (requires `GITHUB_CLIENT_ID` env var)
2. Fetches repositories where user has admin permissions
3. User marks repos for archive/delete operations in TUI
4. Operations execute in dry-run mode by default (use `--execute` for actual operations)
5. Confirmations required for destructive operations (typing "Yes, I am sure" for deletes)

### Key Dependencies

- **tokio**: Async runtime for concurrent operations
- **octocrab**: GitHub API client
- **ratatui + crossterm**: Terminal UI framework
- **secrecy**: Secure handling of auth tokens
- **clap**: CLI argument parsing

## Environment Requirements

- `GITHUB_CLIENT_ID`: Required for GitHub OAuth authentication (obtain from GitHub OAuth App settings)

## Testing Approach

The project uses standard Rust testing with `cargo test`. Tests should be added for new functionality, particularly for the domain logic and GitHub API interactions.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ariejan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ariejan)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
