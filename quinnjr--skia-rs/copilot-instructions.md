## commands

> Common workspace commands for skia-rs development


# Workspace Commands

## Development

```bash
# Check all crates
cargo check --workspace

# Build debug
cargo build --workspace

# Build release
cargo build --release --workspace

# Run tests
cargo test --workspace

# Run benchmarks
cargo bench -p skia-rs-bench

# Generate docs
cargo doc --workspace --no-deps --open

# Format code
cargo fmt --all

# Lint
cargo clippy --workspace -- -D warnings
```

## Git Workflow

- Keep the `skia/` submodule updated for reference
- Commit messages should reference Skia types/functions being implemented
- Use feature branches for new implementations
- Update `TODO.md` as features are completed

## Conventional Commits

Commit messages must follow Conventional Commits:

```
feat(core): add Matrix inversion
fix(path): correct cubic curve bounds calculation
docs: update API documentation
refactor(canvas): simplify save/restore stack
test(paint): add property tests for color conversion
```

## Pre-commit Hooks

The project uses `cargo-husky` for git hooks:

- **pre-commit**: Runs `cargo fmt --check` and `cargo clippy`
- **pre-push**: Runs full test suite
- **commit-msg**: Validates Conventional Commits format

---
> Source: [quinnjr/skia-rs](https://github.com/quinnjr/skia-rs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
