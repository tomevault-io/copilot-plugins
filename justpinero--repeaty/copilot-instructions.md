## repeaty

> - **Repeaty** — Repeat after Peaty.

# CLAUDE.md — Repeaty

## Identity
- **Repeaty** — Repeat after Peaty.
- One PWA that unifies SRS flashcards, comprehension speed scoring, and Whisper pronunciation feedback, with AI-generated decks/feedback for Pro.

## Stack
React 18 + Vite + TS · Tailwind · Dexie · Workbox PWA · Supabase (Postgres + Auth + Storage + Edge Functions) · TS end-to-end · Vitest + Playwright.

## Key commands
- Install: `pnpm i`
- Dev (web): `pnpm --filter @repeaty/web dev`
- Validate (same checks CI runs): `bash scripts/validate.sh`
- Supabase local: `supabase start` · migrations: `supabase db push`

## References (load on demand)
- See @references/architecture.md for stack decisions and ADRs.
- See @references/schema.md for the data model and RLS policies.
- See @references/api-contracts.md for Edge Function signatures.
- See @references/env-vars.md for required env vars (no values).
- See @references/deployment-landmines.md for Vercel + Supabase + LLM gotchas.
- See @references/security-landmines.md for input validation and RLS patterns.
- See @references/repeaty-pwa.md for PWA + Capacitor abstraction notes.

## Action loop (compressed)
**PRIME** — read CLAUDE.md, check `.claude/handoff.md` (incorporate + delete), read target request and any referenced files, run tests to verify working state, state understanding + files to touch + app status. Zero code until primed.
**PLAN** — break into atomic steps; identify tests (one per acceptance criterion); flag deps + new packages (justify before installing); update relevant `references/*.md`; check references first to avoid reinventing; if request crosses concerns, propose splitting.
**RED** — write a failing test for every acceptance criterion. Tests MUST fail. Commit `test(X.Y): red — acceptance criteria tests`. Pure presentational components (no state/logic/data) may use the tests-after escape — state the reason in the commit; tests must still exist before "done".
**GREEN** — minimum code per test until all pass. Follow coding-standards. No commented-out code, console.logs, or orphan TODOs. Commit `feat(X.Y): green — all acceptance criteria met`.
**VALIDATE** — `bash scripts/validate.sh` (must mirror CI exactly). Lint (incl. a11y), types, unit, applicable E2E, smoke. On failure: write `requests/phase-N-fixes/fix-...md` with root cause + acceptance criteria, fix in-scope or flag for review, re-validate. On pass: commit `[phase-X.Y] description`, update `audits/debt.md` if shortcuts taken; if last in phase → `/phase-complete`.

## Coding standards (top 5 — full set in coding-standards skill)
1. Explicit over implicit; one concern per function; all async errors handled.
2. Validate all external data with Zod before DB or business logic.
3. A11y is not optional — native HTML or full role/tabIndex/keyboard/aria; lint blocks CI.
4. No magic strings/numbers; new deps must be justified in `architecture.md`.
5. Soft deletes (`deletedAt`); UTC timestamps; no console.logs or commented-out code in commits.

## Testing protocol
TDD is the strong default — RED before GREEN, one test per acceptance criterion. Tests-after escape only for purely presentational components with zero logic. Tests are never optional before a request is marked done.

## Git workflow
- `main` is always stable and deployable. Each phase = `phase-N-[name]` branch. Each completed request = one commit on that branch. Commit only after Validate passes.
- Phase branch merges to `main` only after `/phase-complete` (all four audits pass with no Criticals/blocking drift).
- Commit format: `[phase-X.Y] description`.

## Compaction
Before compacting, preserve: current phase, current request, files modified this session, test pass/fail status, and any blocking audit findings. After compaction, the `PostCompact` hook re-injects `CLAUDE.md`, `.claude/handoff.md`, and `audits/debt.md`.

## Definition of done
- All acceptance criteria have passing tests.
- `bash scripts/validate.sh` is green (lint incl. a11y, types, unit, applicable E2E).
- References updated if architecture/schema/API changed.
- Any shortcuts logged in `audits/debt.md`.
- Commit lands as `[phase-X.Y] description`.

---
> Source: [JustPinero/Repeaty](https://github.com/JustPinero/Repeaty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
