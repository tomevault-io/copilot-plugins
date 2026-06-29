## trackam

> Reusable Phase 0 workspace/account/admin scaffold.

# AGENTS.md

## Purpose
Reusable Phase 0 workspace/account/admin scaffold.

## Current mode
Document and harden the scaffold that exists today.
Do not write future-roadmap docs as if they are already implemented.

## Read first
- docs/GAP_ANALYSIS.md
- docs/ARCHITECTURE.md
- docs/DECISIONS.md
- docs/DOMAIN_MODEL.md
- docs/DB_SCHEMA.md
- docs/API_SPEC.md
- docs/UI_ARCHITECTURE.md
- docs/IMPLEMENTATION_PLAN.md when planning follow-up work
- docs/MIGRATION_WORKFLOW.md when schema, bootstrap, CI, or deploy behavior is touched
- frontend/AGENTS.md when working in frontend
- backend/AGENTS.md when working in backend

## Evidence order
During doc or audit work:
1. code
2. config
3. migrations and bootstrap scripts
4. existing docs

During implementation:
1. docs
2. code or config
3. validation

## Output style
Be concise.
Prefer bullets, short sections, and compact tables.

## Rules
- Document current reality, not idealized architecture.
- When sources disagree, record:
  - current state
  - gap
  - recommended target
- Treat script, workflow, and schema-source mismatches as first-class drift.
- Do not invent missing commands, routes, tables, or product surfaces.
- Keep diffs scoped to the touched runtime or harness area.
- When non-domain work uncovers a scaffold-level or template-level improvement, bring that change back to this template or the root scaffolder instead of leaving it only in one generated app.
- Update prompt-routing docs when the doc set or skill set changes.

## Current Phase 0 baseline
- Backend modules:
  - auth
  - users
  - roles
  - notifications
  - events
  - devices
  - sessions
  - settings
- Frontend surfaces:
  - public landing
  - auth
  - user dashboard
  - admin dashboard

## Validation
Before marking done:
- backend: `npm test`
- frontend: `npm run lint`
- frontend: `npx tsc -b`
- frontend: `npm test -- --run`
- frontend: `npm run build`
- report unavailable backend lint, typecheck, and build scripts explicitly

---
> Source: [Jeffreyon/trackam](https://github.com/Jeffreyon/trackam) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-28 -->
