## claude-mem-lite

> Lightweight persistent memory system for Claude Code. MCP server + hooks plugin.

# claude-mem-lite

Lightweight persistent memory system for Claude Code. MCP server + hooks plugin.

## Quick Reference

- **Version**: 2.55.0
- **Package manager**: npm
- **Test**: `npx vitest run` (61 test files, vitest)
- **Lint**: `npx eslint .`
- **Benchmark**: `node benchmark/benchmark.mjs`
- **DB**: better-sqlite3 + FTS5 full-text search
- **Node**: >=18, ESM (`"type": "module"`)

## Architecture

| Module | Role |
|--------|------|
| `cli.mjs` | CLI entry point — routes subcommands to mem-cli.mjs or install.mjs |
| `mem-cli.mjs` | CLI commands: search, recent, recall, get, timeline, save, delete, update, export, compress, maintain, fts-check, stats, context, browse, registry |
| `hook.mjs` | Main hook entry — handles session-start/stop/post-tool-use/user-prompt |
| `hook-context.mjs` | CLAUDE.md context injection, adaptive time windows, token budgeting |
| `hook-llm.mjs` | Haiku-based summarization and title generation |
| `hook-memory.mjs` | Semantic memory injection on user prompt |
| `hook-episode.mjs` | Episode batching for observations |
| `hook-handoff.mjs` | Cross-session handoff state (/clear, /exit continuity) |
| `hook-shared.mjs` | Shared constants/utilities (RUNTIME_DIR, session mgmt) |
| `hook-semaphore.mjs` | Concurrency control for hook execution |
| `hook-update.mjs` | Auto-update via GitHub Releases (24h check, dev-mode skip) |
| `hook-optimize.mjs` | LLM-powered optimization: re-enrich, normalize, cluster-merge, smart-compress |
| `server.mjs` | MCP server — 17 tools total: 6 core exposed via `tools/list` (mem_search/mem_recent/mem_recall/mem_get/mem_save/mem_timeline) + 11 hidden-but-callable (mem_delete/mem_update/mem_export/mem_compress/mem_maintain/mem_optimize/mem_fts_check/mem_stats/mem_registry/mem_use/mem_browse). Hidden tools stay routable by exact-name `tools/call`; Claude Code agents reach them via the `claude-mem-lite <cmd>` CLI. Split flag lives in `tool-schemas.mjs`. |
| `registry.mjs` | Resource registry DB schema + CRUD |
| `registry-retriever.mjs` | FTS5 search + BM25 composite scoring + domain filtering |
| `registry-indexer.mjs` | Resource indexing pipeline |
| `tfidf.mjs` | TF-IDF vector engine — tokenization, vocabulary, vectors, cosine similarity, RRF merge |
| `tier.mjs` | Temporal tier system — activity-based time window classification |
| `schema.mjs` | DB schema definitions and migrations (incl. vocab_state, observation_vectors) |
| `utils.mjs` | FTS query sanitization, synonym expansion, CJK extraction, token estimation |
| `scripts/post-tool-use.sh` | Bash fast pre-filter (~5ms, skips low-value tools) |
| `scripts/user-prompt-search.js` | UserPromptSubmit hook — auto-search memory on user prompts |

## Key Patterns

- CLI commands: `claude-mem-lite search|recent|recall|get|timeline|save|delete|update|export|compress|maintain|optimize|fts-check|stats|context|browse|registry`
- Tool name mapping: Claude Code Agent tool = `'Agent'` (not `'Task'`); Skill via `event.tool_input?.skill`
- Tests use `:memory:` DB — schema changes must sync to test files
- FTS5 search: sanitizeFtsQuery (synonym expansion) → BM25 scoring → OR fallback → concept co-occurrence
- Context delivery: SessionStart hook stdout emits the `<claude-mem-context>` block fresh from DB; CLAUDE.md is no longer auto-updated (pre-v2.30 left a stale snapshot here)
- Skill commands (`/search`, `/recall`, `/recent`, `/timeline`) use `!` preprocessing for CLI injection

## Mem usage contract (applies to ALL sessions touching this repo)

This project *is* the memory plugin. Dogfood it. The rules below override soft "proactive trigger"
language in the MCP tool description — when the two conflict, this contract wins.

**Before you Edit/Write any code file**: the PreToolUse hook (`scripts/pre-tool-recall.js`) has
already run `mem_recall` for that file. If you saw lines like `#NN [bugfix] ...` in your
tool-result preamble, you **must** cite `#NN` in your response (even a one-liner
`"per #NN, I avoided re-introducing X"`) so the user can verify you actually consumed the
lesson. Failure to cite when a lesson was surfaced = contract violation.

**After solving a non-trivial bug** (≠ typo fix, ≠ rename): you **must** call
`mem_save(type='bugfix', lesson_learned='<one-line root cause + one-line fix>',
importance=2)`. Test: could a future session touching the same file have avoided this bug
if they'd seen the lesson? If yes → save it. If no → it wasn't a real bug fix.

**After making a non-obvious architectural decision** (≠ renaming, ≠ moving code): call
`mem_save(type='decision', lesson_learned='<constraint + why this choice + what it trades off>')`.
Empirical note: `decision` observations have 72.7% hit rate vs `change` at 16.5% — one good
decision memory is worth ~20 change memories. Do not inflate this — decision is reserved
for real tradeoffs, not style choices.

**Do not write `lesson_learned: 'none'` just to satisfy the schema.** Either write a lesson
that a future session could actually use, or leave the field NULL and accept a low-importance
observation. The Haiku prompt defaults to "none" far too aggressively; when you save manually,
you override that default.

**When searching memory via CLI/MCP**: default search now excludes low-signal fallback titles
(`Modified X`, `Worked on X`, raw error logs). If you're auditing or specifically hunting a
file-change record, pass `--include-noise` (CLI) or `include_noise=true` (MCP).

---
> Source: [sdsrss/claude-mem-lite](https://github.com/sdsrss/claude-mem-lite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
