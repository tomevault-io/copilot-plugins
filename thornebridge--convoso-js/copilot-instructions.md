## convoso-js

> You are working on convoso-js, an unofficial TypeScript SDK for the Convoso API.

You are working on convoso-js, an unofficial TypeScript SDK for the Convoso API.

## Key Facts
- Zero runtime dependencies, native fetch (Node 18+)
- All API endpoints are POST with application/x-www-form-urlencoded bodies
- auth_token is injected by HttpClient — never in resource method params
- Response shapes match the raw API exactly (no normalization)
- Dual format: ESM + CJS with full .d.ts declarations

## Architecture
- src/client.ts — Convoso class composes all 16 resources
- src/http.ts — HttpClient wraps fetch, injects auth_token, retry + backoff, hooks
- src/errors.ts — ConvosoError → ConvosoApiError / ConvosoHttpError
- src/error-codes.ts — 44 known Convoso error codes with descriptions
- src/resources/base.ts — BaseResource holds HttpClient ref
- src/resources/*.ts — 16 resource classes, one per API section
- src/types/*.ts — Per-resource param/response types with JSDoc

## Commands
- npm run typecheck — tsc --noEmit
- npm run build — tsup → dist/ (ESM + CJS + .d.ts)
- npm test — vitest run
- npm run lint — ESLint
- npm run format — Prettier

## Conventions
- undefined/null params are stripped before encoding
- Paginated resources have *All() async generator methods
- Every type field has JSDoc documentation
- Tests use vitest with globals enabled

## Adding a New Resource
1. Create src/types/new-resource.ts with param/response interfaces
2. Export from src/types/index.ts
3. Create src/resources/new-resource.ts extending BaseResource
4. Add to Convoso class in src/client.ts
5. Add tests in tests/resources/
6. Add API reference docs in docs/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thornebridge) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
