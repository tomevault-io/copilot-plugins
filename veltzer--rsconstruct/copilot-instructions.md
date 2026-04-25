## rsconstruct

> A fast, incremental build tool written in Rust with tera support, Python linting, and parallel execution.

# RSConstruct - Rust Build Tool

A fast, incremental build tool written in Rust with tera support, Python linting, and parallel execution.

Detailed documentation is in `docs/src/`. Key references:
- Commands: `docs/src/commands.md`
- Configuration: `docs/src/configuration.md`
- Architecture (subprocess execution, path handling, caching): `docs/src/architecture.md`
- Processor contract: `docs/src/processor-contract.md`
- Coding standards: `docs/src/coding-standards.md`
- Per-processor docs: `docs/src/processors/`

## Philosophy

- **Simplicity first** — keep the code simple whenever possible. Avoid clever solutions that are hard to understand or maintain. When in doubt, choose the straightforward approach.
- **Convention over configuration** — simple naming conventions, explicit config loading, incremental builds by default.
- **No macros** — the goal is zero `macro_rules!` in the codebase. Use regular functions, generics, traits, and structs to eliminate duplication. Every existing macro is a refactoring target to be replaced with plain Rust. Do not add new macros.
- **Cross-platform via `src/platform.rs`** — all OS-specific code (file permissions, signal handling) lives in `src/platform.rs` behind `#[cfg(unix)]` / `#[cfg(not(unix))]` guards. The rest of the codebase calls these wrappers and stays platform-agnostic. Do not add `#[cfg(...)]` blocks outside of `platform.rs`.
- **Strict by default** — never silently skip errors or ignore failures. Non-strict systems hide problems and are a disaster. If a tool is missing, fail. If a test fails, fix it before moving on.
- **All tests must pass** — always run `cargo test` with no filters or skips. Do not move forward with any failing test. If a test fails, fix it immediately — the failure is real.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/veltzer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
