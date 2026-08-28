## dsh-agui-demo

> - Use Bun for dependency installation, scripts, tests, and local development.

# HIS Development Guide

## Project Shape

- Use Bun for dependency installation, scripts, tests, and local development.
- `app/` is the React 19 + Vite client. Use TanStack Router for routes, TanStack Query for remote state, and Zustand only for local UI state.
- `server/` is the Hono API. Use Drizzle ORM with Bun SQLite and Zod at every HTTP boundary.
- `ui/` is the visual reference source. Do not import its prototype HTML or CSS into production code; translate its design language into typed React components and semantic theme tokens.
- Keep shared API contracts in `packages/contracts/`. Contracts must not depend on browser or server-only modules.

## Commands

- Install all dependencies from the repository root with `bun install`.
- Run the full stack with `bun run dev`.
- Run quality checks with `bun run check` and tests with `bun run test`.
- Generate a Drizzle migration after schema changes with `bun run db:generate`.

## Frontend Rules

- Add shadcn components through `bunx --bun shadcn@latest`; prefer existing components over custom controls.
- Use semantic CSS tokens from `app/src/styles.css`. Do not scatter raw brand or status colors through components.
- Use Lucide icons and icon-only buttons for familiar tools. Every unfamiliar icon-only action needs a tooltip and accessible label.
- Route components coordinate data and page composition. Reusable shell, clinical, and feedback UI belongs under `app/src/components/`.
- All API reads and mutations go through Query hooks in `app/src/features/` or `app/src/lib/`. Do not fetch in effects.
- Keep URL-worthy state such as patient ID in TanStack Router. Keep ephemeral UI state such as theme and mobile panel mode in Zustand.
- Preserve the reference UI's clinical density: cool white/gray surfaces, 1px dividers, compact typography, no persistent card shadows, and radii at or below 8px.
- Desktop and mobile layouts must both be usable. Tables may scroll horizontally; clinical text and controls must never overlap.

## Backend Rules

- Mount business endpoints under `/api`; keep `/health` for readiness checks.
- Parse params, query strings, and request bodies with Zod before using them.
- Return a consistent `{ data }` success envelope and `{ error: { code, message } }` failure envelope.
- Keep route handlers thin. Database queries and workflow rules belong in focused repository/service modules when they grow beyond a few statements.
- Use transactions for multi-table writes. Never trust client-provided clinical status transitions without validating the current database state.
- Seed data must be deterministic and idempotent so local setup works on a fresh database.
- Model the outpatient workflow around Encounter state. Clinical Documents, Clinical Requests, Diagnostic Reports, and Prescriptions have independent lifecycles.
- Keep editable drafts separate from formal actions. Saving a draft must never imply ordering, issuing, signing, or completing an Encounter.
- The server owns completion prerequisites and validates them again for every signing or completion command.

## Verification

- A change is complete only after relevant TypeScript checks, tests, and production builds pass.
- API changes require at least one happy-path and one validation/error-path test.
- Workflow changes must verify persistence by reading the updated resource back from the API.
- UI changes must be inspected at desktop and mobile widths, including loading, empty, error, and disabled states where applicable.

## Safety

- This repository is a demonstration HIS, not a production medical device. Keep the disclaimer in user-facing documentation.
- Never log secrets or full patient payloads. Use fictional seed data only.
- Do not add Agent or AI-assisted clinical behavior until that scope is explicitly resumed. The current product is a clinician-operated HIS workflow.

---
> Source: [CaiZongyuan/DSH-AGUI-demo](https://github.com/CaiZongyuan/DSH-AGUI-demo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
