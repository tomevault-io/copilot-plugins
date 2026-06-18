## template-rust

> This is a GitHub template repository for bootstrapping production-grade Rust projects. Fork or

# template-rust — Claude Guide

This is a GitHub template repository for bootstrapping production-grade Rust projects. Fork or
"Use this template" on GitHub to get a fully wired project with CI, linting, security auditing,
coverage, and Docker support out of the box.

## Scaffolding Overview

| File / Dir | Purpose |
|---|---|
| `rust-toolchain.toml` | Pins stable Rust channel + required components |
| `rustfmt.toml` | Opinionated formatting (edition 2021, 100-col, crate-level imports) |
| `clippy.toml` | Clippy config — CI uses pedantic + nursery, suppresses `module_name_repetitions` |
| `deny.toml` | `cargo-deny`: license allow-list, bans duplicate deps, advisory checks |
| `Cargo.toml` | Release profile with LTO=thin, single codegen unit, symbol stripping |
| `Dockerfile` | Four-stage build: cargo-chef cache → build → distroless runtime (non-root) |
| `.pre-commit-config.yaml` | Local hooks: fmt check, clippy, deny |
| `.github/workflows/ci.yml` | Full CI matrix (see below) |
| `.github/workflows/release.yml` | release-please automated semver |
| `release-please-config.json` | release-please package config |
| `.release-please-manifest.json` | release-please version manifest |

## Build Commands

```bash
# Build
cargo build
cargo build --release

# Test (preferred — nextest for speed + better output)
cargo nextest run
cargo nextest run --all-features

# Format
cargo fmt
cargo fmt --check          # CI mode

# Lint
cargo clippy --all-targets --all-features -- \
  -D warnings \
  -W clippy::pedantic \
  -W clippy::nursery \
  -A clippy::module_name_repetitions

# Security
cargo deny check            # license + advisory + bans
cargo audit                 # RustSec advisories only

# Coverage
cargo llvm-cov --all-features --lcov --output-path lcov.info

# Docker
docker build -t template-rust .
docker run --rm template-rust
```

## CI Pipeline

The CI workflow (`.github/workflows/ci.yml`) runs:

1. **fmt** — `cargo fmt --check` on stable
2. **clippy** — matrix [stable, nightly]; nightly is `continue-on-error`
3. **test** — `cargo nextest run` on stable
4. **deny** — `cargo deny check` (licenses, advisories, bans)
5. **audit** — `cargo audit` via rustsec/audit-check
6. **coverage** — `cargo llvm-cov --lcov` → Codecov upload
7. **miri** — `cargo miri test` on nightly (`continue-on-error`)

## Release Flow

Releases are automated via [release-please](https://github.com/googleapis/release-please).
Merge a commit with a [Conventional Commit](https://www.conventionalcommits.org/) message to
`main` and release-please will open a versioned PR. Merging that PR creates a GitHub Release
and triggers the publish job.

## Architecture Notes

- **Single binary** crate layout — add library crates under `crates/` if the project grows into a workspace.
- **Distroless runtime** — the Docker image runs as `nonroot:nonroot` with no shell. Healthcheck stub assumes a `--health-check` flag; replace with an HTTP probe or remove.
- **cargo-chef** — dependency layer caching means rebuilds only recompile changed source, not all deps.
- **MSRV** is `1.80.0` — update `rust-version` in `Cargo.toml` and `msrv` in `clippy.toml` together.


## Related Templates

| Language | Repository |
|----------|------------|
| Python | [todie/template-python](https://github.com/todie/template-python) |
| Node.js | [todie/template-node](https://github.com/todie/template-node) |
| Terraform | [todie/template-terraform](https://github.com/todie/template-terraform) |

---
> Source: [todie/template-rust](https://github.com/todie/template-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
