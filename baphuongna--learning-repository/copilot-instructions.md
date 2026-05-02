## learning-repository

> Guide for autonomous coding agents in this repository.

# AGENTS.md
Guide for autonomous coding agents in this repository.

## Repository Snapshot
- Monorepo with `pnpm` workspaces and `turbo`.
- Frontend: `apps/web` (Next.js 14, TypeScript, Tailwind CSS).
- Data package: `packages/data` (Prisma schema, SQLite dev workflows).
- Backend: `services/rust-doc-service` (Rust, Axum, SQLx, SQLite).
- Package manager: `pnpm@9.0.0`.
- Rust edition: `2024`.
- Workspace root: `D:\my\learning-repository`.

## Important Paths
- `apps/web/app/*` (routes and layouts)
- `apps/web/components/*` (UI components)
- `apps/web/lib/api/*` (web API clients)
- `apps/web/components/shared/ui/*` (shared primitives)
- `packages/data/prisma/*` (schema and seed)
- `services/rust-doc-service/src/*` (backend source)
- `services/rust-doc-service/src/http/routes/*` (HTTP handlers)
- `services/rust-doc-service/src/core/*` (core infra)
- `services/rust-doc-service/src/domains/*` (domain logic)

## Cursor/Copilot Rule Files
- Checked `.cursorrules`: not found.
- Checked `.cursor/rules/`: not found.
- Checked `.github/copilot-instructions.md`: not found.
- If these files are added later, they override this guide.

## Build, Lint, Test Commands
Run from repository root:

```bash
pnpm install
pnpm build
pnpm lint
pnpm test
pnpm dev:web
pnpm dev:rust
pnpm db:generate
pnpm db:push
pnpm db:migrate
pnpm db:studio
pnpm --filter data db:seed
```

## Package-Scoped Commands
Web app:

```bash
pnpm --filter web dev
pnpm --filter web build
pnpm --filter web start
pnpm --filter web lint
```

Data package:

```bash
pnpm --filter data db:generate
pnpm --filter data db:push
pnpm --filter data db:migrate
pnpm --filter data db:studio
pnpm --filter data db:seed
```

Rust service:

```bash
cargo run --manifest-path services/rust-doc-service/Cargo.toml
cargo build --manifest-path services/rust-doc-service/Cargo.toml
cargo test --manifest-path services/rust-doc-service/Cargo.toml
```

## Running a Single Test (Important)
Rust tests currently live in `services/rust-doc-service/src/tests.rs`.

```bash
# By test-name substring
cargo test --manifest-path services/rust-doc-service/Cargo.toml creates_user_and_reads_profile_count

# Exact single test
cargo test --manifest-path services/rust-doc-service/Cargo.toml creates_user_and_reads_profile_count -- --exact

# Exact single test with logs
cargo test --manifest-path services/rust-doc-service/Cargo.toml creates_user_and_reads_profile_count -- --exact --nocapture

# Filter by topic/module keyword
cargo test --manifest-path services/rust-doc-service/Cargo.toml folder
```

Notes:
- No dedicated frontend test runner is configured (no Vitest/Jest/Playwright configs).
- Frontend verification is `lint + build + manual checks`.
- Root `pnpm test` uses `turbo run test`; backend confidence depends on `cargo test`.

## Verification Matrix
- Frontend-only changes:
  - `pnpm --filter web lint`
  - `pnpm --filter web build`
- Prisma/data changes:
  - run touched `pnpm --filter data db:*` commands
- Rust changes:
  - `cargo build --manifest-path services/rust-doc-service/Cargo.toml`
  - `cargo test --manifest-path services/rust-doc-service/Cargo.toml`
- Cross-cutting changes:
  - run both frontend and Rust verification
- Always report actual command outcomes.

## Architecture and Boundaries
- `apps/web` is the user-facing app.
- `services/rust-doc-service` is the active backend runtime.
- `packages/data` is shared schema/data tooling, not runtime API.
- Keep route handlers thin; move business logic into `core`/`domains`.
- Preserve soft-delete behavior (`status = 'DELETED'`) where applicable.

## Code Style Guidelines

### Imports and Modules
- Group imports: framework/runtime, third-party, internal alias/relative.
- Follow nearby import ordering; avoid reorder-only diffs.
- Remove unused imports.
- Use `import type` for TypeScript type-only imports.
- Keep files single-responsibility.

### Formatting
- Follow existing formatting and lint rules.
- In web code, match local style: single quotes and semicolons.
- Prefer small functions and early returns.
- Keep nesting shallow.
- Add comments only for non-obvious logic.
- Preserve meaningful Vietnamese comments already in code.

### TypeScript and Frontend
- Keep `strict: true` behavior in `apps/web/tsconfig.json`.
- Avoid `any`; prefer explicit interfaces/types and narrow DTOs.
- Use `z.infer<typeof schema>` for Zod-backed forms.
- Use alias `@/*` when it improves readability.
- Centralize API calls in `apps/web/lib/api/*`.
- Preserve existing 401 behavior in API client (clear auth and redirect `/login`).
- Add `'use client'` only when client-only hooks/APIs/events are required.

### Rust and Backend
- Use `AppResult<T>` and typed `AppError` for HTTP-facing paths.
- Keep SQLx access in repository-style modules.
- Validate input at boundaries and fail fast.
- Prefer explicit structs/enums over ad hoc tuple/map shapes.
- Keep config in `src/core/config.rs`; DB setup in `src/core/database.rs`.
- Return safe client-facing errors; never expose SQL, stack traces, tokens, or secrets.

### Naming Conventions
- React component names/files: `PascalCase`.
- Functions, variables, hooks, helpers: `camelCase`.
- Type aliases/interfaces/Rust structs/enums: descriptive `PascalCase`.
- Constants: uppercase only for true constants.
- Prefer explicit domain terms over abbreviations.

### Error Handling and Security
- Handle async failures where UI state must recover.
- Use `finally` to clean loading/submitting state.
- In Rust, map errors to clear variants (`BadRequest`, `Forbidden`, `NotFound`, `Internal`, etc.).
- Log internal failures with context, but keep sensitive data out of logs/responses.
- Never hardcode secrets; use environment variables.
- Validate and sanitize user input.

## Agent Workflow
Before editing:
- Read nearby files and mirror local conventions.
- Determine whether work is frontend, data, backend, or cross-cutting.

During editing:
- Keep scope tight; avoid unrelated cleanup.
- Preserve backward compatibility unless requested otherwise.

Before finishing:
- Run relevant build/lint/test commands for touched areas.
- Report changed files, behavior impact, and verification results.
- Mention gaps, assumptions, or follow-up work.

---
> Source: [baphuongna/learning-repository](https://github.com/baphuongna/learning-repository) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
