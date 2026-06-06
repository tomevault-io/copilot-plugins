## obscura

> Rust port of [javascript-obfuscator](https://github.com/javascript-obfuscator/javascript-obfuscator). 100% feature parity target; 10–100× speed target.

# Project: obscura

Rust port of [javascript-obfuscator](https://github.com/javascript-obfuscator/javascript-obfuscator). 100% feature parity target; 10–100× speed target.

## If you're a new agent: read `.agents/PRIMER.md` FIRST.

It's the 5-minute onboarding doc — locked decisions, glossary, where things live, how to run things. Don't grep the codebase blind. Prime your context, then work.

## Invariants (enforced by clippy + CI)

- **No `unwrap`/`expect`/`panic`** in library code (`crates/obscura-core`, `obscura-napi`, `obscura-wasm`). Allowed only in `main.rs` and `#[cfg(test)]` blocks. Errors propagate via `obscura_core::Result<T, ObfuscationError>`.
- **Determinism.** Given `(source, options)`, output MUST be byte-identical across runs. RNG is `ChaCha20Rng` seeded from `options.seed` (or input hash when seed is 0). NEVER use `thread_rng` / `Math.random`-style nondeterminism in core.
- **Parity bar.** Every transform's output diffs against upstream goldens in `fixtures/golden/<transform>/`. PRs that regress parity are blocked.
- **No `unsafe` in core.** `#![forbid(unsafe_code)]` is on. napi/wasm crates may use `unsafe` strictly inside binding-generator macros.
- **MSRV = 1.96** (see `rust-toolchain.toml`). Don't reach for newer language features.

## Tool priorities (this user's environment)

- `yeet` commands over built-in tools where applicable (see global CLAUDE.md). If `yeet` isn't installed in this project, use the built-ins.
- `code-review-graph` MCP tools FIRST for codebase exploration (faster than grep, gives structural context). Fall back to Grep only when graph doesn't cover the question.
- For multi-transform work in Phase 2: use `Agent` calls with `isolation: 'worktree'` so parallel agents don't conflict.

## Workflow

```bash
# One-time
git config core.hooksPath .githooks
cd tools/parity-harness && npm install && cd ../..

# Build
cargo build --workspace

# Test
cargo test --workspace
cargo test -p obscura-core            # core only, fast
cargo test --workspace --test parity -- --include-ignored   # full parity suite

# Lint (CI runs these)
cargo fmt --all --check
cargo clippy --workspace --all-targets -- -D warnings

# Bench
cargo bench -p obscura-core
```

## Adding a transform: TL;DR

Full recipe: `.agents/skills/add-transform.md`. Summary:

1. Create `crates/obscura-core/src/transforms/<name>.rs` implementing `TransformStage`.
2. Register it in `transforms/mod.rs` `registry()` at the correct upstream-order position.
3. Add option fields to `options.rs` (camelCase serde rename).
4. Add fixtures under `fixtures/corpus/<name>/` and run `node tools/parity-harness/generate.js <name>` to produce goldens.
5. Add tests under `crates/obscura-core/tests/transform_<name>.rs`.
6. `cargo test -p obscura-core --test transform_<name>` until green, then full parity.

## Pointers

- `docs/ARCHITECTURE.md` — pipeline ordering, scope tracking, transform contract.
- `docs/TRANSFORMS.md` — per-transform spec with upstream source links.
- `docs/TASKS.md` — open work, claimable by parallel agents.
- `.agents/PRIMER.md` — agent onboarding (READ FIRST).
- `.agents/skills/*.md` — concrete recipes (add-transform, verify-parity, bench).
- `.agents/agents/*.md` — specialized subagent definitions.
- `/Users/crash0v3rrid3/.claude/plans/come-up-with-a-jazzy-salamander.md` — full plan.

---
> Source: [Crash0v3rrid3/obscura](https://github.com/Crash0v3rrid3/obscura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-06 -->
