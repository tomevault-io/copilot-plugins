## vritti-core

> pnpm workspace monorepo containing the Vritti Core (Platforms) backend and frontend.

# vritti-core

pnpm workspace monorepo containing the Vritti Core (Platforms) backend and frontend.

## Apps

| App | Stack | Port | Path |
|-----|-------|------|------|
| core-server | NestJS + Fastify + Drizzle | 3000 | `apps/core-server/` |
| core-web | React + Rsbuild + Tailwind v4 | 3012 | `apps/core-web/` |

## External Dependencies

`@vritti/quantum-ui` and `@vritti/api-sdk` are linked via pnpm overrides (see `pnpm-workspace.yaml`). They live outside this repo at `../quantum-ui` and `../api-sdk`.

## Commands

```bash
pnpm dev              # Start both apps in parallel
pnpm dev:ssl          # Start both with HTTPS
pnpm build            # Build all projects
pnpm lint             # Lint all projects (Biome)
pnpm test             # Run unit tests
pnpm test:e2e         # Run e2e tests
pnpm check            # Biome check --write across all
nx graph              # Show project dependency graph
npx nx release        # Version bump + changelog (conventional commits)
```

## Testing

| Scope | Tool | Location |
|-------|------|----------|
| Server unit | Jest + SWC | `apps/core-server/` |
| Web unit/component | Vitest + Testing Library | `apps/core-web/` |
| Server e2e | Jest + Supertest | `e2e/core-server-e2e/` |
| Web e2e | Playwright | `e2e/core-web-e2e/` |

## Structure

```
vritti-core/
├── apps/
│   ├── core-server/     # NestJS backend
│   └── core-web/        # React frontend
├── e2e/
│   ├── core-server-e2e/ # API e2e tests
│   └── core-web-e2e/    # Browser e2e tests
├── libs/                  # Future shared packages
├── .claude/rules/         # Convention rule files
├── biome.json             # Shared linter/formatter
└── nx.json                # Nx config with versioning
```

## Conventions

See `.claude/rules/` for detailed pattern documentation:
- `swagger-docs.md` — API controller Swagger docs pattern
- `error-handling.md` — RFC 9457 exception patterns
- `backend-controller.md` — Controller thin layer rules
- `backend-service.md` — Service business logic rules
- `backend-repository.md` — Repository data access rules
- `backend-dto.md` — DTO organization (request/response/entity)
- `frontend-conventions.md` — Frontend patterns and component usage
- `frontend-hook.md` — TanStack Query hook conventions
- `frontend-service.md` — Axios service conventions
- `comment-style.md` — Comment style rules
- `export-conventions.md` — Export patterns

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vritti-ai-platforms) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
