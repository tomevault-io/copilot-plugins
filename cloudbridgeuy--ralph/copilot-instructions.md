## ralph

> cargo xtask install              # Install to ~/.local/bin

# CLAUDE.md

## Quick Reference

### Installation

```bash
cargo xtask install              # Install to ~/.local/bin
cargo xtask install --path /usr/local/bin  # Custom path
```

### Upgrade

```bash
ralph upgrade                 # Upgrade to latest version
ralph upgrade --force         # Force upgrade
```

## Unnegotiables

These principles are non-negotiable when working on this codebase:

1. **No Dead Code**: Remove unused code immediately. This applies to both Rust and TypeScript. If code is not called, delete it.

2. **Functional Core - Imperative Shell**: Follow this pattern strictly. Pure business logic in core modules, I/O operations in shell. See the dedicated section below for details.

3. **Progressive Disclosure**: Keep documentation layered. High-level overview in CLAUDE.md, detailed docs in `docs/context/` files. Don't duplicate information.

4. **Follow Existing Patterns**: When implementing features, use patterns already established in the codebase. Don't invent new approaches when existing ones work. Consistency trumps novelty.

5. **No .unwrap() or .expect() in Production Code**: Use proper error handling with `?` operator and Result/Option combinators. Test code may use `.unwrap()` for brevity. Any intentional panics must be documented with `#[allow(...)]` and comments explaining the invariant.

6. **Maximum 5 Function Arguments**: Functions with more than 5 arguments must use config/options structs. This is enforced by `clippy::too_many_arguments` with threshold 5 (see `clippy.toml`). Test code may have exceptions where it makes tests clearer.

7. **No Serial or Integration Tests**: Tests must not use `#[serial]` or modify global state (environment variables, current directory). Integration tests that spawn subprocesses or require sequential execution are not permitted. Prefer unit tests on pure functions.

8. **Code Formatting**: All code must be formatted with `cargo fmt`. Run `cargo xtask lint` before committing to ensure formatting compliance. Install git hooks with `cargo xtask lint --install-hooks` for automatic pre-commit formatting checks.

**Example - Refactoring too many arguments:**

```rust
// Before: 6 arguments - violation!
fn invoke_with_recovery(
    command: &str,
    max_attempts: usize,
    timeout_secs: u64,
    iteration: usize,
    theme_config: Option<&ThemeConfig>,
    session_elapsed_ms: u64,
) -> Result<SubprocessResult, Error> { ... }

// After: 1 argument - config struct
struct InvocationConfig<'a> {
    command: &'a str,
    max_attempts: usize,
    timeout_secs: u64,
    iteration: usize,
    theme_config: Option<&'a ThemeConfig>,
    session_elapsed_ms: u64,
}

fn invoke_with_recovery(config: &InvocationConfig) -> Result<SubprocessResult, Error> { ... }
```

## Functional Core - Imperative Shell

We advocate the use of this pattern when writing code for this repo.

The pattern is based on separating code into two distinct layers:

**Functional Core**: Pure, testable business logic free of side effects (no I/O, no external state mutations). It operates only on the data it's given.

**Imperative Shell**: Responsible for side effects like database calls, network requests, and sending emails. It uses the functional core to perform business logic.

### Example Transformation

**Before (mixed logic and side effects):**

```rust
async fn send_user_expiry_emails(db: &Database, email_service: &EmailService) -> Result<()> {
    let users = db.get_users().await?;

    for user in users {
        if user.subscription_end_date > Utc::now() {
            continue;
        }
        if user.is_free_trial {
            continue;
        }

        email_service
            .send(
                &user.email,
                &format!("Your account has expired {}.", user.name),
            )
            .await?;
    }

    Ok(())
}
```

**After (separated):**

**Functional Core:**

```rust
// Pure filtering logic - no side effects
fn get_expired_users(users: &[User], cutoff: DateTime<Utc>) -> Vec<&User> {
    users
        .iter()
        .filter(|user| user.subscription_end_date <= cutoff)
        .filter(|user| !user.is_free_trial)
        .collect()
}

// Pure email generation - no side effects
fn generate_expiry_emails(users: &[&User]) -> Vec<Email> {
    users
        .iter()
        .map(|user| Email {
            to: user.email.clone(),
            subject: "Account Expired".to_string(),
            body: format!("Your account has expired {}.", user.name),
        })
        .collect()
}
```

**Imperative Shell:**

```rust
// Orchestrates I/O operations using pure functions
async fn send_user_expiry_emails(db: &Database, email_service: &EmailService) -> Result<()> {
    let users = db.get_users().await?;
    let expired = get_expired_users(&users, Utc::now());
    let emails = generate_expiry_emails(&expired);
    email_service.bulk_send(&emails).await?;
    Ok(())
}
```

### Benefits

- **More testable**: Core logic can be tested in isolation without mocking I/O
- **More maintainable**: Pure functions are easier to reason about and modify
- **More reusable**: Business logic (e.g., `getExpiredUsers`) can be reused for other features like reminder emails
- **More adaptable**: Imperative shell can be swapped out (e.g., change from email to SMS) without touching core logic

## Progressive Disclosure

Detailed documentation is kept in dedicated files. Consult these when working on related features.

- [Clap CLI Patterns](docs/context/rust-cli-clap.md) - Command routing, subcommands, arguments, and derive macros
- [Error Handling](docs/context/rust-cli-error-handling.md) - color_eyre, thiserror, and error propagation patterns
- [Serde Serialization](docs/context/rust-cli-serde.md) - JSON serialization, field attributes, and API type patterns
- [I/O Patterns](docs/context/rust-cli-io.md) - stdin/stdout handling, terminal detection, async I/O
- [Environment Configuration](docs/context/rust-cli-env-config.md) - Env vars, secrets, config precedence
- [Project Structure](docs/context/rust-cli-project-structure.md) - Workspace setup, xtask, module organization
- [Stream Processor Architecture](docs/context/stream-processor-architecture.md) - StreamProcessor builder, result handlers, output blocks, block-buffered prose, table enhancement
- [Prompt Template System](docs/context/prompt-template.md) - Placeholders, customization, and two-phase workflow
- [Git Workflow](docs/context/git-workflow.md) - Semantic commits, git history usage, Progressive Disclosure pattern
- [ralph ask Command](docs/context/ralph-ask.md) - Single-shot prompts, session continuation, conversation history
- [ralph persona Command](docs/context/ralph-persona.md) - Named agent conversations, persona-scoped sessions
- [Orchestration](docs/context/orchestration.md) - Multi-agent directives, handover/ask, conversation routing
- [ralph edit Command](docs/context/ralph-edit.md) - Session history editing, TOML projection, FC-IS module split
- [ralph strategy Command](docs/context/ralph-strategy.md) - Strategy TOML schema, discovery, CLI surface, FC-IS architecture
- [ralph strategy sync](docs/context/ralph-strategy-sync.md) - Project scaffolding, bundled assets, idempotent sync behavior
- [Conversation Loop Strategy](docs/context/conversation-loop.md) - Human-driven conversation loop, human-in-the-loop directives, transcript types, fiction team

## Glossary

Terms that only make sense in the context of the project should be defined in this table for easy access.

| Term      | Definition                                   |
| --------- | -------------------------------------------- |
| **ralph** | Main binary crate - the main cli application |

---
> Source: [cloudbridgeuy/ralph](https://github.com/cloudbridgeuy/ralph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
