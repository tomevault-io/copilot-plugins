## hanabi

> - Hanabi is an online multiplayer Hanabi game with a React 19 SPA and a Node/Express server using Socket.IO v4 for real-time gameplay.

# Agent Guide

## Repo snapshot

- Hanabi is an online multiplayer Hanabi game with a React 19 SPA and a Node/Express server using Socket.IO v4 for real-time gameplay.
- pnpm + Nx monorepo with Vite for web (`apps/web/`) and tsx/tsc for server (`apps/server/`).
- Shared types and utilities in `packages/shared/`.
- Runtime target: Node v24.11.1 with mise/direnv.

## Tooling expectations

- Use mise with `.tool-versions`; load via direnv when present (`mise install`, then `direnv allow`).
- Run commands from the active checkout root (`git rev-parse --show-toplevel`), including worktrees.
- Use `pnpm` for all package operations.
- Use `pnpm dev` to run both web and server in development mode.
- Use `pnpm typecheck` and `pnpm lint` to verify code quality.

## Coding guardrails

- Preserve runtime behavior; avoid changing gameplay logic unless explicitly requested.
- Be cautious with websocket changes; keep long-lived connection semantics intact.
- Keep secrets in `.env` and never commit them.
- Prefer small, focused changes and avoid unnecessary abstractions.

## Committing

- If asked to commit and push, use the current branch and push to origin.
- If asked to "ship it", commit and push the current branch, then merge it to `main`; Railway deploys `main` automatically.

## Work checkpoints (run after a logical chunk)

- If available, run lint/typecheck/format from repo root and report results.
- Update README/specs/AGENTS if behavior or interfaces change.

## Working from specs

- When executing a spec, after finishing a step, mark it in the spec with a green check mark emoji (✅) and add any implementation notes beside the step.

---
> Source: [btraut/hanabi](https://github.com/btraut/hanabi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
