## baker

> Source lives under `src/`, grouped by responsibility: `cli/` for argument parsing and orchestration, `template/` for rendering and filesystem operations, `prompt/` for interactive flows, and `config/` for loading `baker.yaml`. Shared utilities sit in `constants.rs`, `error.rs`, and `ext/`. Integration-style scenarios reside in `tests/`, sample project templates in `examples/`, and static assets under `assets/`. Generated artifacts land in `target/` (Rust default) or `out/` for packaged builds—keep these paths out of commits.

# Repository Guidelines

## Project Structure & Module Organization
Source lives under `src/`, grouped by responsibility: `cli/` for argument parsing and orchestration, `template/` for rendering and filesystem operations, `prompt/` for interactive flows, and `config/` for loading `baker.yaml`. Shared utilities sit in `constants.rs`, `error.rs`, and `ext/`. Integration-style scenarios reside in `tests/`, sample project templates in `examples/`, and static assets under `assets/`. Generated artifacts land in `target/` (Rust default) or `out/` for packaged builds—keep these paths out of commits.

## Build, Test, and Development Commands
- `cargo build`: compile the CLI and library. Add `--release` when benchmarking packaging. 
- `cargo run -- <template> <output>`: execute Baker against a local template, helpful when smoke-testing changes (e.g., `cargo run -- examples/demo tmp/demo`).
- `cargo test`: run unit and integration tests; CI expects a clean pass. Combine with `-- --ignored` if you add ignored cases. 
- `cargo fmt` and `cargo clippy --all-targets --all-features`: enforce formatting and lint rules before opening a PR.

## Coding Style & Naming Conventions
Follow Rustfmt defaults (4-space indentation, trailing commas, use blocks in control flow). Prefer `snake_case` for functions and variables, `UpperCamelCase` for types and traits, and module-per-file organization mirroring the existing directory layout. Use descriptive log messages—`log::info!` for user-visible output, `log::debug!` for trace data. Keep documentation comments concise and focused on behaviour rather than implementation trivia.

## Testing Guidelines
Unit tests usually sit beside modules (see `src/template/operation.rs`), while higher-level flows belong in `tests/`. Name tests after behaviour (`renders_loop_templates`) and avoid brittle filesystem assertions by using `tempfile::TempDir`. Ensure new features include coverage and run `cargo test` locally before pushing.

## Commit & Pull Request Guidelines
The history follows Conventional Commits (`chore(deps): ...`, `feat: ...`, `fix: ...`). Apply the same style and keep subjects under ~60 characters. For pull requests, include a brief description of the change set, note any manual verification (commands run, template generated), link related issues, and add screenshots or logs when altering user-facing behaviour. Small, focused PRs are easier to review; split large refactors into staged commits when possible.

## Architecture Overview
Baker’s flow is: CLI parses options → config loader fetches `baker.yaml` → prompt layer collects answers → template processor renders or copies files → optional hooks run pre/post generation. When contributing, respect this separation and favour adding new behaviour within the appropriate layer instead of cross-cutting shortcuts.

---
> Source: [aliev/baker](https://github.com/aliev/baker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
