## autodev

> cargo install --path . --root ~/.local

# AGENTS.md

## Build

```bash
cargo check
cargo build
cargo install --path . --root ~/.local
```

## Validate

```bash
cargo test
cargo clippy --all-targets --all-features -- -D warnings
```

## Essentials

- CLI binary: `auto`
- Required tools on PATH: `claude`, `codex`, `pi`, `gh`
- Main source tree: `src/`
- Dated specs: `specs/`
- Generated/runtime state: `.auto/`, `bug/`, `nemesis/`
- Checkpoint exclusions are `.auto/`, `bug/`, `nemesis/`, and `gen-*` via `CHECKPOINT_EXCLUDE_RULES` in `src/util.rs`
- `auto parallel` host-owned queue state includes `IMPLEMENTATION_PLAN.md`,
  `REVIEW.md`, legacy queue files, and `RECEIPTS-DRIFT.md`; lane workers should
  preserve evidence in commits/logs instead of editing those shared files
  directly.
- Verification receipt JSON under `.auto/symphony/verification-receipts/` is
  staging/compatibility data. Durable task proof should travel in
  `Auto-Verification-Receipt-*` commit-message footers produced by the host.

## Loom Access

- Loom operator SSH should use Tailscale: `ssh -i ~/.ssh/id_ed25519_hetzner root@100.124.18.111`.
- Public SSH on the Loom IPv4 may be firewalled even while the HTTPS edge is healthy.

---
> Source: [happybigmtn/autodev](https://github.com/happybigmtn/autodev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
