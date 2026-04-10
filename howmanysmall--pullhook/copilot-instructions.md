## pullhook

> You are an expert at working with Rust CLI tools and have been onboarded to the `pullhook` codebase.

# AGENTS.md

You are an expert at working with Rust CLI tools and have been onboarded to the `pullhook` codebase.

## Project Overview

**pullhook** is a Rust CLI tool (binary: `pullhook`) that runs commands when files change after `git pull`.
It is a drop-in compatible replacement for the npm package `git-pull-run`, with bounded concurrency,
resilient diff base fallback, and safer command execution.

## Development Commands

### Build

```bash
cargo build
cargo build --release
cargo install --path .
```

### Test

```bash
cargo nextest run
cargo nextest run -E 'test(test_name)'
cargo nextest run --no-tests=pass
```

### Lint and Quality (run before pushing)

```bash
cargo fmt --all --check
cargo clippy --all-targets -- -D warnings
cargo audit
cargo deny check
cargo shear
```

### Lint and Quality (always run before finishing a task)

```bash
bash ./scripts/pre-commit.sh
```

### Benchmarks

```bash
cargo bench
```

### JS Tooling (formatting and commit lint support)

```bash
bun install
bun x --bun lint-staged
```

## Architecture

### End-to-End Data Flow

```text
CLI parse -> validate -> (--install? detect PM) -> git repo root
  -> resolve diff base -> get changed files -> compile glob pattern
  -> match files -> build task list -> run_parallel (rayon) -> report results
```

### Module Responsibilities (`src/`)

- **`main.rs`** - Entry point. Sets up tracing and signal handling (`ctrlc`), validates args, and orchestrates the full flow.
- **`cli.rs`** - `clap` derive-based CLI parsing and validation. `--install` implies `--once` and conflicts with `--pattern`/`--command`.
- **`git.rs`** - Git operations: repo root discovery, changed files (`diff --name-only`), and diff base resolution.
- **`matcher.rs`** - Glob matching via `globset` with a shim for bash extglob syntax.
- **`pm.rs`** - Package manager detection using lock files first, then config files, with ambiguity checks.
- **`runner.rs`** - Bounded parallel command execution with rayon, shell-word parsing, and optional `--shell` via `sh -c`.
- **`error.rs`** - Domain error types via `thiserror`; `anyhow::Result` is only used at the top level.

### Diff Base Resolution Chain

1. Explicit `--base` flag
2. `HEAD@{1}` (reflog)
3. `ORIG_HEAD` (merge/pull)
4. `HEAD~1` (fallback for shallow clones)

### Pattern and Matching Notes

Supported extglob forms:

- `+(a|b)` - one or more
- `*(a|b)` - zero or more
- `?(a|b)` - zero or one
- `@(a|b)` - exactly one
- `!(a|b)` - none of

Nested extglobs are not supported and should error. Extglobs are expanded into separate `GlobMatcher` instances.

### Package Manager Detection Notes

Two-phase detection in `pm.rs`:

1. Check lock files first and error on ambiguity.
2. Fall back to config files and default to npm when needed.

### Parallel Execution Notes

- Uses `rayon::ThreadPoolBuilder` with default jobs `min(CPUs, 8)`.
- Uses an `AtomicBool` shutdown flag checked before each task.
- Captures output per task to avoid interleaving.
- Returns results in task order (not completion order).

## Conventions

### Error Handling

- Prefer typed errors in `error.rs` using `thiserror`.
- Use `anyhow::Result` only at the top level (`main`).

### Linting and Safety

- `clippy::all`, `clippy::pedantic`, and `clippy::nursery` are enabled as warnings.
- `unsafe_code` is forbidden.

### Formatting

- Rust formatting is configured in `rustfmt.toml` (hard tabs, 120 width, 2024 formatting rules).
- JSON/JSONC formatting uses Biome.
- TOML formatting uses `tombi`.
- Markdown formatting uses `rumdl`.

### Toolchain and Build Performance

- Rust is pinned to `1.93.1` via `rust-toolchain.toml` (edition 2021).
- `sccache` is configured as the rustc wrapper in `.cargo/config.toml`.
- Incremental compilation is disabled (sccache is the primary cache strategy).
- No custom linker is configured by default.

### Commit Style

- Conventional commits are enforced by commitlint.

## Git Hooks (Lefthook)

Pre-commit hooks run in parallel:

- `bun x --bun lint-staged`
- `gitleaks`
- `cargo clippy --all-targets -- -D warnings`
- `cargo audit`
- `cargo deny check`
- `cargo shear`

Pre-push hook:

- `cargo nextest run --no-tests=pass`

## CI

Required checks in `.github/workflows/ci.yaml`:

- `fmt`
- `clippy`
- `nextest`
- `audit`
- `deny`
- `shear`
- `gitleaks`
- `docs`

Informational (PR-only, non-blocking):

- `cargo-bloat` binary size report
- `cargo-zigbuild` cross-compile smoke tests

## Key Dependencies

- `clap` - CLI parsing with derive macros
- `globset` - glob matching
- `rayon` - parallel execution
- `shell-words` - command parsing
- `tracing` / `tracing-subscriber` - logging
- `ctrlc` - signal handling
- `thiserror` - typed domain errors
- `anyhow` - top-level error handling

## Important Configuration Files

| File | Purpose |
| --- | --- |
| `rust-toolchain.toml` | Pin Rust version |
| `rustfmt.toml` | Rust formatting rules |
| `.cargo/config.toml` | Build optimizations and linker config |
| `deny.toml` | Dependency license/advisory policy |
| `lefthook.yaml` | Git hook configuration |
| `dist-workspace.toml` | `cargo-dist` release configuration |

## Release

Releases are managed by `cargo-release` for versioning and `cargo-dist` for distribution.
See [RELEASE.md](./RELEASE.md) for release workflow details.

Install `cargo-release` to automate releases:

```bash
cargo install cargo-release
```

Run a release:

```bash
# Update CHANGELOG.md first, then:
cargo release 0.2.0
# or cargo release minor
```

Artifacts are published to GitHub Releases, Homebrew (`howmanysmall/pullhook`), npm (`@pobammer/pullhook`),
and shell/powershell installers.

Supported targets include `aarch64` and `x86_64` for macOS, Linux, and Windows.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/howmanysmall)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/howmanysmall)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
