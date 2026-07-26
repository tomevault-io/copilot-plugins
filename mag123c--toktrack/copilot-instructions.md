## toktrack

> Guidance for any AI coding agent (Codex, Claude, Cursor, …) working in this repo.

# AGENTS.md

Guidance for any AI coding agent (Codex, Claude, Cursor, …) working in this repo.
This file is intentionally tool-agnostic. The canonical project context lives in:

- [`CLAUDE.md`](./CLAUDE.md) — quick start, workflow, commands, commit rules
- [`.claude/ai-context/architecture.md`](./.claude/ai-context/architecture.md) — layers, paths, traits, data flow
- [`.claude/ai-context/conventions.md`](./.claude/ai-context/conventions.md) — naming, TDD, error handling, commits

Read those first. The notes below are the few things most easily missed.

## Non-obvious rules

- **Degrade gracefully on partial failure.** `DataLoaderService` aggregates many
  independent sources; one source failing must never abort the whole load. Warn
  and continue. Optional/remote sources should fall back to their last good
  snapshot. A source the user selected explicitly may fail loudly, but one pulled
  in implicitly (a configured default) must not break a plain command. See the
  "Data Loading" section in `conventions.md`.
- **Errors use `ToktrackError`**, not `anyhow`, in library code.
- **Comments and test names are English only.** (Private `.dev/` docs are exempt.)
- **TDD**: no implementation without a test first (RED → GREEN → REFACTOR).

## Before you finish

```bash
make check    # fmt + clippy + test — must pass before committing
```

## Commits

```
{type}({scope}): {description}
```
types: `feat|fix|refactor|docs|test|chore|perf` · scopes: `parser|tui|services|cache|cli`

Do not add AI/agent attribution lines (no `Co-Authored-By` for the agent) to commits.

---
> Source: [mag123c/toktrack](https://github.com/mag123c/toktrack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
