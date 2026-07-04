## agentdoctor

> AgentDoctor is a Rust 2024 CLI-first workspace that audits repositories for AI

# Agent Instructions

## Project overview
AgentDoctor is a Rust 2024 CLI-first workspace that audits repositories for AI
coding agent readiness. The project must stay clean, deterministic,
cross-platform, and maintainable. Treat this as production-quality developer
tooling, not a prototype.

## Operating principles
- Prefer the professional, durable solution over a quick patch.
- Keep changes scoped to the requested behavior and avoid unrelated refactors.
- Read the relevant code before editing; do not guess when the repository can
  answer the question.
- Plan first for ambiguous or multi-file work, then implement.
- Do not leave dead code, fake stubs, unused helpers, TODO-heavy placeholders,
  or partial implementations that pretend to be complete.
- Make behavior verifiable with tests or a clear command-level check.
- Preserve deterministic output for generated files and reports.

## Stack
- Rust 2024 workspace.
- `clap` for CLI parsing.
- `serde`, `serde_json`, and `toml` for serialization.
- `ignore` for repository walking.
- `directories` for platform-native config paths.
- `assert_cmd`, `tempfile`, and fixture repos for tests.

## Commands
- format: `cargo fmt --all`
- format check: `cargo fmt --all -- --check`
- lint: `cargo clippy --workspace --all-targets -- -D warnings`
- test: `cargo test --workspace`
- build: `cargo build --workspace --release`

## Project structure
- `crates/agentdoctor-engine/` owns scanning, rule execution, scoring, and
  deterministic init generation.
- `crates/agentdoctor-config/` owns global and project configuration loading,
  saving, and precedence.
- `crates/agentdoctor-report/` renders already-built reports only.
- `crates/agentdoctor-cli/` owns CLI parsing, prompts, terminal output, exit
  codes, and user-facing errors.
- `fixtures/` contains regression repositories used by tests.
- `scripts/`, `install.sh`, and `install.ps1` contain installer helpers.
- `.github/workflows/` contains CI and release automation.

## Architecture boundaries
- Do not put scanning, rule, scoring, generation, or report-building logic in
  the CLI crate.
- Do not make report rendering inspect the filesystem or run scan logic.
- Do not make config code print, prompt, scan, or render reports.
- Keep library crates free of terminal styling, prompts, and process exits.
- Add abstractions only when they remove real duplication or clarify ownership.
- Prefer explicit domain types and typed errors in library crates.

## Code quality
- Use `#![forbid(unsafe_code)]` in every Rust crate.
- Do not add `unsafe`.
- Do not use `.unwrap()` or `.expect()` in production code unless the invariant
  is obvious and the reason is documented.
- Do not panic for normal user input, filesystem state, invalid config, or CLI
  mistakes.
- Use `Path` and `PathBuf`; never string-concatenate filesystem paths.
- Keep modules focused. Do not create broad `utils.rs` dumping grounds.
- Avoid large files and unrelated helper functions.
- Keep public APIs small, typed, and stable enough for the CLI, future TUI, and
  release automation to reuse.
- Do not add heavy dependencies, network clients, async runtimes, AI SDKs, or
  parser frameworks without a clear product need.

## Testing standards
- Add or update tests for every behavior change.
- Prefer engine fixture regression tests for scan, rule, score, and generation
  behavior.
- When adding an agent surface such as skills, prompts, custom agents, hooks,
  MCP config, or tool-specific instruction files, add real filesystem-based
  tests that prove discovery, rule findings, score evidence, and JSON-safe
  output behavior.
- Use CLI integration tests for command behavior, stdout/stderr contracts, exit
  codes, config precedence, and no-write guarantees.
- Use report tests for JSON shape and pretty output grouping.
- Golden output tests are appropriate for deterministic generated files.
- Do not weaken or delete tests to make a change pass. Fix the implementation or
  update expectations only when the behavior intentionally changed.

## Security and privacy
- Do not add telemetry.
- Do not call AI APIs.
- Do not perform network access during scans.
- Do not read, print, or copy secret values from `.env` files.
- Treat `.env`, credentials, tokens, private keys, and local machine paths as
  sensitive.
- Generated content must not include secrets or machine-local private values.

## Installer and release rules
- Keep Windows, macOS, and Linux install paths documented and tested where
  practical.
- Do not break `install.ps1`, `install.sh`, or `scripts/install-local.ps1`
  behavior when changing release artifacts.
- Keep upgrade network access isolated to lifecycle commands. Scans must remain
  offline and deterministic.
- Keep uninstall confirmation-gated; non-interactive uninstall must require an
  explicit `--yes`.
- Release archives must include the binary, README, and LICENSE.
- CI must stay strict: fmt, clippy with `-D warnings`, tests, and release build.

## Before finishing
- Run `cargo fmt --all -- --check` after Rust changes.
- Run `cargo clippy --workspace --all-targets -- -D warnings` after Rust
  changes.
- Run `cargo test --workspace` after behavior, test, config, report, CLI, or
  fixture changes.
- Run `cargo build --workspace --release` after CLI, dependency, release, or
  installer changes.
- For docs-only changes, run the smallest relevant validation and state what was
  not run.
- Verify the installed or built CLI manually when changing install, release, or
  command behavior.
- Final responses must clearly state what changed and which checks were run.

---
> Source: [youssefsz/agentdoctor](https://github.com/youssefsz/agentdoctor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-04 -->
