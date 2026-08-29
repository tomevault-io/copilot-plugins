## scay

> This file orients any coding agent (or future session) resuming work on **scay**.

# AGENTS.md — working context for coding agents

This file orients any coding agent (or future session) resuming work on **scay**.
Read it first, then [`ROADMAP.md`](./ROADMAP.md) (current state + next step) and
[`docs/OVERVIEW.md`](./docs/OVERVIEW.md) (architecture + rationale).

## What scay is

A **small, debloated CLI coding agent** in Go. Four LLM backends (OpenAI,
Anthropic, Google Gemini, Ollama) behind one hand-rolled `Provider` interface —
**no vendor SDKs**. One module, one binary, nothing else to compile. See
OVERVIEW for the design.

## Prime directive: stay small

- **The binary carries exactly ONE sanctioned third-party dependency:** the official
  MCP SDK (`github.com/modelcontextprotocol/go-sdk`, plus its transitive deps), and
  the SDK types appear **only** in `internal/mcp`. Verify with:
  `go list -f '{{if not .Standard}}{{.ImportPath}}{{end}}' -deps .` → only
  `github.com/robitec97/scay/...` and the MCP SDK subtree
  (`modelcontextprotocol/go-sdk`, `google/jsonschema-go`, `segmentio/*`,
  `yosida95/uritemplate`, `golang.org/x/{oauth2,sys}`). (Don't grep for dots — the
  std library itself vendors `golang.org/x/...` internally.)
- Prefer the standard library. Reach for a dependency only when it demonstrably
  earns its weight — and never in the LLM/provider/agent/tools/config packages.

## Command surface (deliberately tiny)

`scay` opens the interactive session; **everything else lives inside it** as
slash commands (`/auth`, `/model`, `/effort`, `/mcp`, `/compact`, `/clear`, `/exit`).
The only subcommands are `scay yolo` (session with the approval gate bypassed)
and `scay update` (self-update from GitHub releases), plus `version`/`help`.
Don't add new top-level subcommands — extend the slash commands instead.

## Layout

```
main.go              # tiny entrypoint → internal/cli
internal/
  cli/       # dispatch (run/yolo/update) + the interactive session + slash commands
  agent/     # the loop + Session (flat message list) + system prompt
  provider/  # Provider interface + neutral types + per-provider adapters + effort ladder
  tools/     # read/list/grep/edit/write/bash + registry + approval gate
  config/    # load/save config.json (0600) + env overrides + Credential + mcpServers/.mcp.json
  mcp/       # MCP client over the official Go SDK (the ONLY pkg that may import it)
```

**Dependency direction points inward:** `internal/agent` must not import
`internal/cli` or `internal/mcp`.

## Build / run / test

```sh
go build ./...           # build everything (one module, no nested anything)
go vet ./...             # static checks
go test ./...            # tests
go build -o scay . && ./scay version
scripts/release.sh vX.Y.Z  # cross-compiled archives + checksums into dist/ (version via -ldflags)
```

Go lives at `/opt/homebrew/bin/go` on this machine (Homebrew, Apple Silicon); if
`go` isn't on PATH, prefix commands with `export PATH="/opt/homebrew/bin:$PATH"`.

## Conventions

- Idiomatic, readable Go. Comment the *why*, match surrounding density.
- Provider adapters hoist the divergent concepts (system prompt, effort, tool-call
  identity, reasoning blocks) — the agent loop stays provider-agnostic. See the
  provider-divergence table in OVERVIEW.
- Tools are safe by construction: read-before-edit, unique-match edits, printed
  diffs, and an approval gate on write/edit/bash (`scay yolo` bypasses it).
- **Update `ROADMAP.md` and `docs/OVERVIEW.md` at the end of each milestone** so the
  next session resumes cleanly.

---
> Source: [robitec97/scay](https://github.com/robitec97/scay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
