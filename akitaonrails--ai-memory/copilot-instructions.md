## ai-memory

> > Read this every session before touching code. The long-form research and

# CLAUDE.md — ai-memory project directives

> Read this every session before touching code. The long-form research and
> design specs live under [`docs/`](docs/); this file is the operating rules.

## What this project is

A self-contained Rust binary providing long-term memory for AI coding agents
(Claude Code, OpenAI Codex, OpenCode) over the Model Context Protocol.
Storage = markdown-in-git wiki (source of truth) + SQLite (derived index).
Capture = automatic via agent lifecycle hooks, never manual `write_note`.
Consolidation = Karpathy "LLM Wiki" pattern with versioned supersession.

Read [`docs/design-decisions.md`](docs/design-decisions.md) for the full spec.
Read [`docs/research-karpathy-llm-wiki.md`](docs/research-karpathy-llm-wiki.md)
for what "Karpathy-faithful" means in practice.

## Stack (do not deviate without updating `docs/design-decisions.md` §4)

- **Runtime:** Rust 1.95 (pinned in `rust-toolchain.toml`), edition 2024,
  resolver 3, async via `tokio`.
- **MCP:** `rmcp` (official `modelcontextprotocol/rust-sdk`).
- **Store:** `rusqlite` + `refinery` migrations, FTS5 in v1, `sqlite-vec` in
  v0.2. **One file**, one writer actor, one read pool.
- **Wiki:** markdown on disk, `notify-debouncer-full` watcher with heartbeat
  + reconciliation, `git2` for versioning.
- **HTTP:** `axum` for hook ingress + MCP HTTP/SSE.
- **LLM:** typed clients per provider (Anthropic, OpenAI, OpenAI-compat) via
  `reqwest`. **Never** a generic gateway like LiteLLM (cognee #2840 lesson).
- **Config:** `figment`, one read at startup, passed by `&Arc<Config>`.
- **Logging:** `tracing` with module filters; never let the appender's own
  module log at INFO+ (agentmemory #519 lesson).

## Repository layout

```
crates/
  ai-memory-core/        # domain types, errors. NO IO.
  ai-memory-store/       # SQLite, single-writer actor, migrations.
  ai-memory-wiki/        # markdown read/write, watcher, git.
  ai-memory-mcp/         # rmcp transport + tool router.
  ai-memory-hooks/       # hook payload schemas + HTTP ingress.
  ai-memory-llm/         # LlmProvider trait + 3 impls.
  ai-memory-consolidate/ # Karpathy ingest/query/lint pipeline.
  ai-memory-cli/         # `ai-memory` binary entry point.
hooks/                   # vendored hook scripts per agent.
docker/                  # Dockerfile + compose.
docs/                    # research + design (DO NOT delete).
tests/                   # workspace integration tests.
```

## Workflow rules

1. **Milestone by milestone.** Do not start M(n+1) until every "Done when"
   bullet in M(n) passes. See [`docs/design-decisions.md`](docs/design-decisions.md)
   for the milestone list. No mixing.
2. **No dead code, no half-built features.** If a feature is not finished,
   it does not land. If you must stub something, document it as `M(n) TODO`
   in the relevant module's doc-comment with the milestone number.
3. **Tests before claiming done.** Every milestone requires:
   - `cargo fmt --all -- --check` (no diffs)
   - `cargo clippy --workspace --all-targets -- -D warnings` (no warnings)
   - `cargo test --workspace` (all green)
   - Manual exercise of the new feature against a real agent CLI when applicable.
4. **Document the why in code, not the what.** No comments restating the line
   above; only comments explaining a constraint, an incident, or a non-obvious
   invariant.
5. **Add a unit test before the implementation, not after.** Especially for
   parsers, ID derivation, and any retention/decay math.
6. **Don't refactor outside the milestone.** Touch only what the current
   milestone requires; resist scope creep.

## Cross-cutting invariants (carved in, never violated)

These come straight from issue-tracker research on agentmemory, basic-memory
and cognee — every one of them is in `docs/design-decisions.md` §14 with
issue citations. **Treat any code review that violates one of these as a
blocking issue:**

1. One config-read path. No `std::env::var` outside `Config::load()`.
2. Single-writer SQLite actor. All writes through one `mpsc` channel.
3. Indexes commit in the same transaction as the data. No
   background-task-indexing-after-return.
4. Typed `(WorkspaceId, ProjectId, PagePath)` identity in every layer.
5. Hooks are fire-and-forget. Sub-second timeouts. Return 202 immediately.
6. Privacy strip is a typed boundary (`RawHookPayload → Sanitized<Observation>`).
7. JSON-schema structured outputs only. No XML, no `instructor`-style wrapping.
8. `{provider, model, dim}` stored next to every embedding. Refuse on mismatch.
9. Live-process check (`sysinfo`) before any destructive op.
10. Atomic file writes (tmp + rename + fsync). Watcher ignores own writes.
11. Default data dir is an absolute canonical platform path.
    Logged loudly on startup.
12. No global singletons / `lazy_static` configs.
13. Zero-LLM default path. LLM features opt-in via env.
14. Tracing subscribers explicitly filter their own module.

## Mistakes documented in the research — do NOT repeat

- [`docs/issues-agentmemory.md`](docs/issues-agentmemory.md): install/ops
  landmines (iii-engine sidecar, distroless volumes, cwd-relative paths).
- [`docs/issues-basic-memory.md`](docs/issues-basic-memory.md): file watcher
  pain, manual-capture friction, multi-workspace retrofit.
- [`docs/issues-cognee.md`](docs/issues-cognee.md): LiteLLM/instructor wire
  drift, multi-store sync bugs, dependency landmines.

When in doubt about a design decision, search those files for the keyword.

## Quick commands

```bash
# Build everything.
cargo build --workspace

# Lint + format + test (run before every commit).
cargo fmt --all -- --check
cargo clippy --workspace --all-targets -- -D warnings
cargo test --workspace

# Auto-format.
cargo fmt --all

# Exercise the binary.
./target/debug/ai-memory --version
./target/debug/ai-memory init
AI_MEMORY_DATA_DIR=/tmp/x ./target/debug/ai-memory init
./target/debug/ai-memory status --json

# CI parity (requires cargo-deny + cargo-audit installed).
cargo install cargo-deny cargo-audit
cargo deny check
cargo audit
```

## What this project is NOT (v1 non-goals)

See [`docs/design-decisions.md`](docs/design-decisions.md) §13 for the full
list. Highlights: no multi-tenant auth, no web UI, no Postgres backend in v1,
no alternative vector backends, no remote sync (use `git remote` on the wiki
dir), no multimodal.

## Plan & status

The current execution plan is at
[`/home/akitaonrails/.claude/plans/cuddly-moseying-karp.md`](/home/akitaonrails/.claude/plans/cuddly-moseying-karp.md)
(local to the maintainer's `~/.claude/`).
Live progress is tracked via the TaskList tool inside Claude Code sessions.

---
> Source: [akitaonrails/ai-memory](https://github.com/akitaonrails/ai-memory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
