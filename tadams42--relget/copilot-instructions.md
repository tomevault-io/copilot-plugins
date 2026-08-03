## relget

> Rust CLI that installs/updates CLI utilities directly from GitHub, GitLab and Codeberg releases.

# relget

Rust CLI that installs/updates CLI utilities directly from GitHub, GitLab and Codeberg releases.

## Build & run

```sh
cargo build
cargo run -- --help
cargo run -- install --apps ripgrep
cargo run -- install --apps ripgrep,bat        # comma-separated
cargo run -- registry list-apps-ids
cargo run -- completions zsh
cargo run -- uninstall --apps ripgrep
cargo run -- install --apps ripgrep --offline  # use only cached data
```

Use `--prefix tmp/try-relget/` to avoid needing `sudo` during local testing.

## Workspace layout

This is a virtual Cargo workspace with two crates:

- `relget/` — the main binary crate (`cargo run`, `cargo build`, `cargo test` target this by default)
- `xtask/` — build automation (`cargo xtask update-docs`)

All registry type definitions (`AppEntry`, `CategoryEntry`, etc.), `impl AppEntry` helpers, and the
semantic `validate()` function live in `relget/src/registry/types.rs`. That file is shared between
the runtime crate (as a normal module) and `relget/build.rs` (via `#[path]` inclusion), so the
build script and the binary agree on one set of types and one validation implementation — it must
stay self-contained (std + serde only).

App definitions live in `relget/src/registry/data/<letter>/<app-id>.jsonc`. `relget/build.rs` validates and compiles them into an embedded binary at build time; no registry changes require touching Rust code.

## Adding a new app

To add support for a new app, see the **[Contributing a new app](README.md#contributing-a-new-app)**
section in `README.md`. All app definitions live in `relget/src/registry/data/<letter>/<app-id>.jsonc` — no
Rust code is required.

## Token handling

Tokens are optional. Without them, `relget` works anonymously (subject to API rate limits).

`relget` by default tries to load tokens from config file. You may assume that this config file exists and is correctly set up in local development environment. You don't need to provide tokens in any way when calling `relget` for local testing.

## Cache

`RelgetCache` is reused for GitHub, Codeberg, and GitLab:
- GitHub: `~/.cache/relget/{owner}/{repo}/release.json` and `asset.{id}`
- Codeberg: `~/.cache/relget/codeberg/{owner}/{repo}/release.json` and `asset.{id}`
- GitLab: `~/.cache/relget/gitlab/{owner}/{repo}/release.json` and `asset.{id}`
- Release cache TTL: 1 day
- Asset cache: permanent (keyed by asset ID, which changes when a new release is published)

## Code conventions

- use `cargo +nightly fmt` to format the code
- use `cargo check --workspace` and `cargo clippy --no-deps` to lint the code
- `cargo clippy` skips re-linting files unchanged since the last build (incremental cache); to get a
  fresh lint of all local source without recompiling dependencies use
  `cargo clean -p relget -p registry-core -p xtask && cargo clippy --no-deps`
- git commit messages should use past tense (`added foobar` instead of `add foobar`,
  `adding foobar` or `adds foobar`)
- git commit messages should be prefixed by short category like `refact:`, `build:`,
  `ci:`, `feat:`, `docs:`, `chore:` and similar
- when adding new app into registry, commit message should contain Markdown syntax link
  to app's repository. That way, when `CHANGELOG.md` is generated and viewed in `GitHub`
  it contains nicely formatted links to added apps.

We use `cargo xtask update-docs` to keep `CHANGELOG.md` and `SUPPORTED_APPS.md` up to date. This means that after each `git commit` you should run `cargo xtask update-docs` and then fold changes into that latest commit. Note that our `xtask` doesn't put all commits into `CHANGELOG.md`: the ones with prefixes defined `NOISE_PREFIXES` in `xtask/main.rs` are skipped.

---
> Source: [tadams42/relget](https://github.com/tadams42/relget) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
