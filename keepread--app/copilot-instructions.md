## app

> Focus Reader is a read-it-later app that supports both self-hosted single-user and multi-tenant SaaS deployments, deployed on Cloudflare (Workers, D1, R2, Pages), ingesting content from email newsletters, web articles, RSS feeds, and bookmarks into a unified reading interface.

# Agents Guide

Focus Reader is a read-it-later app that supports both self-hosted single-user and multi-tenant SaaS deployments, deployed on Cloudflare (Workers, D1, R2, Pages), ingesting content from email newsletters, web articles, RSS feeds, and bookmarks into a unified reading interface.

## Build & Test

```bash
pnpm install                              # Install all dependencies
pnpm build                                # Build all packages (turbo)
pnpm typecheck                            # Type-check all packages
pnpm test                                 # Run all tests (requires build first)
pnpm --filter @focus-reader/<pkg> test    # Single package
```

**Always verify with `pnpm build && pnpm typecheck && pnpm test` before committing.**

## Repository Structure

```text
focus-reader/
├── packages/
│   ├── shared/          # Types, constants, utilities (no deps)
│   ├── db/              # D1 migrations, typed query helpers
│   ├── parser/          # Email/article/RSS/PDF parsing, sanitization, export
│   └── api/             # REST API business logic
├── apps/
│   ├── web/             # Next.js 15 frontend on Cloudflare Pages
│   ├── email-worker/    # Cloudflare Email Worker
│   ├── rss-worker/      # Cloudflare Worker (scheduled RSS polling)
│   └── extension/       # Browser extension (WXT + React)
└── docs/
    ├── architecture/    # Live system architecture
    ├── development/     # Live contributor/workflow docs
    ├── product/         # Live product specs
    ├── reference/       # Live reference docs
    └── archive/         # Historical specs/plans
```

## Quick Reference

- **Entity types:** `packages/shared/src/types.ts` (D1 conventions: `number` for bools, `string` for timestamps/UUIDs)
- **Query helpers:** `packages/db/src/queries/` — every function takes `ctx: UserScopedDb` as first param
- **Multi-tenancy:** `packages/db/src/scoped-db.ts` — `UserScopedDb` wrapper (`{ db: D1Database, userId: string }`)
- **API pattern:** `apps/web/src/app/api/` routes are thin wrappers around `@focus-reader/api`
- **Authentication:** Mode-scoped via `AUTH_MODE`; see `apps/web/src/lib/auth-middleware.ts` and `packages/api/src/auth.ts`
- **Sanitization:** `packages/parser/src/sanitize.ts` — manual linkedom walker (NOT DOMPurify)

## Key Constraints

- **vitest** pinned to `~3.2.0` — `@cloudflare/vitest-pool-workers` does NOT support vitest 4.x
- **No DOMPurify** — incompatible with Workers; use `packages/parser/src/sanitize.ts`
- **D1 foreign keys** NOT enforced in production — implement cascading deletes in application logic
- **`UserScopedDb` everywhere** — only admin and child-entity query modules should use raw `D1Database`
- **Always `pnpm build` before `pnpm test`** — tests depend on `^build` in turbo.json
- **Import paths** use `.js` extensions (ESM convention)

## Live Docs

Use these as source-of-truth for current implementation:

- [Docs Index](docs/README.md)
- [Architecture Overview](docs/architecture/overview.md)
- [Web App Architecture](docs/architecture/web-app.md)
- [Cloudflare Runtime](docs/architecture/cloudflare-runtime.md)
- [Email Ingestion](docs/architecture/email-ingestion.md)
- [Product Spec (Current)](docs/product/product-spec-current.md)
- [UI Spec (Current)](docs/product/ui-spec-current.md)
- [Repo Structure (Reference)](docs/reference/repo-structure.md)
- [Browser Extension (Reference)](docs/reference/browser-extension.md)
- [Testing](docs/development/testing.md)
- [TypeScript Conventions](docs/development/typescript-conventions.md)

## Historical Docs

Historical intent/planning documents are preserved under `docs/archive/` and are not source-of-truth for current implementation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/keepread)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/keepread)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
