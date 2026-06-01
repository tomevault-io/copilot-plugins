## weblink

> This repo is a SolidJS + TypeScript (strict) WebRTC chat / file-transfer app.

# weblink — Agent Working Agreement

This repo is a SolidJS + TypeScript (strict) WebRTC chat / file-transfer app.

## Quick commands

- Dev: `bun dev`
- Tests: `bun test`
- Build: `bun run build`
- Preview: `bun preview`

## Code style / conventions

- TypeScript `strict: true` (see `tsconfig.json`).
- Formatting is handled by Prettier (see `.prettierrc.js`):
  - `printWidth: 60`, `tabWidth: 2`
  - Tailwind class sorting plugins are enabled
- Prefer explicit types at module boundaries (public APIs, protocols).
- Prefer `AbortController` for listener lifetimes; avoid leaked intervals/listeners.

## Architecture notes

- `src/libs/core`: low-level primitives (WebRTC session, RTC messaging, file transfer).
- `src/libs/services`: app-level orchestration/singletons (session/cache/transfer managers).
- UI should talk to services/state via stable interfaces; keep WebRTC details inside `core`.

## Refactor guidelines (stability-first)

- Make changes incrementally and keep behavior compatible unless explicitly approved.
- Avoid “god modules”: split by responsibility (sender/receiver/protocol/state).
- Add tests for new protocol/state-machine logic (Vitest) and keep existing flows working.
- Prefer dependency injection (pass services in) over adding new global singletons.
- If a protocol/message shape changes, version it and keep backward compatibility where possible.

## Docs

- The refactor roadmap lives at `docs/refactor-plan.md`. Keep it updated as work progresses.

---
> Source: [99percentpeople/weblink](https://github.com/99percentpeople/weblink) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
