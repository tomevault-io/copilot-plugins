## poly-backend-5ba74c95

> Fastify REST API. Read this before touching code.

# Agent instructions — `poly-backend`

Fastify REST API. Read this before touching code.

## Where to touch

- **Bootstrap**: `src/main.ts` (Fastify boot + listen).
- **App plugin**: `src/app/app.ts` (CORS + route registration).
- **Domains** live under `src/<domain>/` with this exact file split:
  - `routes.ts` — Fastify route handlers (`/api/<domain>`)
  - `service.ts` — business logic, takes a repo via constructor
  - `repository.ts` — in-memory data + access methods
  - `types.ts` — domain types (`Product`, etc.)
  - `*.spec.ts` — colocated tests (one per layer)
- Mirror this layout for new domains (e.g., `src/cart/`).
- Register new route plugins in `src/app/app.ts`.

## Tooling

- No monorepo, no Nx. Single project at the repo root.
- ESM (`"type": "module"`). Use `.js` extensions in TS imports (nodenext resolution).
- Tests: Vitest, globals on, node env. Config in `vitest.config.ts`.
- TS: `tsconfig.json` for build (excludes specs), `tsconfig.spec.json` for tests.
- ESLint via flat config in `eslint.config.mjs`.

## Conventions

- Repos export both the class and a default-wired singleton (e.g., `productsRepository`).
- Services accept a repo via constructor for testability, export a default-wired singleton too.
- No Swagger / OpenAPI. Don't reintroduce it.
- No `@fastify/autoload`. Register plugins explicitly in `app.ts`.

## Verifying changes

```sh
npm run lint
npm test
npm run build
npm run dev      # http://localhost:3000
```

---
> Source: [Poly-Demo/poly-backend-5ba74c95](https://github.com/Poly-Demo/poly-backend-5ba74c95) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-21 -->
