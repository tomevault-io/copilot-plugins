## remote-codex

> This repository contains an IM-first Codex remote-control daemon.

# AGENTS.md

This repository contains an IM-first Codex remote-control daemon.

## Project intent

- Runtime backend is `codex app-server` only.
- v1 channel is Discord.
- Deployment model is local/single-host daemon (no external control plane).
- CLI is the operator surface.

## Repository layout

- `packages/core-runtime`: app-server process lifecycle + JSON-RPC transport.
- `packages/im-gateway`: IM adapters and command parsing.
- `packages/state-store`: local config/state/audit persistence.
- `packages/ops-cli`: daemon orchestration and CLI commands.
- `ref/`: local reference projects (`Claude-to-IM-skill`) for lookup only.

## Development workflow

1. Use Node.js 24 LTS+.
2. Run tests with `npm test` after changes.
3. Test-only validation is sufficient for this repo; lint is optional and not required by CI.
4. Keep interfaces stable between modules:
- `RuntimeEngine`
- `ApprovalBroker`
- `ChannelAdapter`
- `SessionBinding` / `PolicyProfile`
5. Prefer incremental changes with tests alongside behavior changes.
6. Keep secrets local and redacted in logs.

## Commit rules

- Make atomic, small commits.
- One logical change per commit.
- Commit messages should be short and clear.
- Use conventional prefixes:
  - `feat: ...`
  - `fix: ...`
  - `chore: ...`
  - `test: ...`
  - `docs: ...`

Examples:

- `feat: add app-server runtime reconnect handler`
- `fix: handle stale daemon pid cleanup`
- `test: add approval timeout roundtrip case`

## Guardrails

- Do not add external relay/mobile flows to v1 core.
- Keep allowlist and approval gating on by default for command/file-change actions.


<claude-mem-context>
# Memory Context

# [auto] recent context, 2026-04-22 9:31pm GMT+8

No previous sessions found.
</claude-mem-context>

---
> Source: [zychenzy/remote-codex](https://github.com/zychenzy/remote-codex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
