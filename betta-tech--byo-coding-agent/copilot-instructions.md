## byo-coding-agent

> This file is loaded into the system prompt at startup by the harness itself

# Project context for AI agents

This file is loaded into the system prompt at startup by the harness itself
(see `loadAgentsContext` in `main.go`) and is also picked up by external
agents like Claude Code that follow the AGENTS.md convention.

## What this repo is

A small, opinionated coding-agent harness in Go (~1,000 lines), written as a
learning project. The README and `follow_along/` chapters explain why each
piece exists.

## Layout

- `main.go` — wiring: provider, tools, MCP servers, AGENTS.md, root agent, TUI.
- `commands.go` — slash command registry. Touches every extension point.
- `delegate.go` — the delegate tool. Lives in main to avoid an import cycle.
- `internal/api/` — shared message types. No internal deps.
- `internal/provider/` — `Provider` interface + Anthropic adapter.
- `internal/tool/` — `Tool` interface, `Registry`, one file per local tool.
- `internal/mcp/` — wrappers that expose remote MCP server tools as local Tools.
- `internal/compact/` — compaction strategies (no-op, sliding window, summarize, logging decorator).
- `internal/agent/` — the agent loop. One `Agent` per conversation.
- `internal/subagent/` — `Subagent` interface + research subagent + active tracker.
- `internal/ui/` — banner, spinner, the Bubble Tea program that owns the UI.

## Conventions

- **Errors as tool results, not Go errors.** When a tool fails, return
  `(message, true)` so the model can read and recover. Don't crash the loop.
- **One file per tool / strategy / subagent.** Tools self-register via
  `init()`; subagents register explicitly in `main` because they need a
  `Provider` at construction time.
- **The provider abstraction is real.** Anthropic SDK types are only allowed
  in `internal/provider/anthropic.go`. If you find them anywhere else, that's
  a leak.
- **Comments explain *why*, not *what*.** Names carry the *what*.

## Build / run

```sh
export ANTHROPIC_API_KEY=sk-ant-...
go run .         # launches the TUI
go build ./...   # sanity check
go vet ./...     # before committing
```

## Optional config files (gitignored)

- `mcp.json` — MCP server definitions (see `mcp.example.json`).
- `AGENTS.md` — this file. Lives at the repo root and is committed because
  it's documentation; in a real project you'd decide per-repo whether to
  track it.

## What this project is NOT

- A production agent. No streaming, no auth, no sandboxing, no persistence.
- A best-practices Go reference. The `internal/`-heavy layout was chosen for
  pedagogy, not idiom (see `follow_along/en/10-project-structure.md`).

---
> Source: [betta-tech/byo-coding-agent](https://github.com/betta-tech/byo-coding-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-21 -->
