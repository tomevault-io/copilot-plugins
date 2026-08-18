## tuxcleaner

> TuxCleaner is a safety-first Linux maintenance CLI and TUI written in Rust. All source code, user-facing CLI text, documentation, issues, and pull request content must be written in English.

# CLAUDE.md

## Project overview

TuxCleaner is a safety-first Linux maintenance CLI and TUI written in Rust. All source code, user-facing CLI text, documentation, issues, and pull request content must be written in English.

## Required validation

Run these commands before considering a change complete:

```bash
cargo fmt --all -- --check
cargo clippy --all-targets --all-features -- -D warnings
cargo test --all-targets
sh -n install.sh scripts/package-release.sh scripts/render-demo.sh scripts/render-social-preview.sh docs/demo/fixtures/bin/*
sh tests/install.sh
```

## Safety invariants

- Scanning must remain read-only.
- Every destructive command must require explicit selection or `--yes`.
- Every destructive command must support `--dry-run`.
- Do not execute shell command strings. Use typed actions and fixed argument arrays.
- Add external commands to the executor allowlist with exact arguments.
- Never delete `/`, the home directory, `.git`, `.config`, `.ssh`, or `.gnupg`.
- Never follow symlinks while scanning or deleting.
- Docker volumes are never part of group cleanup.
- Large personal files and hidden application data are reported only, never deleted by `analyze`.
- System operations may use narrow `sudo` commands. User cleanup must not use sudo.
- Self-updates must use exact release assets and verify SHA-256 before replacement.
- Application IDs must come from the current catalog and match their source and package exactly.
- Native application uninstall must show a package-manager transaction preview before execution.
- Application configuration and user data are outside uninstall scope.

## Architecture

- Keep distribution-specific behavior in `src/distro.rs`.
- Keep discovery separate from execution.
- Keep application discovery and protected-package policy in `src/uninstall.rs`.
- Model cleanup operations with `CleanupAction` rather than direct filesystem calls from scanners.
- Keep path and command validation in `src/executor.rs`.
- Preserve JSON output compatibility by adding fields rather than renaming or removing them.
- Add fixture and refusal tests for every new distribution or cleanup rule.

## Git preferences

- Do not add co-author trailers unless explicitly requested.
- Do not use em dashes in public-facing writing.
- Keep commits focused and use imperative English commit subjects.

---
> Source: [debba/tuxcleaner](https://github.com/debba/tuxcleaner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
