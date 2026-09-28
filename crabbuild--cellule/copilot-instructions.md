## cellule

> Read the nearest crate `AGENTS.md` before changing that crate. This workspace is a reusable Rust framework; product authentication, HTTP, cloud credentials, and deployment wiring belong to the embedding application.

# Cellule contributor guide

Read the nearest crate `AGENTS.md` before changing that crate. This workspace is a reusable Rust framework; product authentication, HTTP, cloud credentials, and deployment wiring belong to the embedding application.

## Layers

`cellule-types → cellule-store → cellule-ltx → cellule-runtime → cellule-app → cellule-host`. Higher layers may use lower layers; `cellule-host` also uses `cellule-runtime` directly. The optional `cellule-peer-http` adapter depends only on `cellule-runtime`; applications still own HTTP endpoints and authorization. Keep storage transport separate from authority and application policy. `scripts/check-boundaries.py` checks workspace dependencies and the pure coordination kernel.

## Contracts

- A Cell has one fenced writer. A successful command response follows durable publication or a recoverable follower-log proof.
- Recovery verifies the authority-pinned root and every required chunk; reconstruction is byte-identical or fails.
- Persisted IDs, descriptors, object paths, LTX formats, and signed peer messages are compatibility contracts. Read producers and consumers before changing them.
- Blob part deletion requires a complete cross-Cell reference set, quiesced writes, and a grace boundary.
- Shutdown drains accepted work and releases leases, slots, tasks, and SQLite handles on every exit path.

## Work

Search callers, callees, sibling implementations, tests, and documentation before changing an API. Keep one canonical path; avoid speculative configuration and compatibility shims. Preserve source errors. Do not use `unwrap`, `expect`, or `panic!` outside tests. Keep comments near non-obvious ownership and ordering invariants. Update documentation and runnable examples with behavior changes.

Verification routes:

| Check | Command |
| --- | --- |
| Format | `cargo fmt --all --check` |
| Features and targets | `cargo check --workspace --all-targets --all-features --locked` |
| Tests | `cargo test --workspace --all-features --locked` |
| Local LTX | `cargo test -p cellule-ltx --no-default-features --locked` |
| Lints | `cargo clippy --workspace --all-targets --all-features --locked -- -D warnings` |
| API docs | `RUSTDOCFLAGS='-D warnings' cargo doc --workspace --all-features --no-deps --locked` |
| Boundaries/layout | `python3 scripts/check-boundaries.py` and `python3 scripts/check-module-layout.py` |
| Document syntax and links | `python3 scripts/check-doc-rust-fences.py` and `python3 scripts/check-doc-links.py` |
| SQL/peer contracts | `node crates/cellule-runtime/docs/validate.mjs` |

Never weaken qualification profiles or expected evidence to silence a failure.
Use CI or an isolated verification snapshot for broad suites and process tests.
Cloud and fault tests require their documented environment. On workstations with the mounted Workspace volume,
set `CARGO_TARGET_DIR` beneath the mounted `$HOME/Workspace/crabbuild-target`,
with one directory per checkout.

Keep the main documentation scannable: short explanations, contract tables,
diagrams, and valid Rust examples. The packaged application guide has a doctest;
all Rust fences pass the syntax gate. Cellule is the authoritative implementation for Cell framework changes.
Keep module entries, focused production submodules, and their unit tests in the
same directory; use integration suites for public behavior.

---
> Source: [crabbuild/cellule](https://github.com/crabbuild/cellule) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-28 -->
