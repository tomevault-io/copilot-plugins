## namakh-in

> - All of `pnpm fmt`, `pnpm lint`, and `pnpm typecheck` must pass before considering tasks completed.

# AGENTS.md

## Task Completion Requirements

- All of `pnpm fmt`, `pnpm lint`, and `pnpm typecheck` must pass before considering tasks completed.
<!-- - NEVER run `pnpm test`. Always use `pnpm run test` (runs Vitest). -->

## Project Snapshot

namakh is a e-commerce website & store for all of Namakh.in's products.

This repository is a VERY EARLY WIP. Proposing sweeping changes that improve long-term maintainability is encouraged.

## Core Priorities

1. Performance first.
2. Reliability first.
3. Keep behavior predictable under load and during failures (session restarts, reconnects, partial streams).

If a tradeoff is required, choose correctness and robustness over short-term convenience.

## Maintainability

Long term maintainability is a core priority. If you add new functionality, first check if there is shared logic that can be extracted to a separate module. Duplicate logic across multiple files is a code smell and should be avoided. Don't be afraid to change existing code. Don't take shortcuts by just adding local logic to solve a problem.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sah-Nikhil)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Sah-Nikhil)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
