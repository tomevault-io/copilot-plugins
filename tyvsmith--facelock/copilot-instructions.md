## facelock

> Facelock is a Linux face authentication PAM module written in Rust. Single unified binary (`facelock`) handles CLI, daemon, one-shot auth, and benchmarks. The PAM module (`pam_facelock.so`) is a thin client that either connects to the daemon or spawns `facelock auth`. IPC uses the D-Bus system bus (`org.facelock.Daemon`), not Unix sockets.

# AGENTS.md — Facelock

## Project Overview

Facelock is a Linux face authentication PAM module written in Rust. Single unified binary (`facelock`) handles CLI, daemon, one-shot auth, and benchmarks. The PAM module (`pam_facelock.so`) is a thin client that either connects to the daemon or spawns `facelock auth`. IPC uses the D-Bus system bus (`org.facelock.Daemon`), not Unix sockets.

## Crates

Cargo workspace, 11 crates. What each is for — the manifests carry no `description`, so this is the only map.

- `facelock-core` — config, types, errors, D-Bus interface, traits
- `facelock-camera` — V4L2 capture, auto-detection, preprocessing
- `facelock-face` — ONNX inference (SCRFD + ArcFace)
- `facelock-store` — SQLite face embedding storage
- `facelock-daemon` — auth/enroll logic, rate limiting, liveness, audit, D-Bus server
- `facelock-cli` — the `facelock` binary, plus a library target so the domain layer is testable
- `facelock-bench` — standalone benchmark and calibration utility
- `pam-facelock` — the PAM module, a thin client to the daemon
- `facelock-tpm` — TPM-sealed key encryption, software AES-256-GCM
- `facelock-polkit` — polkit authentication agent
- `facelock-test-support` — mocks and fixtures

Dependencies are whatever the manifests say; there is no separate allowed-list to keep in sync. The one enforced boundary is the PAM ceiling below.

## Build & Verify

```bash
cargo build --workspace
cargo test --workspace
cargo clippy --workspace -- -D warnings
cargo run --bin facelock -- --help
```

## GPU Support (Optional)

GPU acceleration is a runtime option — no special build flags needed. Install a
GPU-enabled ONNX Runtime package and set `execution_provider` in config:

```bash
sudo pacman -S onnxruntime-opt-cuda   # NVIDIA
# Then edit /etc/facelock/config.toml: execution_provider = "cuda"
```

Supported providers: `cpu` (default), `cuda` (NVIDIA), `rocm` (AMD), `openvino` (Intel).

## Core Rules

- Do not change binary names, paths, config keys, database schema, or auth semantics without updating `docs/contracts.md`.
- Keep the PAM module free of heavy dependencies (no ort, no v4l, no facelock-core). `pam_facelock.so` loads into every PAM-using process, `sshd` and `login` included. Full ceiling in `.claude/rules/pam-boundary.md`.
- Keep all inference local. No cloud services, no runtime model downloads in the auth path.
- Prefer minimal dependencies and clear crate boundaries.

## Security Rules

See `.claude/rules/security.md`, which loads when you open daemon, PAM, TPM, or auth-path sources.

## Code Style

- `thiserror` for library error types, `anyhow` in binaries.
- Return `Result<T>` over panicking. Never `unwrap()` in library code.
- `tracing` for structured logging. Control verbosity via `RUST_LOG` env filter.
- `#[cfg(test)]` modules in each source file for tests.

## Testing Strategy

See `.claude/rules/testing.md`, which loads when you open the justfile, `test/`, crate integration tests, or CI workflows.

## Workspace Conventions

- Version declared once in root `Cargo.toml`, inherited via `version.workspace = true`.
- Inter-crate deps use relative paths (`path = "../facelock-core"`).
- Release profile: LTO + single codegen unit + strip.

## Releasing

- `just release X.Y.Z` bumps version in Cargo.toml, PKGBUILD, spec, debian/changelog.
- Tag `vX.Y.Z` triggers CI to build binaries, .deb, and .rpm.
- Update `CHANGELOG.md` before committing the release.
- See `docs/releasing.md` for full process and versioning contract.

## Commits, PRs, and Issues

Applies to anything an agent authors in this repository, including runs
triggered from GitHub (`@claude` in a comment or issue).

- Titles: `<type>(<scope>): <subject>`. Types: `feat` `fix` `refactor` `polish` `docs` `test` `ci` `chore` `perf` `build` `style` `revert`.
- No AI attribution anywhere: no "Generated with Claude Code" footer, no `Co-Authored-By` trailer, no mention of Claude, AI, agents, or assistants in a title, body, commit message, or changelog entry.
- Open PRs as drafts. The maintainer marks them ready.
- PR and issue bodies are bullets: verb-first fragments, lowercase, no terminal period. Prose only for the `Why`, capped at four sentences. A small PR is two or three bullets with no headings.
- Never paste command output, CI logs, or pass/fail tables into a PR. Naming a check under `Validation` is fine; reporting its result is not.
- Every PR says why the change exists. If it cannot be derived, ask; never invent one.
- Docs are part of the deliverable. A change to paths, config keys, CLI flags, D-Bus interface, or behavior updates `docs/` (and `docs/contracts.md` where it applies) in the same PR.

---
> Source: [tyvsmith/facelock](https://github.com/tyvsmith/facelock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
