## swarm

> - Group imports so there is exactly one `use` per crate.

# AGENTS

## Coding Style

- Group imports so there is exactly one `use` per crate.
- For `std`, prefer a single grouped import such as `use std::{...};` instead of multiple `use std::...;` lines.
- Keep imports explicit and stable. Do not use wildcard imports.
- Follow the existing command layout:
  - top-level CLI parsing in `opts.rs`
  - command handlers in `cmd/`
  - storage and domain logic in top-level modules such as `repos.rs`

---
> Source: [penberg/swarm](https://github.com/penberg/swarm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
