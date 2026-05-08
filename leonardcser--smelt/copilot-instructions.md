## smelt

> cargo test --workspace

## Commands

```bash
# build
cargo build

# test
cargo test --workspace

# format and lint
cargo fmt && cargo clippy --workspace --all-targets -- -D warnings
```

Whenever you add a new user-facing feature or change user-facing behavior,
update the README.md and the docs/ folder. Don't document internal
implementation details — only things end users need to know.

---
> Source: [leonardcser/smelt](https://github.com/leonardcser/smelt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
