## project-template-rs

> This file provides guidance to AI coding assistants when working with code in

# AGENTS.md

This file provides guidance to AI coding assistants when working with code in
this repository.

## Overview

Rust monorepo template for starting new GitHub repositories. Uses a Cargo
workspace with crates under `crates/`. The example crates are placeholders meant
to be replaced.

## Architecture

- **Workspace root** (`Cargo.toml`): Defines shared package settings and lints
  inherited by all crates.
- **Crates** (`crates/`): Each crate inherits workspace configuration via
  `*.workspace = true`.
- **Toolchain** (`rust-toolchain.toml`): Pins a specific Rust version with
  required components.
- **CI** (`.github/workflows/rust.yml`): Runs fmt, clippy (warnings as errors),
  and tests across multiple OS/toolchain combinations.

## Lints

The workspace configures strict clippy and rustc lints in `Cargo.toml`. Read
`[workspace.lints.rust]` and `[workspace.lints.clippy]` before writing code.
Many common patterns (unwrap, indexing, unsafe, as-casts, print macros) are
restricted.

## Notes

- `Cargo.lock` is gitignored (library-oriented template). Remove it from
  `.gitignore` if the project becomes a binary/application.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/viell-dev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/viell-dev)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
