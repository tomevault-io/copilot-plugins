## memex

> Always run these checks before committing. Do not commit if either fails.

# Agents

## Before committing

Always run these checks before committing. Do not commit if either fails.

```bash
cargo fmt --check
cargo clippy -- -D warnings
```

If `cargo fmt --check` fails, run `cargo fmt` and include the formatting fix in your commit.

---
> Source: [nicosuave/memex](https://github.com/nicosuave/memex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
