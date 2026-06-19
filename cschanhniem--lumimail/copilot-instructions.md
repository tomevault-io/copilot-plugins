## lumimail

> You are working on **Lumimail**, a self-hosted multi-tenant email platform. Next.js 16 App Router on Cloudflare Workers via OpenNext, Drizzle ORM + D1, Tailwind v4, shadcn/Radix UI, TanStack Query, Zod. No backend/frontend split — everything in `src/`.

# AGENTS.md

You are working on **Lumimail**, a self-hosted multi-tenant email platform. Next.js 16 App Router on Cloudflare Workers via OpenNext, Drizzle ORM + D1, Tailwind v4, shadcn/Radix UI, TanStack Query, Zod. No backend/frontend split — everything in `src/`.

## Default behavior

When asked to implement any feature, fix, refactor, or behavior change, internally follow:

```text
Start with the spec. Do not implement first.

Find or create docs/specs/F<NN>-<slug>.md.
Define current behavior, desired behavior, edge cases, error states, test plan.
Add a Bug/Change Log entry draft.
Write failing tests in tests/unit/ (and tests/e2e/ for user-visible flows).
Implement the smallest correct change.
Run npm run verify (and npm run e2e for user-visible changes).
Update the spec to match final behavior.
Update docs/MVP_SCOPE.md if applicable.
Report what was changed, tested, and not verified.
```

## Rules

- Do not skip the spec because the change seems small.
- Do not write implementation before understanding current behavior — read the route handler/component, not just its file name.
- Do not modify tests only to make them pass, weaken assertions, or delete failing tests without explanation.
- Do not ignore edge cases or introduce unrelated refactors.
- Do not claim tests passed without running them.
- Do not leave specs stale.
- Do not introduce new libraries, frameworks, or abstractions unless the spec requires them or existing architecture clearly supports them.
- Prefer explicit behavior over clever abstraction.
- Preserve existing conventions unless there is a documented reason to change.
- Make minimal, reversible changes.
- Document assumptions and decisions in the spec.

## Ambiguity

If requirements are ambiguous, add an "Open Questions" section to the spec. If the ambiguity blocks correct implementation, ask. Otherwise, make the safest reasonable assumption, document it under Decisions, and continue. Never silently choose behavior that affects security, billing, permissions, data loss, or user-visible contracts.

## Verification

Run after every change:
```bash
npm run verify    # typecheck + lint + test:cov (100% on touched files)
npm run e2e       # for user-visible changes
```

## Codebase map

| Concern | Location |
|---------|----------|
| API route handlers | `src/app/api/**/route.ts` |
| Dashboard (mailbox-scoped) | `src/app/(dashboard)/` |
| Admin (account-scoped) | `src/app/(admin)/` |
| DB schema | `src/db/schema/index.ts` |
| Zod validators | `src/lib/validators.ts` |
| IDs (nanoid, prefixed) | `src/lib/ids.ts` |
| Auth (session cookie) | `src/lib/auth/` |
| Email handling | `src/lib/email/` |
| Cloudflare API client | `src/lib/cloudflare-api.ts` |
| Worker entry | `worker.ts` |
| Drizzle migrations | `drizzle/migrations/` |
| Unit/integration tests | `tests/unit/` (mirrors `src/` paths) |
| E2E tests | `tests/e2e/` (Playwright) |

Key conventions: DB via `getDb(env)`, IDs via `newId(prefix)`, auth via `requireUser()` (server) / `authFetch` (client), all request bodies Zod-validated, cross-tenant isolation mandatory on all multi-tenant endpoints.

## Where to start

- **Bootstrap:** run `./setup.sh` (copies `.dev.vars`, installs deps, applies local D1 migrations).
- **Orient:** [`docs/ARCHITECTURE.md`](./docs/ARCHITECTURE.md) — data-flow diagrams + where every behavior lives.
- **Pick work:** [`docs/AGENT_TASKS.md`](./docs/AGENT_TASKS.md) — self-contained tasks sized for one PR.
- **Full lifecycle:** [`docs/ENGINEERING.md`](./docs/ENGINEERING.md).
- **Contribution mechanics (commits, PRs, CLA):** [`CONTRIBUTING.md`](./CONTRIBUTING.md).
- **Security invariants:** [`SECURITY.md`](./SECURITY.md) — cross-tenant isolation is the strongest one.
- **Specs & registry:** [`docs/specs/`](./docs/specs/) and [`docs/MVP_SCOPE.md`](./docs/MVP_SCOPE.md).

---
> Source: [cschanhniem/lumimail](https://github.com/cschanhniem/lumimail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
