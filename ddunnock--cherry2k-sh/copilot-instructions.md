## cherry2k-sh

> > **Project**: Cherry2K.sh - Zsh Terminal AI Assistant

# CLAUDE.md

> **Project**: Cherry2K.sh - Zsh Terminal AI Assistant
> **Language**: Rust (≥1.93, Edition 2024)
> **Workflow**: Get Shit Done (GSD) Claude Code

---

## Overview

Cherry2K.sh is a zsh terminal-based AI assistant built in Rust with a provider-agnostic architecture. It supports multiple AI backends (OpenAI, Anthropic, Ollama) through a unified trait abstraction, uses SQLite for conversation persistence, and integrates into zsh via pure shell functions and ZLE widgets.

## Quick Start

```bash
# Build and install
cargo build --release
./install.sh  # Sets up zsh integration

# Development
cargo check                    # Fast type checking
cargo clippy -- -D warnings    # Lint with warnings as errors
cargo test                     # Run tests
cargo llvm-cov --fail-under-lines 80  # Coverage check
```

## Project Structure

```
cherry2k/
├── crates/
│   ├── core/                   # Domain logic + provider abstraction
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   ├── provider/       # AI provider trait + implementations
│   │   │   │   ├── mod.rs
│   │   │   │   ├── trait.rs    # AiProvider trait definition
│   │   │   │   ├── openai.rs
│   │   │   │   ├── anthropic.rs
│   │   │   │   └── ollama.rs
│   │   │   ├── conversation/   # Conversation management
│   │   │   ├── config/         # Configuration handling
│   │   │   └── error.rs        # Error types (thiserror)
│   │   └── Cargo.toml
│   ├── storage/                # SQLite persistence (rusqlite)
│   │   ├── src/
│   │   │   ├── lib.rs
│   │   │   ├── schema.rs       # Database schema
│   │   │   ├── migrations/     # SQL migrations
│   │   │   └── repository.rs   # Data access layer
│   │   └── Cargo.toml
│   └── cli/                    # Terminal interface
│       ├── src/
│       │   ├── main.rs
│       │   ├── commands/       # CLI commands
│       │   ├── repl/           # Interactive mode
│       │   └── output/         # Terminal formatting
│       └── Cargo.toml
├── zsh/                        # Zsh integration
│   ├── cherry2k.plugin.zsh     # Main plugin file
│   ├── widgets/                # ZLE widget functions
│   └── completions/            # Zsh completions
├── .claude/
│   └── standards/              # Project standards (see below)
├── Cargo.toml                  # Workspace root
└── CLAUDE.md                   # This file
```

## Standards Reference

**CRITICAL**: Always consult these standards before implementation:

| File                                                   | When to Read                              |
|--------------------------------------------------------|-------------------------------------------|
| [constitution.md](.claude/standards/constitution.md)   | Always - global principles, quality gates |
| [rust.md](.claude/standards/rust.md)                   | All Rust development                      |
| [testing.md](.claude/standards/testing.md)             | Writing tests                             |
| [security.md](.claude/standards/security.md)           | API keys, secrets, input validation       |
| [git-cicd.md](.claude/standards/git-cicd.md)           | Commits, branches, PRs                    |
| [documentation.md](.claude/standards/documentation.md) | Doc comments, README updates              |

## GSD Workflow

This project uses **Get Shit Done (GSD)** methodology with Claude Code:

1. **Before Starting**: Run `/gsd:progress` to check current state
2. **Planning**: Use `/gsd:plan-phase` for feature planning
3. **Execution**: Use `/gsd:execute-phase` for implementation
4. **Debugging**: Use `/gsd:debug` for systematic issue resolution
5. **Verification**: Always verify with `/gsd:verify-work`

### GSD Commands Reference

```bash
/gsd:new-project      # Initialize project roadmap
/gsd:plan-phase       # Plan implementation phase
/gsd:execute-phase    # Execute planned work
/gsd:debug           # Systematic debugging
/gsd:verify-work     # Verify completion
/gsd:progress        # Check overall progress
```

## Architecture

### AI Provider Abstraction

The core architecture centers on a provider-agnostic trait:

```rust
/// Core AI provider trait - all providers MUST implement this.
#[async_trait]
pub trait AiProvider: Send + Sync {
    /// Send a message and receive a streaming response.
    async fn complete(&self, request: CompletionRequest) -> Result<CompletionStream, ProviderError>;

    /// Provider identifier for logging/config.
    fn provider_id(&self) -> &'static str;

    /// Validate configuration before use.
    fn validate_config(&self) -> Result<(), ConfigError>;
}
```

### Provider Implementations

| Provider  | API Type     | Auth    | Features                          |
|-----------|--------------|---------|-----------------------------------|
| OpenAI    | REST         | API Key | GPT-4, GPT-3.5, streaming         |
| Anthropic | REST         | API Key | Claude models, streaming          |
| Ollama    | REST (local) | None    | Local models, no network required |

### SQLite Storage

Uses `rusqlite` with migrations for:
- Conversation history
- User preferences
- Provider configurations
- Session metadata

**Homebrew Integration**: SQLite installed via `brew install sqlite3` for optimal macOS performance.

### Zsh Integration

