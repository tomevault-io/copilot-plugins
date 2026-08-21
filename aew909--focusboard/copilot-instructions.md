## focusboard

> Before making code, database, authentication, routing, or deployment changes:

# Repository Instructions

## Current Plan

Before making code, database, authentication, routing, or deployment changes:

1. Read `ROADMAP.md`.
2. Confirm which roadmap slice is currently `IN PROGRESS` or `NEXT`.
3. Keep work scoped to that slice unless the user explicitly changes the plan.

## Roadmap Maintenance

Whenever a roadmap slice is committed and pushed:

1. Update `ROADMAP.md` in the same commit when practical.
2. Mark the completed slice `COMPLETE`.
3. Record its commit hash and push date.
4. Record any migrations, environment changes, deployment actions, or verification still pending.
5. Mark the next active slice `NEXT` or `IN PROGRESS`.
6. Add material architecture decisions or deviations to the Decision Log.

Do not claim a slice is complete while required migration, verification, or deployment work remains.

## Multi-Client Invariants

- Preserve Liona's existing board, slugs, settings, and event history during migration.
- Treat the client as the tenant and authorization boundary.
- Enforce access server-side for every read, write, upload, and content-generation request.
- Do not rely on hidden UI or semi-secret URLs for authorization.
- Avoid reintroducing hardcoded single-board behavior.

---
> Source: [AEW909/FocusBoard](https://github.com/AEW909/FocusBoard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
