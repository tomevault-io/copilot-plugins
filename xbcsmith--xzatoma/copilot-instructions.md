## xzatoma

> **CRITICAL**: Mandatory rules for AI agents working on XZatoma. Non-compliance

# AGENTS.md - AI Agent Development Guidelines

**CRITICAL**: Mandatory rules for AI agents working on XZatoma. Non-compliance
will result in rejected code.

---

## Critical Rules

## Rule 0: Use the Agent Harness Tools

Use `agent_harness` for all agent interactions

### Rule 1: File Extensions

- Use `.yaml` for ALL YAML files (NOT `.yml`)
- Use `.md` for ALL Markdown files (NOT `.MD`, `.markdown`)
- Use `.rs` for ALL Rust files

CI/CD pipelines expect `.yaml`. Using `.yml` causes build failures.

### Rule 2: Markdown File Naming

- Use `lowercase_with_underscores.md` for all documentation files
- `README.md` is the ONLY exception to the lowercase rule
- Never use CamelCase, kebab-case, spaces, or uppercase

Inconsistent naming breaks documentation links.

### Rule 3: No Emojis

- No emojis in code, comments, documentation, or commit messages
- Exception: This AGENTS.md file only

Emojis cause encoding issues and break tooling.

### Rule 4: Quality Gates (ALL Must Pass)

Run in this order before claiming any task complete:

```bash
cargo fmt --all
cargo check --all-targets --all-features
cargo clippy --all-targets --all-features -- -D warnings
cargo test -p xzatoma --lib -- --skip providers::copilot --skip mcp::auth
```

**NEVER run `cargo test --all-features` or `cargo test -p xzatoma --lib` bare.**
The `providers::copilot` and `mcp::auth` modules link against the macOS
`Security.framework` via the `keyring` crate. On macOS, any freshly compiled
binary that links that framework triggers an OS Keychain access dialog on first
execution, even when no `#[ignore]`-guarded test function runs. Always pass
`--skip providers::copilot --skip mcp::auth` to keep tests hermetic.

**Keyring access chain -- know this to avoid re-introducing the bug:**

```text
keyring::Entry::{get,set,delete}_password
  <- CopilotProvider::{get_cached_token, cache_token, clear_cached_token}
    <- is_authenticated(), authenticate(), fetch_copilot_models()
  <- TokenStore::{save,load,delete}_token
    <- AuthManager::get_token(), handle_401()
```

`CopilotProvider::new()` itself is safe -- it only stores strings. The keyring
is hit as soon as any method that checks or updates authentication is called.

Modules that are gated by `#[ignore = "requires system keyring"]` (not by
`--skip`) include tests in `src/acp/stdio.rs` that call `run_client_server_test`
or `create_session`. Those helpers initialize a session with the default config
(provider = copilot) which calls `provider.list_models()` -> `authenticate()` ->
`get_cached_token()` -> keyring. Never remove the `#[ignore]` annotation from
those tests.

To run the full suite including keyring round-trips (only in a trusted
environment where Keychain prompts are acceptable):

```bash
XZATOMA_RUN_KEYCHAIN_TESTS=1 cargo test -p xzatoma --lib -- --include-ignored
```

**MANDATORY**: All Markdown files must pass linting and formatting checks:

```bash
markdownlint --fix --config .markdownlint.json "${FILE}"
prettier --write --parser markdown --prose-wrap always "${FILE}"
```

Stop immediately and fix if any command fails.

### Rule 5: Documentation is Mandatory

- Create `docs/explanation/<feature_name>_implementation.md` for every feature
  or task
- Add `///` doc comments to every public function, struct, enum, and module
- Include runnable examples in doc comments (they are compiled by `cargo test`)
- Never skip documentation because "code is self-documenting"

### Rule 6: Use the Agent Harness Tools

Do not write custom scripts for tasks that can be accomplished with the agent
tools.

---

## Project Overview

- **Name**: XZatoma
- **Type**: Autonomous AI agent CLI
- **Language**: Rust (latest stable)
- **Purpose**: Execute tasks through conversation with AI providers using basic
  file system and terminal tools
- **Providers**: GitHub Copilot, Ollama

### Module Structure

```text
src/
├── main.rs              # Entry point
├── lib.rs               # Library root
├── cli.rs               # CLI parsing and user interface
├── config.rs            # Configuration management
├── error.rs             # Error types and conversions
├── agent/               # Autonomous execution loop
│   ├── mod.rs
│   ├── agent.rs         # Main agent logic
│   ├── conversation.rs  # Message history
│   └── executor.rs      # Tool execution
├── providers/           # AI provider abstraction
│   ├── mod.rs
│   ├── base.rs          # Provider trait
│   ├── copilot.rs       # GitHub Copilot
│   └── ollama.rs        # Ollama
└── tools/               # Basic file and terminal tools
    ├── mod.rs
    ├── file_ops.rs      # File operations
    ├── terminal.rs      # Terminal execution
    └── plan.rs          # Plan parsing
```

### Architecture Principles

XZatoma is intentionally simple. Do NOT over-engineer it.

- Separate concerns by technical responsibility: CLI, agent, providers, tools
- Avoid unnecessary abstraction layers
- Do not abstract prematurely - wait until you have 3 examples
- Do not add complex inheritance hierarchies
- Keep tools generic (file ops, terminal) - no specialized tools

### Module Dependencies

