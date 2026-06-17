## clawbuddy

> - All of `bun fmt`, `bun lint`, and `bun typecheck` must pass before considering tasks completed.

# AGENTS.md

## Task Completion Requirements

- All of `bun fmt`, `bun lint`, and `bun typecheck` must pass before considering tasks completed.
- NEVER run `bun test`. Always use `bun run test` (runs Vitest).
- The Docker Compose-based Makefile targets are optional for future CI/CD or containerized workflows; local task completion should still use the direct Bun commands above unless a task explicitly asks otherwise.

## Project Snapshot

ClawBuddy is a Self-hosted, privacy-first AI agent platform with sandboxed tool execution. Inspired by Openclaw in many ways, but trying to improve the formula by adding more capabilities

This repository is a VERY EARLY WIP. Proposing sweeping changes that improve long-term maintainability is encouraged.

## Core Priorities

1. Performance first.
2. Reliability first.
3. Keep behavior predictable under load and during failures (session restarts, reconnects, partial streams).

If a tradeoff is required, choose correctness and robustness over short-term convenience.

## Maintainability

Long term maintainability is a core priority. If you add new functionality, first check if there is shared logic that can be extracted to a separate module. Duplicate logic across multiple files is a code smell and should be avoided. Don't be afraid to change existing code. Don't take shortcuts by just adding local logic to solve a problem.

## Package Roles

- `apps/api`: Hono
- `apps/web`: React/Vite UI. Owns session UX, conversation/event rendering, and client-side state. Connects to the server

## Coding Principles — Non-negotiable

These are hard rules, not suggestions. Violations will be rejected.

### 1. Keep it simple or don't do it

Every piece of code must justify its existence. If the simplest solution works, use it. No abstractions "for later", no clever tricks, no premature generalization. If there are two ways to do something and one is simpler, that's the one you pick — no exceptions.

### 2. Delete dead code without hesitation

Dead code, unused imports, orphaned functions, commented-out blocks — delete them immediately. The repo is not a museum. Git has history; the code doesn't need to remember what it used to be. If something isn't called, it doesn't exist. No `// TODO: maybe use this later`, no `_unusedVar` renames. Delete it.

## Reference Repos

- OpenCode : https://github.com/anomalyco/opencode
- OpenClaw : https://github.com/openclaw/openclaw

Use these as implementation references when designing protocol handling, UX flows, and operational safeguards.

---
> Source: [DanielD2G/ClawBuddy](https://github.com/DanielD2G/ClawBuddy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
