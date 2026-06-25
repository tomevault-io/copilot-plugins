## longbridge-mcp

> Use `cargo +nightly fmt` for code formatting.

## Code formatting

Use `cargo +nightly fmt` for code formatting.

## Code style

### Generic bounds

Place generic bounds in `where` clauses rather than inline.

### Imports

- Use `crate::` instead of `super::` in non-test code. `use super::*` is acceptable in `#[cfg(test)]` modules.
- Declare imports at the top with `use` so the body uses short names. Avoid long inline paths.

### Comments

Do not use decorative divider comments (e.g. `// ── Section name ───────`). If a file needs clearer structure, split it into separate modules or files instead.

### Panic messages

`panic!()`, `unreachable!()`, `debug_assert!()`, and `assert!()` must always include a descriptive message string.

### Error handling

- Prefer `Result` over panics.
- Use `thiserror` for custom error types.

### Clippy

Run `cargo clippy --all-features --all-targets` and fix all warnings before committing.

### Documentation

- Public items should have doc comments.
- Use `//!` for module-level documentation, `///` for item documentation.

## Commit messages

Follow Conventional Commits: `<type>(<scope>): <description>`

Types: feat, fix, docs, style, refactor, test, chore

---
> Source: [longbridge/longbridge-mcp](https://github.com/longbridge/longbridge-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-25 -->
