## datacleaner

> - Strict mode everywhere — no `any`, no `as` casts unless justified with comment


# Cross-Cutting Coding Conventions

## TypeScript
- Strict mode everywhere — no `any`, no `as` casts unless justified with comment
- `const` by default, `let` only when mutation needed, never `var`
- Functional and declarative patterns — no classes except where frameworks require
- Use descriptive names with auxiliaries: `isProcessing`, `hasError`, `canRetry`
- Prefer `Map`/`Set` over plain objects for lookups
- Never mutate input data — always return new objects/arrays

## File & Directory Naming
- Directories: lowercase-with-dashes (`upload-wizard/`, `job-status/`)
- React components: PascalCase (`UploadZone.tsx`, `JobPreview.tsx`)
- Utilities, services, configs: camelCase (`apiClient.ts`, `job.service.ts`)
- Schemas: `*.schema.ts` — Route handlers: `*.route.ts` — Tests: `*.test.ts`
- Constants: UPPER_SNAKE_CASE for values, camelCase for files

## File Structure Order (every file)
```
1. Imports (external → internal → relative)
2. Types / Interfaces
3. Constants
4. Helper functions (private)
5. Main export(s)
```

## Import Conventions
- Group imports: (1) node builtins, (2) external packages, (3) internal `@/` aliases, (4) relative
- Use path aliases: `@/` for `src/` in both frontend and backend
- Barrel exports (`index.ts`) for public module APIs only — never for internal files

## Validation
- **All external input** validated with Zod (API requests, env vars, file metadata, LLM responses)
- Schemas colocated with their domain: `backend/src/schemas/`, `frontend/src/validators/`
- Share schemas via `packages/shared/` when used by both frontend and backend

## Error Handling
- Early returns / guard clauses — never deeply nested if/else
- Custom error classes for typed error handling (see @error-handling.mdc)
- `pino` logger in backend (built into Fastify), never `console.log()`
- Toast notifications for user-facing errors in frontend

## Git Conventions
- Commit format: `type(scope): description` — e.g., `feat(pipeline): add phone cleaner`
- Types: `feat`, `fix`, `refactor`, `test`, `docs`, `chore`, `perf`
- One logical change per commit

## Environment Config
- All env vars validated with Zod at startup (see `backend/src/config/env.ts`)
- Use `.env.example` as the source of truth for required variables
- Never hardcode secrets, URLs, or config values

## Documentation
- JSDoc on all exported functions and complex helpers
- Brief module-level comment at top of each file explaining its role
- Keep rule files updated when architecture changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/inderjotpujara) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
