## nanemu

> Rust sources live in `src/`, split by pipeline stage: `isa/` for the parser/AST, `core/` for CPU construction, `system/` for SoC graphs, `decode/` for generated trees, and `exec/` for the interpreter. Declarative architecture inputs belong in `defs/core` and `defs/system`, while integration fixtures sit in `tests/` (e.g., `tests/powerpc`). Reference docs, design notes, and research live in `docs/` and `research/`; temporary experiments stay inside `obsolete/` to avoid polluting releases.

# Repository Guidelines

## Project Structure & Module Organization
Rust sources live in `src/`, split by pipeline stage: `isa/` for the parser/AST, `core/` for CPU construction, `system/` for SoC graphs, `decode/` for generated trees, and `exec/` for the interpreter. Declarative architecture inputs belong in `defs/core` and `defs/system`, while integration fixtures sit in `tests/` (e.g., `tests/powerpc`). Reference docs, design notes, and research live in `docs/` and `research/`; temporary experiments stay inside `obsolete/` to avoid polluting releases.

## Build, Test, and Development Commands
Use `cargo check` for a fast validation of new code, then `cargo build --all-targets` when you need binaries and generated IR. `cargo test` runs unit + integration suites; target a single area with `cargo test core::` or `cargo test --test powerpc_core`. Format Rust code via `cargo fmt` and lint deeper logic with `cargo clippy --workspace --all-targets`. Run `cargo doc --open` before publishing APIs so the DSL surface stays documented.

## Coding Style & Naming Conventions
Follow standard Rustfmt defaults (4-space indentation, 100-column soft limit). Modules/files use `snake_case.rs`, types and traits use `UpperCamelCase`, and functions plus locals stay in `snake_case`. Keep DSL examples concise: `.isa` and `.coredef` filenames mirror the architecture tag (e.g., `tricore_162.coredef`). Always add module-level comments when introducing new IR opcodes or execution semantics.

## Testing Guidelines
Unit tests live beside implementations under `src/**/tests`. Integration flows go in `tests/` and should name files after the architecture under test (`powerpc_vle.rs`). Prefer table-driven cases that exercise both the declarative specs and the interpreter. When adding parser features, pair them with fixture documents inside `defs/` plus targeted assertions in `tests/mod.rs`. Run `cargo test -- --nocapture` if you need trace output, and keep perf-sensitive tests behind `#[ignore]` gates.

## Commit & Pull Request Guidelines
History shows short, imperative summaries (e.g., “Created core harness for writing to core”). Keep the first line under 72 chars and expand context in the body only when necessary. Every PR should describe the affected ISA/system modules, include reproduction steps or commands executed, and link any tracking issues. Attach before/after traces or screenshots when touching decoder output so reviewers can validate semantics quickly.

---
> Source: [lancelot2112/nanemu](https://github.com/lancelot2112/nanemu) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
