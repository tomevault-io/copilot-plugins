## ai-cv-maker

> For every functional change — new feature, modification, or refactor — use the `add-feature` skill. Do not write code first and spec later.

# Claude Instructions — Applysy

## Workflow: Spec First, Always

For every functional change — new feature, modification, or refactor — use the `add-feature` skill. Do not write code first and spec later.

The order is:
1. User story / goal
2. Spec (what it does, edge cases, acceptance criteria)
3. Tests
4. Implementation
5. Verification — prove each acceptance criterion works in a real browser against the local stack (the skill's `browser-verifier` subagent does this)
6. Documentation update
7. Commit on the feature branch and open a PR automatically (merging stays with the user)

Never skip the spec phase, even for small changes.

The skill does all work in a dedicated git worktree on a feature branch by default — `main` in the primary checkout stays clean until the branch is reviewed and merged.

**Exception — infrastructure / security / cross-service work:** document the
decision as an ADR in `doc/adr/` (template in `doc/adr/README.md`) instead of
a detailed user story. Optionally pair it with an outcome-level user story for
verification; acceptance criteria must stay observable-outcome level (no
resource names, no tech choices — those belong in the ADR and spec).

## Documentation Must Stay in Sync

After every implementation, verify that relevant docs are updated:
- `ARCHITECTURE.md` — if the system topology, data model, API routes, or service responsibilities changed
- `doc/` — any feature-level or API docs in scope
- `README.md` — if setup, commands, or high-level description changed
- In-code comments — only where the *why* is non-obvious

If docs are out of date, update them as part of the same change, not a follow-up.

## UI Changes: Angular Only

`apps/ui-angular` is the sole production SPA. All UI changes go here. `apps/ui-react` has been removed from the project.

## Project Structure

Nx monorepo. Key apps:

| App | Stack | Role |
|---|---|---|
| `apps/ui-angular` | Angular 21 | Primary SPA (main UI) |
| `apps/chat-ui` | React Web Component | AI chat widget embedded in Angular |
| `apps/profile-builder-ui` | React Web Component | Canonical document-style profile editor (`<applysy-profile-builder>`), embedded in Angular at `/job-profiles/:id` (ADR-0013) |
| `apps/cv-api` | ASP.NET Core 9 | REST API, PDF generation, gRPC client |
| `apps/llm-service` | Python / gRPC (grpc.aio) | LLM gateway |
| `apps/admin-api` | NestJS | Admin backend |
| `apps/admin-ui` | Next.js | Admin frontend |

Shared library: `libs/auth` (`@applysy/auth`) — auth utilities shared by Angular and React.

See `ARCHITECTURE.md` for full system design, data model, and API reference.

## Common Commands

```sh
# Serve Angular UI
npx nx run ui-angular:serve

# Serve chat widget (dev, port 4202)
npm run serve:chat

# Run all tests
npx nx run-many --target=test

# Run specific app tests
npx nx run ui-angular:test
npx nx run cv-api:test

# Lint
npx nx run-many --target=lint

# Docker local stack
docker-compose up
```

## Key Conventions

- Angular app uses standalone components (no NgModules)
- Auth token stored in `localStorage` via `@applysy/auth` constants — do not hardcode key names
- LLM provider is controlled by env vars (`LLM_PROVIDER`, `LLM_MODEL`, etc.) — no code changes needed to switch
- EF Core migrations live in `apps/cv-api/Infrastructure/Persistence/Migrations/` — always generate a migration for schema changes
- gRPC contract is the source of truth in `proto/llm_service.proto`

---
> Source: [izmaylovdev/ai-cv-maker](https://github.com/izmaylovdev/ai-cv-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
