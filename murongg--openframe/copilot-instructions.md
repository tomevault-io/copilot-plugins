## openframe

> Operational guide for coding agents working in this repository.

# AGENTS.md

Operational guide for coding agents working in this repository.
Use this as the practical execution handbook for day-to-day tasks.

## Scope and Priority

- Repository type: `pnpm` workspace monorepo (`apps/*`, `packages/*`).
- Main product: `apps/desktop` (Electron + React + Vite + TypeScript).
- Shared schema source: `packages/db/schema.ts`.
- Shared provider definitions: `packages/providers`.

Priority order when guidance conflicts:

1. Explicit user instruction
2. `CLAUDE.md`
3. `AGENTS.md` (this file)
4. Future tool-specific rules (`.cursor/rules`, `.cursorrules`, `.github/copilot-instructions.md`)

## Quick Start

- Install deps (repo root): `pnpm install`
- Start app (repo root): `pnpm dev`
- Lint all workspaces (repo root): `pnpm lint`
- Type-check app: `pnpm -C apps/desktop exec tsc --noEmit`

Native dependencies (`electron`, `better-sqlite3`) are expected.
`apps/desktop` runs `electron-rebuild` for `better-sqlite3` in `postinstall`.

## Command Reference

Run from repo root unless noted.

- Dev app: `pnpm dev`
- Build app (all targets): `pnpm build`
- Build current platform only: `pnpm build:current`
- Build per target: `pnpm build:mac`, `pnpm build:win`, `pnpm build:linux`
- Lint all workspaces: `pnpm lint`
- DB generate (proxied): `pnpm db:generate`
- DB migrate (proxied): `pnpm db:migrate`

Direct `apps/desktop` equivalents:

- `pnpm -C apps/desktop dev`
- `pnpm -C apps/desktop build`
- `pnpm -C apps/desktop lint`
- `pnpm -C apps/desktop db:generate`
- `pnpm -C apps/desktop db:migrate`

## Testing Reality

There is currently no dedicated test runner configured (`test` script / Vitest / Jest is not committed).
Use type-check + lint as baseline validation:

- `pnpm -C apps/desktop exec tsc --noEmit`
- `pnpm -C apps/desktop lint`
- Single file lint: `pnpm -C apps/desktop exec eslint src/path/to/file.tsx`
- Multiple files lint: `pnpm -C apps/desktop exec eslint electron/foo.ts src/bar.tsx`

If a test framework is added later, prefer single-test execution (file or `-t` filter).

## Task Playbooks

### Renderer / UI Change

1. Implement changes in `apps/desktop/src/**`.
2. Ensure persistence still goes through preload-exposed `window.*API`.
3. Run:
   - `pnpm -C apps/desktop exec tsc --noEmit`
   - `pnpm -C apps/desktop exec eslint src/path/to/changed-file.tsx`
4. Optionally run `pnpm -C apps/desktop dev` for manual flow verification.

### Electron / Main Process Change

1. Implement in `apps/desktop/electron/**`.
2. Keep IPC contracts synchronized with renderer usage.
3. Run:
   - `pnpm -C apps/desktop exec tsc --noEmit`
   - `pnpm -C apps/desktop exec eslint electron/path/to/changed-file.ts`
4. Verify IPC call path end-to-end from renderer.

### Schema / Migration Change

1. Edit `packages/db/schema.ts`.
2. Run `pnpm -C apps/desktop db:generate`.
3. Commit schema change plus generated files under `apps/desktop/electron/migrations/`.
4. Run `pnpm -C apps/desktop exec tsc --noEmit`.

### i18n Change

1. Update both locale files:
   - `apps/desktop/src/i18n/locales/en.ts`
   - `apps/desktop/src/i18n/locales/zh.ts`
2. Keep key hierarchy aligned (`projectLibrary.*`, etc.).
3. Lint/type-check changed files.

### AI / Media Flow Change

1. Keep prompt payload types consistent (string or `{ text, images }`).
2. Ensure IPC-safe serialization across preload bridge (binary refs as number arrays).
3. Preserve existing queue/status transitions (`queued` -> `running` -> terminal).
4. Validate failure handling and user-facing error messaging.

## Architecture Invariants (Do Not Break)

- Renderer must not access DB or filesystem directly.
- Renderer persistence must go through APIs exposed in `apps/desktop/electron/preload.ts`.
- New entities must update the full chain in order:
  1. `packages/db/schema.ts`
  2. `apps/desktop/electron/handlers/*.ts`
  3. `apps/desktop/electron/preload.ts`
  4. `apps/desktop/electron/electron-env.d.ts`
  5. `apps/desktop/src/db/*_collection.ts` and UI usage
- Handler SQL must use raw `better-sqlite3` via `getRawDb()`, not Drizzle ORM query builder.
- Do not manually edit generated router file `apps/desktop/src/routeTree.gen.ts`.

## Type, Naming, and Data Shape Rules

- TypeScript is strict: keep strong typing, avoid `any` shortcuts.
- Prefer explicit payload/row types for IPC and DB.
- Persisted DB fields use `snake_case` (`project_id`, `created_at`).
- UI/transient values use `camelCase`.
- Use narrow string unions for enums where possible.
- Normalize external/AI values before DB writes.
- Naming:
  - Components: `PascalCase`
  - Functions/variables: `camelCase`
  - DB/IPC fields: `snake_case`
  - Collection files in `src/db`: `*_collection.ts`
  - Handler registration helpers: `registerXHandlers`

## Style and Module Organization

- Import groups:
  1. framework/library
  2. workspace/package
  3. local relative
- Use `import type` for type-only imports where practical.
- Match repo formatting:
  - single quotes
  - no semicolons
  - 2-space indentation
  - trailing commas where valid
- Prefer small focused functions and early returns.
- Add comments only when logic is non-obvious.

## Error Handling and UX Expectations

- Wrap async IPC/AI flows in `try/catch`.
- Show user-facing errors via i18n keys, not raw exception dumps.
- Preserve behavior on failure (no silent partial state mutation).
- Require explicit confirmation (`window.confirm`) for destructive actions.

## Database and Runtime Notes

- Migration files live in `apps/desktop/electron/migrations/`.
- Prefer idempotent migration SQL when startup compatibility matters.
- Runtime SQLite location is under Electron `userData` path (never hardcode absolute DB paths).
- Settings and AI config use `electron-store` JSON (`settings.json`), not SQLite.
- Vector search uses `sqlite-vec`; keep embedding dimension assumptions consistent with existing table setup.

## Common Pitfalls

- Forgetting to regenerate migration after editing `packages/db/schema.ts`.
- Updating handler/preload types but not `electron-env.d.ts`.
- Editing generated files (especially `routeTree.gen.ts`) manually.
- Letting renderer bypass IPC for convenience.
- Adding i18n keys in one locale only.

## Delivery Checklist (Before Hand-off)

1. Keep the change scoped; avoid unrelated refactors.
2. Do not revert unrelated dirty worktree files.
3. Inspect `git diff` for accidental edits.
4. Run relevant validation commands for touched files.
5. Report:
   - what changed
   - why
   - verification commands run
   - follow-up actions (if any)

---
> Source: [murongg/openframe](https://github.com/murongg/openframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-22 -->
