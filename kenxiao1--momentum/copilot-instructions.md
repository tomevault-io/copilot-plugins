## momentum

> This file is the primary operating guide for coding agents working in this repository.

# Momentum (Codex Agent Notes)

## Scope

This file is the primary operating guide for coding agents working in this repository.

## Quick Start

- Install deps: `npm install`
- Dev server (Web): `npm run dev`
- Dev server (Tauri desktop): `npm run tauri dev`
- Production build (Web): `npm run build`
- Production build (Tauri): `npm run tauri build`
- Preview build: `npm run preview`
- Typecheck: `npm run typecheck`
- Lint: `npm run lint`

## Architecture Overview

### Three-Layer Architecture

1. UI Layer (`src/components/`, `src/app/`)
   - Pure presentational components.
   - Never access Supabase directly.
   - Use `useStorage()` for data operations.
2. Domain Logic Layer (`src/hooks/domains/`)
   - Business logic and state transitions.
   - Key hooks include: `useChainsDomain`, `useSessionsDomain`, `useBettingDomain`, `useRulesDomain`, `useRecycleBinDomain`, `useRsipDomain`, `useGroupDomain`, `useImportExportDomain`, `useCheckinDomain`, `usePetDomain`, `useSafeSaveChains`.
3. Infrastructure Layer (`src/storage/`, `src/infra/storage/supabase/`)
   - `MomentumStorage` defines the storage contract.
   - Implementations: `localStorageAdapter` (offline) and `SupabaseStorage` (cloud).
   - Use `storage.kind` (`'local' | 'supabase'`) to branch behavior.
4. Platform Abstraction Layer (`src/utils/platform.ts`, `src/utils/platform-adapters/`)
   - Detects runtime: `web` / `tauri-desktop` / `tauri-mobile`.
   - Adapters for notifications, window management, file I/O.
   - Tauri APIs are lazy-loaded via `src/utils/tauri-bridge.ts` to avoid Web build issues.
5. Tauri Backend (`src-tauri/`)
   - Rust backend for desktop/mobile native features.
   - Commands: `src-tauri/src/commands/` (notifications, window, file_ops).
   - Config: `src-tauri/tauri.conf.json`, `src-tauri/Cargo.toml`.

### Container + View Pattern

Large UI modules should follow container/view separation:

- `*Container.tsx`: state, side effects, orchestration
- `*View.tsx`: pure presentation

Examples: `AppShell`, `FocusMode`, `ChainEditor`.

## Coding Discipline

- Prioritize fixing concrete errors before proceeding to unrelated tasks.
- Keep comments minimal; remove dead code instead of commenting it out.
- Avoid `as any` assertions.
- Keep function cognitive complexity <= 15 (SonarJS budget).
- In `catch` blocks, prefer `normalizeUnknownError()` over `error as Error`.
- Use `logger` from `src/utils/logger.ts`; do not use `console.*` in app code.
- Use `src/utils/env.ts` (`isDev`, `isProd`, `isTest`, `isNonProd`) instead of direct `process.env.NODE_ENV` checks.
- Use `toast` from `src/utils/toast.ts` instead of `alert()`.

## Type System Notes

Key types are in `src/types/index.ts`:

- `Chain` is a discriminated union (`UnitChain | GroupChain`).
- `ChainType` includes: `'unit' | 'group' | 'assault' | 'recon' | 'command' | 'special_ops' | 'engineering' | 'quartermaster'`.
- `ChainDraft` uses `DistributiveOmit` for safe form handling.

When changing chain logic, branch by `type` explicitly to preserve union safety.

## Service Lifecycle

Services with explicit lifecycle should be managed centrally (in `AppShellContainer.tsx`):

- `forwardTimerManager`
- `exceptionRuleCache`
- `ruleStateManager`
- `performanceDashboard`
- `performanceMonitor`

## Local Static Analysis (Dev-Friendly, No Hooks)

All tooling is available via explicit `npm run ...` scripts (no pre-commit hooks). Run what you need while iterating.

### Formatting

- Format (write): `npm run format`
- Format (check): `npm run format:check`

### Code / Types

- ESLint: `npm run lint`
- ESLint (fix): `npm run lint:fix`
- TypeScript: `npm run typecheck`

### CSS / Docs

- CSS (Stylelint): `npm run lint:css` / `npm run lint:css:fix`
- Markdown (markdownlint-cli2): `npm run lint:md`
- Spelling (code): `npm run lint:spell`
- Spelling (docs): `npm run lint:spell:docs`

### Smell / Dependency Hygiene

- Knip (unused files/exports/deps): `npm run quality:knip`
- ts-prune (unused exports): `npm run quality:ts-prune`
- depcheck (unused/missing deps): `npm run quality:depcheck`
- One-shot report bundle: `npm run quality:smell-audit` (writes to `reports/quality/`)
- Licenses summary: `npm run quality:licenses`

### Security / SQL (Optional Locally)

These commands auto-skip if the underlying tool is not installed.

- npm audit (high+): `npm run security:npm-audit`
- Semgrep: `npm run security:semgrep` (recommended install: `pipx install semgrep`)
- SQL lint (Supabase migrations): `npm run lint:sql` (recommended install: `pipx install sqlfluff`)

## Testing (Vitest)

### Commands

