## gguf-runner

> This repository is a pure GGUF model runner/inference engine implemented as a binary crate.

# AGENTS.md

## Scope

This repository is a pure GGUF model runner/inference engine implemented as a binary crate.
All contributors/agents must preserve the current architecture boundaries and keep documentation in sync with code.

## Architecture Overview

- Entrypoint: `src/main.rs`
- Orchestration: `src/app/`
- CLI and env parsing: `src/cli.rs`
- Core inference/runtime: `src/engine/`
- Vendor/model-family adapters: `src/vendors/`

Dependency direction to keep:

- `main -> app -> {cli, vendors, engine}`
- `vendors -> engine::types` (or shared engine domain types only)
- `engine` must not depend on `app` or `cli`

## Module Placement Rules

- Add CLI flags/env vars only in `src/cli.rs`.
- Keep orchestration/business flow in `src/app/`.
- Put inference/runtime kernels/state/loading in `src/engine/`.
- Put family-specific config/prompt logic only in `src/vendors/`.

## Vendor Boundary Rules

- Do not add model-family branches in generic runtime flow (`src/app/*`, `src/engine/*`) such as checks on `config.is_*` or hardcoded vendor token literals.
- Route vendor-specific runtime behavior through vendor policies (for example decode/tokenizer/multimodal policy structs in `src/vendors/mod.rs`) and consume those policies generically.
- If a new family needs special decode behavior, add it as a policy field and set it in the relevant `src/vendors/*` module instead of branching in app/engine code.

## Import and Coupling Rules

- Do not use `use crate::*`.
- In `src/engine/*`, do not use wildcard imports from crate modules (for example `use crate::engine::types::*`).
- Keep `engine` independent from `cli` types.
- `std::arch::*` wildcard imports are acceptable only for architecture intrinsics in SIMD code paths.

## Documentation Requirements (`docs/`)

When architecture-related code changes:

1. Update `docs/module-structure.md` in the same change.

Use these docs as source of truth:

- `docs/module-structure.md`: current module snapshot and responsibilities.

## Required Validation Before Completion

Run these commands from repo root:

1. `cargo fmt --all --check`
2. `cargo clippy --all-targets --all-features`
3. `cargo check`
4. `rg -n "^use crate::\\*;" src/engine -g'*.rs'`
5. `rg -n "^use crate::engine::[A-Za-z0-9_:]+::\\*;" src/engine -g'*.rs'`
6. `rg -n "crate::cli::" src/engine -g'*.rs'`

If `cargo fmt --check` fails, run `cargo fmt --all` and re-run checks.

---
> Source: [apimeister/gguf-runner](https://github.com/apimeister/gguf-runner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
