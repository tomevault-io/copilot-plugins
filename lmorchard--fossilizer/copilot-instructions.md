## fossilizer

> Static-site generator that ingests Mastodon exports (and incremental API fetches) into a browsable personal archive. Rust 2021, CLI via Clap. See `README.md` and `docs/` for user-facing docs.

# fossilizer — project notes for Claude

Static-site generator that ingests Mastodon exports (and incremental API fetches) into a browsable personal archive. Rust 2021, CLI via Clap. See `README.md` and `docs/` for user-facing docs.

## Commands (Makefile-first)

- `make check` — `cargo fmt --all -- --check` + `cargo clippy --all-targets --locked -- -D warnings` (this is the CI lint gate)
- `make test` — `cargo test --locked`
- `make build` — `cargo build --release --locked`
- `make serve` — run the local static-site server
- `make docs` — build the mdBook docs (`./scripts/build-docs.sh`)

Always run `make check` and `make test` after changes; CI enforces both.

## Conventions

- **Errors: `anyhow::Result` everywhere.** Don't reintroduce `Result<_, Box<dyn Error>>`. Produce errors with `anyhow!`/`bail!` and add context with `.context(...)`. Gotcha: `RwLock`/`Mutex` `PoisonError` is `!Send`, so it can't go through anyhow's `?` — map it: `.map_err(|e| anyhow!("... {e}"))?` (see `src/config.rs`).
- **Lint: `#![warn(clippy::pedantic)]`** is enabled crate-wide in `src/lib.rs` and `src/main.rs`, with a curated `#![allow(...)]` block (documented inline). CI runs clippy with `-D warnings`, so keep it clean; add to the allow block (with a reason) only for genuine CLI-vs-library noise.
- **Shared fs/asset helpers** live in `src/util.rs` (`open_outfile_with_parent_dir`, `copy_embedded_assets`). Reuse them rather than re-implementing.
- **TLS: rustls** (`reqwest` uses `rustls-tls`, i.e. bundled webpki/Mozilla roots — chosen for portability). No OpenSSL/native-tls; don't add them back.
- Prefer `&str`/`&Path`/`&[T]` over `&String`/`&PathBuf`/`&Vec<T>` in new signatures (a broad sweep of existing ones is deferred — see below).

## CI gotcha (important)

`.github/workflows/ci.yml`: the **full cross-build matrix runs only on push to `main`/tags, not on PRs** (`if: github.event_name != 'pull_request'`). PRs get only `lint` + a single `x86_64-musl` quick-test. **A green PR is not full validation** — macOS/Windows/aarch64 are checked post-merge. For platform-sensitive changes (deps/TLS, filesystem/symlinks, subprocess spawning), watch the matrix run on `main` after merging (`gh run list --branch main`). The build/test jobs are gated `needs: lint`.

## Integrating changes

- **Always integrate through a Pull Request.** Never merge a feature branch into `main` locally, and never commit directly to `main` — every change (code, docs, config) lands via a PR that Les reviews and merges.
- When finishing a branch, the default action is "push and open a PR," not a local merge.

## Dev sessions

Session artifacts live in `docs/dev-sessions/<timestamp>-<slug>/` (`spec.md`, `plan.md`, `research.md`, `notes.md`); the `notes.md` files record decisions and deferrals worth reading before related work.

## Backlog

Tracked in GitHub issues, not here — check open issues for known bugs, security items, and planned work.

---
> Source: [lmorchard/fossilizer](https://github.com/lmorchard/fossilizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-09 -->
