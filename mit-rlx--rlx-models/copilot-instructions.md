## rust-fmt-clippy

> Enforce cargo fmt and clippy (warnings as errors) on all Rust work


# Rust fmt + clippy

This repo enforces `cargo fmt` and `cargo clippy -- -D warnings` automatically:

- Cursor `afterFileEdit` / `afterTabFileEdit` runs `rustfmt` on edited `.rs` files
- Cursor `stop` re-runs the scoped gate and follow-ups on failure
- Git pre-commit (after `just install-hooks`) gates staged `.rs` files
- CI workflow `.github/workflows/fmt-clippy.yml` gates the full workspace
- Publish preflight uses the same bar (`scripts/publish.sh`)

## Agent requirements

After any Rust edit in a turn:

1. Prefer letting hooks format; if you touch files via shell, run `just fmt` or `rustfmt` on those paths
2. Before finishing substantive Rust work, run `just lint` (workspace) or `./scripts/rust-lint-gate.sh --packages <crate>` for a scoped check
3. Treat clippy warnings as errors — fix them; do not blanket `#[allow]` unless matching existing crate exceptions in `[workspace.lints.clippy]`
4. Do not disable or bypass Cursor hooks / the pre-commit gate

---
> Source: [MIT-RLX/rlx-models](https://github.com/MIT-RLX/rlx-models) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
