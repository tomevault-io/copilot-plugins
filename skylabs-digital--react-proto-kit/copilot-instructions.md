## react-proto-kit

> NPM library (`@skylabs-digital/react-proto-kit`) — NOT a full-stack app, NOT a monorepo.

# Project Context — react-proto-kit

## Type
NPM library (`@skylabs-digital/react-proto-kit`) — NOT a full-stack app, NOT a monorepo.

## Stack
- **Language:** TypeScript 5.x (strict mode)
- **Framework:** React 18+ hooks library
- **Validation:** Zod 4.x
- **Build:** Vite + vite-plugin-dts (ESM + CJS output)
- **Test:** Vitest + @testing-library/react + jsdom
- **Lint:** ESLint 9 (flat config) + Prettier
- **Release:** semantic-release via GitHub Actions → npm publish
- **Package Manager:** Yarn 1.22 (NEVER use npm)

## What does NOT exist
- No backend, no database, no Prisma, no Docker
- No deploy to infrastructure (QA, production)
- No Bruno smoke tests
- No E2E/Playwright tests
- No monorepo workspaces

## Key Commands
- `yarn ci` — lint + type-check + test + build (the full validation)
- `yarn test` — run all tests once
- `yarn test:watch` — watch mode
- `yarn test:coverage` — with coverage
- `yarn build` — Vite build (dist/)
- `yarn lint` / `yarn lint:fix` — ESLint
- `yarn format` / `yarn format:check` — Prettier
- `yarn type-check` — tsc --noEmit
- `yarn release` — semantic-release (CI only)

## Project Structure
```
src/
├── connectors/        # FetchConnector, LocalStorageConnector
├── context/           # GlobalStateProvider, InvalidationManager, DataOrchestratorContext
├── factory/           # createDomainApi, createSingleRecordApi
├── forms/             # useFormData, createFormHandler
├── helpers/           # schemas (createReadOnlyApi, etc.), seedHelpers
├── hoc/               # withDataOrchestrator
├── hooks/             # useList, useCreateMutation, useUpdateMutation, usePatchMutation, useDeleteMutation, etc.
├── navigation/        # useUrlParam, useUrlModal, useUrlDrawer, useUrlTabs, useUrlStepper, UrlModal, etc.
├── provider/          # ApiClientProvider
├── test/              # All test files
├── types/             # All TypeScript interfaces and types
├── utils/             # debug logging
└── index.ts           # Public API barrel export
```

## Public API
Everything exported from `src/index.ts` is the public API. Changes to exports are breaking changes.

## Testing
- Tests are in `src/test/` (not co-located)
- Navigation tests are in `src/navigation/__tests__/`
- Use `describe` / `it` / `expect` from Vitest
- Mock fetch with `vi.fn()` — never make real HTTP requests
- All 273 tests must pass before any PR

## Versioning
- Conventional commits → semantic-release auto-determines version bump
- `feat:` → minor, `fix:` → patch, `BREAKING CHANGE:` → major
- CHANGELOG.md auto-generated

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skylabs-digital) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
