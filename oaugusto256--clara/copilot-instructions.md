## clara

> This file defines **repo-specific rules** for Claude Code when working in this codebase.

# Claude Code — Project Instructions (Clara)

This file defines **repo-specific rules** for Claude Code when working in this codebase.

## Canonical references (must follow)

- `docs/TASKS.md`: map any requested change to the most relevant task(s) before coding.
- `docs/DOMAIN_SCHEMAS.md`: canonical domain types + invariants (single source of truth).
- `docs/LOCAL_DEV.md`: how to run the project locally (front/back/db).

## Core engineering principles

- Prefer **clarity over cleverness**: explicit, readable code; incremental complexity.
- Keep strict architectural boundaries:
  - **Frontend (`apps/web`)**: UI state + rendering only. **No financial calculations** outside the rules engine.
  - **Backend (`apps/api`)**: thin routes + validated inputs + predictable errors. **Do not implement business rules** that belong in the rules engine.
  - **Rules engine (`packages/rules-engine`)**: all financial logic must be deterministic + explainable.
- If scope / acceptance criteria is missing, ask **targeted clarifying questions** before broad changes.
- For any non-trivial change, include a short **verification plan** + highlight affected edge cases.

## Domain invariants (non-negotiable)

Source: `docs/DOMAIN_SCHEMAS.md` (treat as truth).

- **Money is integer minor units** (no floats) and always includes explicit `currency`.
- **No floating-point arithmetic** for financial calculations.
- **Transactions are immutable**; do not silently overwrite historical data.
- `source` is informational only (should not change identity/business meaning).
- **No financial calculation without currency**.

## Frontend rules (`apps/web`)

- Organize by feature under `apps/web/src/features/`.
- Server state belongs in the query/data-fetching layer; components should derive UI state.
- Prefer derived state over stored state for server data.
- Use schema-driven forms/validation; do not invent validation rules inside components.
- Performance mindset for large datasets:
  - virtualize tables
  - avoid unnecessary re-renders
  - memoization only when intentional
- Any new/changed UI must explicitly handle:
  - `loading`
  - `empty`
  - `error`
  - `success`

## Backend rules (`apps/api`)

- Keep routes thin: **route → service → repository (or direct data access) → schema validation**.
- Validate all input at the boundary using shared schemas (never trust frontend payloads).
- Prefer explicit, predictable read-only APIs for dashboard data; avoid magic behavior / implicit side effects.
- Financial logic belongs in `packages/rules-engine`; backend orchestrates/persists, not calculates.
- Error handling must be deterministic/predictable:
  - return clear error messages
  - log actionable details

## Local dev quick commands

See `docs/LOCAL_DEV.md` for details. Quick run:

```bash
pnpm install
pnpm -w -F @clara/api run dev:all
pnpm -w -F @clara/web dev
```

## When editing code

- Prefer small, reviewable diffs.
- Don’t add narrative comments. Only comment when explaining non-obvious intent/constraints.
- If you introduce lints/tests failures, fix them (or clearly explain why they must be deferred).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oaugusto256)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/oaugusto256)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
