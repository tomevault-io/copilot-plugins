## deja

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**deja** is a Claude Code plugin that provides zero-config persistent memory across sessions. It captures file reads, edits, bash commands, and decisions via Claude Code hooks, compresses them locally, and injects context into future sessions. Everything lives in a single SQLite file (`~/.deja/memory.db`) with no external services required.

## Tech Stack

- **Runtime:** Bun (>= 1.3.6) — required for native `bun:sqlite` bindings and fast startup
- **Language:** TypeScript (strict mode)
- **Database:** SQLite via `bun:sqlite` with WAL mode and FTS5 for full-text search
- **Tests:** `bun:test`
- **IPC:** Unix domain socket (macOS/Linux), TCP localhost:19532 (Windows)
- **MCP:** `@modelcontextprotocol/sdk` (only npm dependency at Tier 0)

## Build & Test Commands

```bash
bun test                    # Run all tests
bun test <file>             # Run a single test file
bun run build               # Build to dist/
bun run src/worker/main.ts  # Run the worker directly
```

## Architecture

### Micro-kernel + Pipelines

A thin kernel owns the SQLite database and Unix socket. Everything else is a pipeline stage — small, pure functions that the kernel orchestrates:

```
Hook event → Queue → [Debounce] → [Classify] → [Normalize] → [Extract] → [Store + Index]
```

### Key Process Boundaries

Three separate processes access `memory.db`:

| Process | Access | Purpose |
|---|---|---|
| **Worker** (long-running) | Read-write | Observation processing, indexing, session summaries |
| **SessionStart hook** | Read-write | Context injection (reads DB directly), stats counters |
| **MCP server** (stdio) | Read-only | In-session search via `deja_search`/`deja_timeline`/`deja_observe` tools |

The worker and MCP server never communicate directly. Hooks send events to the worker via Unix socket; the MCP server reads the DB independently. All concurrent access is safe under WAL journal mode.

### Source Layout

```
src/
├── kernel/           # db.ts, socket.ts, wal.ts, timers.ts, lock.ts, migrations/
├── pipelines/
│   ├── ingest/       # classify.ts, normalize.ts
│   ├── extract/      # heuristic.ts, ast.ts (Tier 1), llm.ts (Tier 3)
│   ├── index/        # fts.ts, vector.ts
│   └── search/       # fts.ts, rerank.ts, hybrid.ts
├── hooks/            # session-start.js, post-tool-use.js, prompt-submit.js, session-stop.js
├── mcp/              # server.ts, tools.ts
├── context/          # generator.ts (SessionStart context injection)
├── dashboard/        # index.html (single file, vanilla JS, embedded CSS)
├── cli/              # install, uninstall, learn, search, stats, etc.
└── worker/           # main.ts (entry point)
```

### Dependency Tiers

The codebase uses progressive enhancement tiers. Tier 0 is the full product, not a degraded mode:

| Tier | What it adds | Key files |
|---|---|---|
| **Tier 0** | Heuristic extraction + FTS5 search | `extract/heuristic.ts`, `index/fts.ts` |
| **Tier 1** | tree-sitter AST parsing | `extract/ast.ts` |
| **Tier 2** | sqlite-vec + ONNX local embeddings | `index/vector.ts`, `search/rerank.ts` |
| **Tier 3** | LLM extraction (opt-in API calls) | `extract/llm.ts` |

### Pipeline Stage Contract

Each pipeline stage after debounce is a pure function: input → output, zero side effects, independently testable with no mocking. Target: ~50-100 lines per stage.

### Event Classification

The classifier (`classify.ts`) is the cost-saver — 30-40% of events are tagged `skip` and never stored. Significance levels: `skip | low | medium | high | critical`. First match wins (rules evaluated top-to-bottom). The classifier is a pure function; the pipeline orchestrator handles counter increments.

### Content Normalization

The normalizer (`normalize.ts`) caps `content_summary` at 2000 characters. Truncation cuts at the last complete line. For Write tool events, content is in `input.content` (not `output`). For Read tool events, content is in `output.content`.

### Context Injection (SessionStart)

SessionStart reads the DB **directly** (not through the worker) to stay under 200ms. Budget: 8000 chars default, split 40/50/10 across last session summary, high-significance observations, and cross-project insights. Token estimation: 4 chars ≈ 1 token.

### WAL Failover

Any socket error triggers WAL fallback — the hook appends to `~/.deja/pending.wal` and returns. The worker drains the WAL on startup. WAL uses a lockfile protocol (`flock`) because payloads can exceed `PIPE_BUF`.

## Data Model

Key tables: `observations` (main data), `sessions` (lifecycle), `observations_fts` (FTS5 virtual table with triggers for INSERT/UPDATE/DELETE sync), `stats` (counters), `schema_version`.

- `significance` field on observations: `skip | low | medium | high | critical`
- `kind` field: `file_read | file_edit | file_write | bash_cmd | decision | prompt`
- `raw_event` column stores original hook JSON for future reprocessing
- `embedding` column is nullable BLOB (only populated when Tier 2 is active)
- Timestamps are epoch-only integers (`created_at_epoch`), human-readable derived via SQL
- Foreign keys intentionally NOT enforced (tolerates out-of-order WAL inserts)
- SessionStart/Stop are session lifecycle events, NOT stored as observations

## MCP Tools (3-Layer Protocol)

1. `deja_search` → lightweight index (id, title, significance, kind, date)
2. `deja_timeline` → chronological context around an anchor (same session only)
3. `deja_observe` → full observation details, HARD CAP: max 10 IDs per request

Never call `deja_observe` without filtering through `deja_search` first.

## Platform Considerations

- macOS: Apple's system SQLite disables extension loading — Tier 2 setup downloads a compatible dylib
- Windows: Uses TCP localhost instead of Unix sockets; `lock.ts` exports a platform-agnostic interface using `LockFileEx` instead of `flock`
- Cross-platform locking: `src/kernel/lock.ts` provides `acquireLock(path)` / `releaseLock(fd)`

## Key Design Decisions

- **Project identity is the working directory path** — same repo at two paths = two projects (intentional)
- **No auto-pruning by default** — storage is ~250MB/year at heavy use. Opt-in via `--retention`
- **Worker starts lazily** on first hook fire, shuts down after 30min idle
- **Multiple Claude Code sessions share one worker** — events tagged by `session_id`, processed sequentially
- **API keys never stored in settings.json** — read from env vars at runtime (`ANTHROPIC_API_KEY`, `GEMINI_API_KEY`, `DEJA_LLM_API_KEY`)
- **Migrations are forward-only** — no rollback scripts; new forward migration to reverse
- **FTS writes always go through `INSERT INTO observations`** — never directly into the FTS table; triggers handle sync
- **Dashboard is a separate process** from the worker — reads DB directly, independent lifecycle

---
> Source: [anandmt/deja](https://github.com/anandmt/deja) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