- CI-smoke subset: `npm test` (uses `vitest.ci.config.ts`)
- Unit suite: `npm run test:all` (uses `vitest.config.ts`)
- Integration suite: `npm run test:integration` (uses `vitest.integration.config.ts`)
- "DB" suite: `npm run test:db` (uses `vitest.db.config.ts`)
- Performance suite: `npm run test:performance` (uses `vitest.performance.config.ts`)
- Watch: `npm run test:watch` (CI subset) or `npm run test:all:watch`
- Coverage: `npm run test:coverage`

### Test File Conventions

- Unit: `src/**/*.{test,spec}.{js,ts,jsx,tsx}` and `src/**/__tests__/**/*.{js,ts,jsx,tsx}`
  - Excludes `*.integration.test.*`, `*.db.test.*`, `*.performance.test.*`
- Integration: `*.integration.test.*` or `src/**/__tests__/**/*.integration.*`
- DB: `*.db.test.*` or `src/**/__tests__/**/*.db.*`
- Performance: `*.performance.test.*` or `src/**/__tests__/**/*.performance.*`

### Test Harness Notes

- Shared setup: `src/test/setup.ts` (mocks storage APIs, suppresses `console.*`)
- Integration setup: `src/test/setup.integration.ts`
  - Uses MSW handlers: `src/test/mocks/supabaseMocks.ts`
  - Mocks `import.meta.env` for Supabase config
  - Uses fake timers; advance timers when needed
- DB setup: `src/test/setup.db.ts`
  - Uses in-memory helpers: `src/test/utils/testDatabase.ts`
  - Mocks `src/lib/supabase.ts` to use a test client (no real Supabase required)

### When Adding or Changing Tests

- Prefer unit tests unless behavior depends on storage/network boundaries.
- If you add a new Supabase REST/RPC call used in integration tests, update `src/test/mocks/supabaseMocks.ts`.
- If you add or rename a storage method on `MomentumStorage`, update both implementations and add coverage in the relevant suite.

## Backend / Database (Supabase)

Momentum has no custom backend server: backend = Supabase (Postgres + RLS + SQL functions/RPC).

### Where the Database Lives

- Migrations: `supabase/migrations/*.sql` (PostgreSQL + RLS + functions)
- Schema reference: `docs/api/DATABASE_SCHEMA.md`
- Manual migration notes: `docs/guides/apply-migration.md` (Supabase Dashboard SQL Editor fallback)

### Supabase Client + Types

- Client wrapper: `src/lib/supabase.ts`
  - Uses env vars: `VITE_SUPABASE_URL`, `VITE_SUPABASE_ANON_KEY` (see `.env.example`)
  - Uses typed schema: `src/lib/database.types.ts` (`Database`)
- Lightweight config check (no SDK import): `src/utils/supabaseConfig.ts`

### App-Side Data Access Pattern (Must Follow)

- Storage contract: `src/storage/MomentumStorage.ts`
- Supabase implementation: `src/infra/storage/supabase/SupabaseStorage.ts`
  - Table modules: `src/infra/storage/supabase/{auth,chains,sessions,history,rsip,betting,checkin,taskTimeStats,userSettings}.ts`
  - Mapping layer: `src/infra/storage/supabase/mappers.ts`
- Local/offline implementation: `src/storage/localStorageAdapter.ts`
- UI must not talk to Supabase directly; go through `useStorage()` -> domain hooks (`src/hooks/domains/`) -> services (`src/services/`) -> `MomentumStorage`.

### When You Change the Database (Checklist)

1. Add a new migration in `supabase/migrations/` (do not edit old migrations in-place).
2. Keep RLS consistent: user-scoped tables should enforce `auth.uid() = user_id` (or equivalent) and avoid widening access.
3. Be careful with RPC functions:
   - Avoid function overloading (Supabase RPC can resolve the wrong overload).
   - Keep parameter names/types aligned with `.rpc()` calls (named args are used in the app).
   - For `SECURITY DEFINER` functions, do explicit auth checks (for example `target_user_id = auth.uid()`).
4. Update app code to match:
   - `src/lib/database.types.ts` (regenerate/update to match schema)
   - affected mappers + storage modules in `src/infra/storage/supabase/`
   - the `MomentumStorage` interface + `src/storage/localStorageAdapter.ts` if the interface changes
5. If a migration introduces new columns, keep Supabase storage resilient to older schemas when reasonable (many modules already include missing-column fallbacks).

### Supabase CLI (Typical Workflow)

- Apply migrations: `supabase db push` (or `supabase migration up`)
- Regenerate types (example): `supabase gen types typescript --schema public > src/lib/database.types.ts`
  - Adjust flags/project linkage to match your Supabase CLI setup.

### DB RPC Functions Used by the App

- Betting:
  - `place_task_bet`, `complete_task_with_betting`
  - Write sessions: `create_write_session`, `complete_write_session`
  - Defined/updated across `supabase/migrations/20250905*.sql` and `supabase/migrations/20250906*.sql`
  - Called from `src/infra/storage/supabase/betting.ts`
- Check-in:
  - `perform_daily_checkin`, `get_user_checkin_stats`
  - Defined in `supabase/migrations/20250904000000_add_daily_checkin_system.sql`
  - Called from `src/infra/storage/supabase/checkin.ts`

---
> Source: [KenXiao1/momentum](https://github.com/KenXiao1/momentum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
