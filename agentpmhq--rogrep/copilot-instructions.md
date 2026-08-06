## rogrep

> - `main` is protected. Never commit to it directly: branch → PR

# Instructions for coding agents working on rogrep

## Ground rules

- `main` is protected. Never commit to it directly: branch → PR
  (`gh pr create`) → squash merge. CI (build with `-D warnings` + full test
  suite) must pass.
- rogrep is **local-only**. Do not add network I/O anywhere except behind
  the disabled-by-default seam in `rogrep-model/src/remote.rs`.
- All derived data (SQLite store, tantivy index, checkpoints) rebuilds from
  source rollout files on version bumps — never write a migration. When you
  change parse output, index content, or SQLite schema, bump the matching
  version constant (see the table in CONTRIBUTING.md); forgetting it ships
  stale data to users.

## Build / test

```sh
cargo build --workspace --all-targets   # must be warning-free (CI uses -D warnings)
cargo test --workspace
cargo insta review                      # after intentional parser-output changes
```

Real-corpus smoke test (uses your own session history, safe/read-only on
sources): `cargo build --release -p rogrep && ./target/release/rogrep sync
&& ./target/release/rogrep doctor`.

## Architecture map

Workspace crates, in dependency order:

- `rogrep-model` — normalized types (Turn/Conversation/Exchange), ids,
  project-key normalization, config/paths, remote seam.
- `rogrep-parsers` — discovery, the parse driver, one module per provider
  (`providers/`), SQLite→JSONL spool exporters (`spool/`), fixtures/tests.
- `rogrep-tooltree` — shell segmentation, git/gh facet extraction, per-turn
  facet tokens, git-ops timelines.
- `rogrep-store` — SQLite schema + stats SQL.
- `rogrep-index` — tantivy schema, writer (tail refresh), query grammar.
- `rogrep-engine` — the sync pipeline (flock, checkpoint diff, generation
  handshake).
- `rogrep` — CLI (`src/cmd/`), TUI (`src/tui/`), bundled skill
  (`assets/SKILL.md`).

## Invariants you must not break

1. **Incremental exactness**: `parse(full) == parse(prefix) + resume(tail)`
   — enforced by `crates/rogrep-parsers/tests/incremental.rs` at every line
   boundary, including checkpoint-state equality. Checkpoints freeze at the
   start of the last exchange; late records may only amend turns inside the
   open exchange (`ctx.amendable()`); never mutate frozen turns — aborts
   point forward instead.
2. **Search-index consistency**: the index commits before store checkpoints
   in a sync batch; tail refresh (delete-from-watermark + re-add) must stay
   idempotent so crash-redo is safe.
3. **Default search scope**: corpus search excludes invisible turns
   (injected harness context) and `origin:auxiliary` conversations (codex
   automoderation judges); conversation-scoped `find` greps everything.
   Keep new features consistent with that split.
4. The facet keys documented in `assets/SKILL.md` must exist in
   `rogrep-index::query::KNOWN_FACET_KEYS` (a test enforces it — update
   both together).

## Adding a provider

Follow `docs/providers/TEMPLATE.md`: one module in
`rogrep-parsers/src/providers/`, a registry entry (order matters for
overlapping paths), an `AgentKind` variant, fixtures, snapshot +
incremental-split tests.

---
> Source: [agentpmhq/rogrep](https://github.com/agentpmhq/rogrep) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-05 -->