| Module       | Responsibility                 | Dependencies         |
| ------------ | ------------------------------ | -------------------- |
| `cli.rs`     | CLI parsing and user interface | clap                 |
| `config.rs`  | Configuration management       | serde                |
| `agent/`     | Autonomous execution loop      | providers, tools     |
| `providers/` | AI provider abstraction        | reqwest, async-trait |
| `tools/`     | File and terminal operations   | walkdir, similar     |
| `error.rs`   | Error types and conversions    | thiserror, anyhow    |

### Component Boundaries

Permitted dependencies:

- `agent/` may call `providers/`, `tools/`, and `config.rs`
- `providers/` may call `config.rs`
- `tools/` are independent (no cross-dependencies)
- All modules may use `error.rs`

Forbidden dependencies:

- `providers/` must never import from `agent/` or `tools/`
- `tools/` must never import from `agent/` or `providers/`
- `config.rs` must never import from `agent/`, `providers/`, or `tools/`
- No circular dependencies between modules

---

## Rust Coding Standards

### Error Handling

- Use `Result<T, E>` for all recoverable errors
- Use `?` for error propagation
- Use `thiserror` for custom error types
- Never use `unwrap()` or `expect()` without a justification comment
- Never ignore errors with `let _ =`
- Never use `panic!` for recoverable errors

```rust
// Correct pattern
use thiserror::Error;

#[derive(Error, Debug)]
pub enum ConfigError {
    #[error("Failed to read config file: {0}")]
    ReadError(String),
    #[error("Invalid YAML syntax: {0}")]
    ParseError(String),
}

pub fn load_config(path: &str) -> Result<Config, ConfigError> {
    let contents = std::fs::read_to_string(path)
        .map_err(|e| ConfigError::ReadError(e.to_string()))?;
    let config: Config = serde_yaml::from_str(&contents)
        .map_err(|e| ConfigError::ParseError(e.to_string()))?;
    config.validate()?;
    Ok(config)
}

// Acceptable: unwrap with explicit justification
pub fn get_app_version() -> String {
    // SAFETY: Set at compile time, cannot fail
    env!("CARGO_PKG_VERSION").to_string()
}
```

### Doc Comments

Every public function, struct, enum, and module must have a `///` doc comment:

````rust
/// One-line description.
///
/// Longer explanation of behavior and purpose.
///
/// # Arguments
///
/// * `param` - Description
///
/// # Returns
///
/// Description of return value
///
/// # Errors
///
/// Returns `ErrorType` if condition
///
/// # Examples
///
/// ```
/// use xzatoma::module::function;
///
/// let result = function(arg);
/// assert_eq!(result, expected);
/// ```
pub fn function(param: Type) -> Result<ReturnType, Error> {
    // Implementation
}
````

### Testing Standards

- Write tests for ALL public functions
- Test success, failure, and edge cases
- Achieve >80% code coverage
- Use descriptive names: `test_<function>_<condition>_<expected>`

#### Test Isolation Rule: Never use `AcpRuntime::new()` in tests

Always use `AcpRuntime::new_in_memory()` inside unit tests. `AcpRuntime::new()`
opens the shared on-disk `history.db` (the user's production database). When
multiple tests run in parallel they race for write locks and produce
`Storage("Failed to save ACP session")` failures or hangs.

```rust
// WRONG -- writes to ~/Library/Application Support/.../history.db
let runtime = AcpRuntime::new(crate::Config::default());

// CORRECT -- isolated, in-memory, no disk I/O
let runtime = AcpRuntime::new_in_memory(crate::Config::default());
```

The `executor.rs` tests already follow this pattern. Never regress.

```rust
#[cfg(test)]
mod tests {
    use super::*;

    #[test]
    fn test_parse_config_with_valid_yaml() {
        let result = parse_config("key: value");
        assert!(result.is_ok());
        assert_eq!(result.unwrap().key, "value");
    }

    #[test]
    fn test_parse_config_with_invalid_yaml() {
        let result = parse_config("invalid: : yaml");
        assert!(result.is_err());
    }

    #[test]
    fn test_parse_config_with_empty_string() {
        let result = parse_config("");
        assert!(result.is_err());
    }

    #[test]
    fn test_parse_config_propagates_validation_error() {
        let result = parse_config("invalid_field: value");
        assert!(matches!(result, Err(ConfigError::ValidationError(_))));
    }
}
```

---

## Documentation Organization (Diataxis)

Place documentation in the correct category:

| Directory           | Purpose                                        | Examples                                 |
| ------------------- | ---------------------------------------------- | ---------------------------------------- |
| `docs/tutorials/`   | Learning-oriented, step-by-step lessons        | `getting_started.md`                     |
| `docs/how-to/`      | Task-oriented, problem-solving recipes         | `setup_monitoring.md`                    |
| `docs/explanation/` | Understanding-oriented, conceptual discussion  | `phase4_observability_implementation.md` |
| `docs/reference/`   | Information-oriented, technical specifications | `api_specification.md`                   |

Implementation summaries created by AI agents belong in `docs/explanation/`.

---

## Git Conventions

Do not run git commands. The user handles all git interactions.

---

## Living Document

This file is updated as new patterns emerge.

You are a master Rust developer. Follow these rules precisely. All
implementation summaries go in `docs/explanation/` with lowercase filenames.

---
> Source: [xbcsmith/xzatoma](https://github.com/xbcsmith/xzatoma) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
