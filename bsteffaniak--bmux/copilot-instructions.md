## bmux

> This file defines REQUIRED validation steps for coding agents working in this repo.

# AGENTS.md

This file defines REQUIRED validation steps for coding agents working in this repo.

## Core Architecture Boundary (REQUIRED)

BMUX core must remain domain-agnostic. Windows and permissions are plugin domains, not core architecture.

### Hard Rules

- Do not add or keep windows-domain or permissions-domain logic in core architecture layers.
- Core architecture includes at least:
  - `packages/server/**`
  - `packages/client/**`
  - `packages/ipc/**`
  - `packages/session/**`
  - `packages/terminal/**`
  - `packages/event/**`
- In core architecture, avoid domain-specific types/fields/events/APIs for windows or roles/permissions.
- Domain behavior must be implemented through plugins and generic plugin/service invoke paths.
- Core defaults when plugins are missing:
  - Missing windows plugin: baseline single terminal attach/session/pane flow still works.
  - Missing permissions plugin: permissive single-user behavior.

### Plugin Power Expectations

- Plugins are first-class and may implement critical product behavior.
- Prefer extending generic plugin APIs/capabilities over adding core-special-case code.
- If a feature seems domain-specific, place it in a plugin unless there is a strong, documented reason it must be core-agnostic runtime plumbing.

### Review Gate Before Finishing (REQUIRED)

For any non-doc code change, verify no forbidden domain leakage was introduced in core architecture:

- Run content checks (or equivalent) to confirm no new core references to windows/permissions domain concepts.
- If any are found, treat as blocking and refactor before finishing.

These boundary rules are strict and take precedence over convenience.

## Core Rule

If you change code, run the relevant tests before finishing and report exactly what you ran and whether it passed.

In addition, for any code change anywhere in the repo, run:

- `cargo nextest run --no-fail-fast`

This is required. Treat failures as blocking, and do not finish with known flaky/failing tests.

## Clippy (REQUIRED)

For any code change anywhere in the repo, run:

- `cargo clippy --all-targets -- -D warnings`

This must produce zero errors and zero warnings. Treat any warning as blocking.

Key rules when fixing clippy issues:

- Fix the root cause. Do not add crate-level `#![allow(clippy::...)]` to suppress warnings.
- Item-level `#[allow(clippy::...)]` is acceptable only when:
  - The lint is a genuine false positive for that specific item (e.g., `too_many_lines` on a state machine function that would be less readable if split).
  - The lint cannot be satisfied without breaking the API or reducing clarity (e.g., `cast_possible_truncation` on a bounded numeric conversion in rendering code).
  - A comment explains why the allow is needed.
- Never use `cargo clippy --fix` without verifying it compiles and tests pass afterward -- auto-fixes can break code in macro contexts and async functions.
- Wildcard imports (`use something::*`) are not allowed in production code. They are acceptable in `#[cfg(test)] mod tests` blocks with `use super::*`.

## Cargo Machete (REQUIRED)

For any code change anywhere in the repo, run:

- `cargo machete --with-metadata`

This must report zero unused dependencies. Treat any finding as blocking.

Key rules when addressing unused dependency findings:

- Remove the dependency from `Cargo.toml` if it has no source code references.
- Also remove any feature propagation entries (e.g., `"dep_name/fail-on-warnings"`) for removed deps.
- If a dependency is only used for feature propagation (not imported in source code), remove both the dep and propagation -- transitive deps handle their own features.
- Use `[package.metadata.cargo-machete] ignored = [...]` only for genuine false positives:
  - Dependencies used via proc macro code generation (e.g., `container!` generating `::crate_name::Type` paths).
  - Dev-dependencies that cargo-machete cannot trace into test code.
  - A comment in the `ignored` list explains why the ignore is needed.

## Plugin Changes (REQUIRED)

If a change touches `plugins/**`, rebuild bundled plugins before finishing:

- `bmux plugin rebuild`

During iteration you may run targeted rebuilds with selectors (plugin id, short name, or crate name), but final validation for plugin changes should run the command without selectors.

If `bmux plugin rebuild` is unavailable in the current environment, run direct cargo builds for plugin crates (for example, `cargo build -p <plugin-crate> ...`).

## Minimum Required Commands (CLI/runtime work)

For changes in `packages/cli/**` (runtime, input, pane/layout, protocol, terminal handling), run:

1. `cargo nextest run --no-fail-fast`
2. `cargo test -p bmux_cli`
3. `cargo check -p bmux_cli`
4. `./scripts/smoke-pty-runtime.sh`

## Compatibility-Related Changes (REQUIRED extra check)

If the change touches terminal protocol behavior, TERM/profile logic, or query/reply handling, also run:

5. `./scripts/compat-matrix.sh`

This includes edits under:

- `packages/cli/src/runtime/terminal_protocol.rs`
- `packages/cli/src/runtime/pane_runtime.rs`
- `packages/cli/src/runtime/mod.rs` (terminal doctor / profile / term resolution)
- `packages/config/src/lib.rs` (`behavior.pane_term`, trace flags, profile settings)

## Change-to-Tests Mapping

- Protocol/query/reply/TERM/profile changes
  - Run all 5 commands above.
- Input/keymap/runtime command handling changes
  - Run 1-4.
- Layout/pane lifecycle/compositor changes
  - Run 1-4.
- Config-only changes
  - Run at least 1-3; include 4 if behavior affects runtime startup.
- Any plugin changes under `plugins/**`
  - Run `bmux plugin rebuild`.
- Docs-only changes
  - No mandatory runtime commands.

In addition, for any non-doc code change, always run:

- `cargo nextest run --no-fail-fast`
- `cargo clippy --all-targets -- -D warnings`
- `cargo machete --with-metadata`

## Completion Reporting Format

Agents should report test execution in final response using this format:

- `cargo test -p bmux_cli` - PASS/FAIL
- `cargo check -p bmux_cli` - PASS/FAIL
- `cargo nextest run --no-fail-fast` - PASS/FAIL
- `cargo clippy --all-targets -- -D warnings` - PASS/FAIL
- `cargo machete --with-metadata` - PASS/FAIL
- `./scripts/smoke-pty-runtime.sh` - PASS/FAIL
- `./scripts/compat-matrix.sh` - PASS/FAIL (if required)
- `bmux plugin rebuild` - PASS/FAIL (if required)

If any required command is skipped, explain why.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BSteffaniak) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
