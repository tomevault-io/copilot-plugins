## nt-cli

> A thin CLI that wraps Notion's hosted MCP server, designed for AI agents to interact with Notion through bash.

# nt-cli

A thin CLI that wraps Notion's hosted MCP server, designed for AI agents to interact with Notion through bash.

```
nt CLI  --HTTPS/MCP Streamable HTTP-->  mcp.notion.com (Notion-hosted)
```

Each invocation is a stateless HTTPS request. No daemon, no local MCP process. OAuth tokens stored in `~/.config/nt/`.

## Design Documents

- `idea.md` - Vision, architecture, principles, prior art
- `cli-design.md` - Command structure, output format, error handling, composition examples

Read both before making architectural decisions.

## Working Principles

- Work autonomously when the path is clear, even on long-running tasks.
- Ask for clarification when requirements are ambiguous or when a foundational design choice is uncertain.
- Be explicit about tradeoffs and call out risks directly.
- If a workaround is complex because of tooling or access limits, ask first.
- Treat dangerous or irreversible actions as opt-in; confirm before proceeding.
- Proactively surface refactoring or simplification opportunities during implementation, with a short note on impact.
- If implementing a test reveals a possible bug in the code under test, flag it instead of writing a test that validates incorrect behavior.

## Context Recovery

- Read `context/current.md` at the start of a run.
- Update `context/current.md` at the end of a run.
- Record durable decisions in `context/decisions.md` (append-only).
- Use `context/sessions/YYYY-MM-DD.md` only for substantial work.
- Do not store secrets in `context/`.

## Go Style

- Idiomatic Go; follow the patterns in the existing codebase.
- Small, focused functions. Early returns to reduce nesting.
- Prefer the standard library. Add dependencies only when they earn their weight.
- Use `internal/` for packages that are not part of the public API.
- Errors are values; wrap with context (`fmt.Errorf("doing x: %w", err)`).
- No global mutable state. Pass dependencies explicitly.
- Prefer ASCII punctuation in prose and comments; avoid em dashes.

## CLI Conventions

- All output goes to stdout as JSON. Errors go to stderr as JSON.
- Exit codes: 0 success, 1 general error, 2 auth error, 3 not found, 4 rate limited, 5 permission denied.
- Content arguments accept `-` to read from stdin.
- Resource-scoped commands: `nt page <id> <verb>`, `nt db <id> <verb>`.
- Workspace-level commands: `nt search`, `nt create`, `nt login`, etc.

## Testing

- Test behavior, not implementation. Focus on public interfaces.
- Use table-driven tests for cases with multiple inputs.
- Use `testdata/` directories for fixtures and golden files.
- Run tests with `make test`.

## Commits and PRs

Use conventional commits: `type(scope): description`.

- Types: `feat`, `fix`, `refactor`, `test`, `chore`, `perf`, `docs`
- Scope: package name (e.g., `mcp`, `auth`, `cli`); omit for cross-package changes
- Keep each commit focused on one logical change

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Riki1312)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Riki1312)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