Pure zsh implementation (no external dependencies):
- **Functions**: Shell functions in `cherry2k.plugin.zsh`
- **Widgets**: ZLE widgets for keybindings (e.g., `Ctrl+G` for AI assist)
- **Completions**: Tab completion for commands and options

## Development Guidelines

### Code Quality Gates

All code MUST pass before merge:

```bash
cargo fmt --check              # Formatting
cargo clippy -- -D warnings    # Linting (warnings = errors)
cargo test                     # All tests pass
cargo llvm-cov --fail-under-lines 80  # 80% line coverage minimum
```

### Error Handling

Use `thiserror` for library errors, propagate with `?`:

```rust
#[derive(Debug, Error)]
pub enum ProviderError {
    #[error("API request failed: {0}")]
    RequestFailed(#[from] reqwest::Error),

    #[error("Invalid API key")]
    InvalidApiKey,

    #[error("Rate limited, retry after {0} seconds")]
    RateLimited(u64),
}
```

**MUST NOT** use `.unwrap()` or `.expect()` in library code.

### Async Patterns

- Use `tokio` runtime for async operations
- Native async traits (Rust 2024 edition)
- Library code accepts executor, doesn't create runtime

### Security Requirements

From [security.md](.claude/standards/security.md):

- API keys MUST be stored in environment variables or config file with 0600 permissions
- NEVER log API keys or tokens
- Validate all user input before passing to providers
- Use HTTPS for all API calls (except local Ollama)

## Testing Strategy

### Test Organization

```
crates/core/
├── src/
│   └── provider/
│       └── openai.rs           # Has #[cfg(test)] mod tests
└── tests/
    └── integration_test.rs     # Integration tests
```

### Required Test Patterns

```rust
#[cfg(test)]
mod tests {
    use super::*;

    mod fixtures {
        pub fn mock_config() -> ProviderConfig { ... }
    }

    #[test]
    fn provider_handles_valid_request() {
        // Arrange
        let config = fixtures::mock_config();
        // Act
        let result = ...;
        // Assert
        assert!(result.is_ok());
    }
}
```

### Coverage Enforcement

```bash
cargo install cargo-llvm-cov
cargo llvm-cov --fail-under-lines 80
```

## Commit Guidelines

Follow Conventional Commits (from [git-cicd.md](.claude/standards/git-cicd.md)):

```
feat(provider): add Ollama support for local models
fix(storage): handle SQLite busy timeout
docs(readme): add installation instructions
test(cli): add integration tests for REPL
```

### Scopes

| Scope      | Description                 |
|------------|-----------------------------|
| `provider` | AI provider implementations |
| `storage`  | SQLite/rusqlite code        |
| `cli`      | CLI commands and REPL       |
| `zsh`      | Zsh integration scripts     |
| `config`   | Configuration handling      |

## File Naming Conventions

| Type          | Pattern                | Example                   |
|---------------|------------------------|---------------------------|
| Rust modules  | `snake_case.rs`        | `openai_provider.rs`      |
| Zsh functions | `kebab-case`           | `cherry2k-complete`       |
| Config files  | `snake_case`           | `cherry2k_config.toml`    |
| Migrations    | `NNNN_description.sql` | `0001_initial_schema.sql` |

## Key Dependencies

```toml
[workspace.dependencies]
tokio = { version = "1.49", features = ["full"] }
serde = { version = "1.0.228", features = ["derive"] }
serde_json = "1.0.149"
thiserror = "2.0.18"
anyhow = "1.0.100"
tracing = "0.1.44"
tracing-subscriber = { version = "0.3.22", features = ["env-filter"] }
toml = "0.9.11"
directories = "6.0.0"
clap = { version = "4.5.56", features = ["derive"] }
sentry = { version = "0.46.1", features = ["backtrace", "contexts", "panic", "tracing", "reqwest", "rustls"] }
dotenvy = "0.15.7"
```

## Environment Variables

```bash
# Required for cloud providers
OPENAI_API_KEY=sk-...
ANTHROPIC_API_KEY=sk-ant-...

# Optional
CHERRY2K_CONFIG_PATH=~/.config/cherry2k/config.toml
CHERRY2K_LOG_LEVEL=info
OLLAMA_HOST=http://localhost:11434
```

## Common Tasks

### Adding a New Provider

1. Create `crates/core/src/provider/new_provider.rs`
2. Implement `AiProvider` trait
3. Add to `ProviderFactory` in `mod.rs`
4. Add configuration schema
5. Write tests (mock HTTP responses)
6. Update documentation

### Adding a New CLI Command

1. Create handler in `crates/cli/src/commands/`
2. Register with clap in `main.rs`
3. Add zsh completion if interactive
4. Write integration test
5. Update help text

### Database Migration

1. Create `crates/storage/src/migrations/NNNN_description.sql`
2. Add migration to `run_migrations()` function
3. Test upgrade and rollback
4. Document schema changes

## Troubleshooting

### Common Issues

| Issue                | Solution                                                   |
|----------------------|------------------------------------------------------------|
| SQLite locked        | Check for concurrent processes; increase busy timeout      |
| API rate limited     | Implement exponential backoff; check usage limits          |
| Zsh not loading      | Source plugin file; check `$fpath` for completions         |
| Build fails on macOS | Ensure Xcode CLI tools installed: `xcode-select --install` |

---

*Last updated: 2026-01-30*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ddunnock) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
