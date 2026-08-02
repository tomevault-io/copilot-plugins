## codeman

> Canonical agent/contributor guidance for this repository lives in [CLAUDE.md](CLAUDE.md) —

# Repository Guidelines

Canonical agent/contributor guidance for this repository lives in [CLAUDE.md](CLAUDE.md) —
project structure, build/test/lint commands, code style, testing safety rules
(never run the full suite inside a managed tmux session), security notes, and
the deployment workflow are all maintained there. Please read it before making
changes, and keep it the single source of truth rather than duplicating
sections here.

Quick pointers:

- Type check: `tsc --noEmit` · Lint: `npm run lint` · Format: `npm run format:check`
- Targeted tests only: `npm test -- test/<file>.test.ts` (bare `npm test` is unsafe in managed sessions)
- Route tests use `app.inject()`; new tests needing ports must pick a unique `const PORT =`
- Branch off `master` for all work; Conventional Commit-style messages (`fix(mobile): ...`)
- Never commit secrets or local state from `~/.codeman/`

---
> Source: [Ark0N/Codeman](https://github.com/Ark0N/Codeman) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
