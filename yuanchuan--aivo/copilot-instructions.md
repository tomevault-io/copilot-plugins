## aivo

> `aivo` is a Rust CLI tool providing unified access to multiple AI coding assistants (Claude, Codex, Gemini, OpenCode, Pi) with local API key management and secure storage. Supports OpenAI-compatible providers, GitHub Copilot, OpenRouter, Ollama, and native APIs.

# CLAUDE.md

## Project Overview

`aivo` is a Rust CLI tool providing unified access to multiple AI coding assistants (Claude, Codex, Gemini, OpenCode, Pi) with local API key management and secure storage. Supports OpenAI-compatible providers, GitHub Copilot, OpenRouter, Ollama, and native APIs.

> [!IMPORTANT]
> **Rebuild before testing**: After code changes, always run `cargo build && cargo install --path . --debug` before testing the binary. Never test a stale build. Only use `--release` for final testing before a release.

## Build & Test

```bash
cargo build                             # Debug build (~1s incremental)
cargo test --features test-fast-crypto  # All tests (~1900, fast crypto)
cargo test -- test_name                 # Single test
cargo clippy                            # Lint (fix all warnings before committing)
cargo fmt                               # Format (run before committing)
```

`test-fast-crypto` uses reduced PBKDF2 iterations. A `Makefile` wraps common workflows: `make test`, `make build`, `make clippy`, `make install`, `make release`.

## Git Conventions

- Always squash merge to main: `git merge --squash <branch> && git commit`
- Do not commit automatically to the fix.

## Release Process

1. Bump version in `Cargo.toml` and `npm/package.json` first — never tag without updating.
2. Run `cargo fmt`, `cargo clippy -- -D warnings`, `cargo test`.
3. `cargo build --release && cargo install --path .` to verify.
4. Commit: `git add -A && git commit -m "chore: release vX.Y.Z"`
5. Tag and push: `git tag vX.Y.Z && git push origin main --tags`

## CLI / UX Conventions

Match existing CLI help text formatting exactly (alignment, spacing, bracket style). When implementing interactive UI, verify: keyboard handling (arrows, Ctrl+P/N, ESC, Ctrl+C), selection state pre-selection, column alignment, and edge cases (empty input, single item, long strings).

## Architecture

```
src/main.rs → SessionStore → EnvironmentInjector → AILauncher → Command Handlers
```

- **`src/`**: Entry point, CLI parsing, error handling, TUI components, styling
- **`src/commands/`**: `run` (launch tools), `start` (interactive picker), `chat` (chat TUI + one-shot), `keys`, `serve`, `info`, `models`, `alias`, `logs`, `stats`, `update`
- **`src/services/`**: Session/key/stats storage, AI process launching, provider routing/bridging (Anthropic, OpenAI, Gemini, Copilot, Ollama), model name transforms, HTTP utilities

**Data model**: `ApiKey` (`id`, `name`, `base_url`, `key`, `created_at`) stored AES-256-GCM encrypted in `~/.config/aivo/config.json`. Sentinel `base_url` values `"copilot"` and `"ollama"` identify special provider types.

**Cross-platform**: Platform-specific code gated behind `cfg(unix)` / `cfg(windows)`.

**Exit codes**: 0 = success, 1 = user error, 2 = network, 3 = auth.

## Instructions

Restate the question in fully concrete terms, making every implicit detail explicit. Then answer.

---
> Source: [yuanchuan/aivo](https://github.com/yuanchuan/aivo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
