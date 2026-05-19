## oxidechat

> - Avoid unnecessary `clone()` calls - prefer borrowing with `&T` or `&mut T`

# AGENTS.md

## Performance Guidelines

- Avoid unnecessary `clone()` calls - prefer borrowing with `&T` or `&mut T`
- Use `&str` instead of `String` in function parameters unless ownership is required
- Only clone `Arc<T>` when spawning new tasks - Arc cloning is cheap but still unnecessary if not spawning
- `PgPool` is already Arc-based internally - never clone it, just use `&pool`
- Preallocate `Vec` capacity with `Vec::with_capacity(n)` when size is known
- Use iterator adapters (`.map()`, `.filter()`, etc.) instead of manual loops where possible
- Prefer `&[T]` over `&Vec<T>` in function parameters
- Use `Cow<str>` when you might need to own or borrow strings conditionally
- Avoid `to_string()` and `to_owned()` unless actually transferring ownership
- Use `query_as` or `query_as!` for type-safe database queries
- Batch database operations - avoid N+1 queries by using `ANY($1)` or `IN` clauses
- Use database transactions for multi-step operations that must be atomic
- Return `Result<T, E>` instead of unwrapping - never panic in production code
- Use `async move` blocks only when necessary - prefer borrowing in async functions

## Architecture

- `src/main.rs` - Application entry point, sets up Axum server and database pool
- `src/i18n.rs` - Global i18n translation service (singleton pattern)
- `src/jobs.rs` - Background job scheduler using tokio tasks
- `src/types/` - Shared request/response DTOs and domain types
- `src/tests/` - Unit tests organized by module (e.g., `i18n.rs`, `base.rs`)
- `src/routes/` - HTTP route handlers (handlers only, no type definitions)
- `migrations/` - SQLx database migrations
- `JobState` struct contains shared application state (database pool)
- Wrap `JobState` in `Arc<JobState>` for sharing across async tasks
- Job scheduler spawns independent tokio tasks for each scheduled job
- Each job is a function returning `JobFuture` (pinned boxed future)
- Jobs share state via `Arc<JobState>` - clone the Arc when spawning, not the pool

## Code Organization

### Types Module (`src/types/`)

- All request/response structs live in `src/types/`, not in the code files files
- Organized by domain: `i18n.rs`, `base.rs`, etc.
- `mod.rs` re-exports all types for easy importing: `use crate::types::*;`
- Each file uses section comments for clarity:
    - Request Types (structs with `Deserialize`)
    - Response Types (structs with `Serialize`)
    - Internal Types (DB row structs, helpers)

### Tests Module (`src/tests/`)

- Unit tests live in `src/tests/`, not inline in route files
- One test file per module: `i18n.rs`, `base.rs`
- Use `#[cfg(test)]` wrapper for all test code
- Group related tests in submodules: `mod types { ... }`
- Import types via `crate::types::*`

### Route Files (`src/routes/`)

- Contain only handler functions, no struct definitions
- Import types from `crate::types::{...}`
- Keep handlers thin: validate → delegate → respond

### Data Model Pattern

* **Structure:** Locate domain models and their `impl` blocks in `src/types/` (e.g., `user.rs`).
* **Logic:** Use associated functions (`::`) for queries/constructors and instance methods (`.`) for operations on specific records.
* **State:** Pass `&PgPool` explicitly as a parameter; never rely on global database state.
* **Mapping:** Implement `to_response` methods within the model to handle DTO transformations.
* **Encapsulation:** Keep all SQL logic and business rules for a type within its dedicated file.

## Code Quality

- Clippy lints are enforced: `correctness` and `suspicious` are denied, `perf` and `style` are warnings
- Cognitive complexity threshold: 15
- Max function arguments: 5 (use structs for more)
- Max function lines: 100
- Document public APIs with `///` doc comments
- Include `# Errors` and `# Panics` sections in documentation where applicable
- Use `#[must_use]` for functions whose return value should not be ignored
- Prefer explicit types over `_` in public APIs
- Use `thiserror` or similar for custom error types
- Do not use static strings, use the i18n system instead

## Testing

- Use `#[sqlx::test]` for database tests - provides isolated test database
- Use `#[tokio::test]` for async tests
- Tests live in `src/tests/` with one file per module
- Group related tests in submodules: `mod types { }`, `mod handlers { }`
- Clean up test data or use transactions that rollback
- Test error cases, not just happy paths
- Use `Result<(), Error>` return type in tests for cleaner error propagation

## Frontend
- Use Tailwind classes for styling
- Use `useMainStore()` for global state management
- Use Shadcn UI for components (shad-vue)
- All ShadCN components are prefixed with `Shad`, e.g., `ShadButton` or `ShadSelect`, they are auto injected
- Use the store.getTranslation() function for translations, always use them, no static strings
- Use customizable Tailwind classes like bg-primary, text-primary, etc. and not use fixed colors

## Important Notes

- Always use parameterized queries (`$1`, `$2`) to prevent SQL injection
- Never use `unwrap()` or `expect()` in production code paths
- Job scheduler runs indefinitely - ensure jobs handle errors gracefully
- Database URL must be set in `.env` file as `DATABASE_URL`
- Migrations run automatically on startup
- Keep the code as simple as possible so beginners can also understand it
- Do not add comments to the code itself
- When writing axum routes, please use the response "module" to create responses
- NO SQL statements in route handlers, please add a function to the specific type's module for database operations (check if a smilar function already exists)

## Updating This Document

- Update this file when adding new modules or restructuring code
- Update when introducing new shared state patterns
- Update when adding performance-critical patterns
- Update when changing error handling strategies
- Update when modifying the job scheduler or testing approach

---
> Source: [NxtCore/OxideChat](https://github.com/NxtCore/OxideChat) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
