## rhdh-plugins

> - Install: `yarn install`

# Intelligent Assistant Plugin for RHDH

## Build & Test Commands

- Install: `yarn install`
- Build: `yarn build:all`
- Test all: `yarn test:all`
- Test single file: `yarn test -- path/to/test.ts`
- E2E tests: `yarn test:e2e`
- Lint: `yarn lint`
- Lint single file: `yarn lint --fix path/to/file.ts`
- Type check: `yarn tsc`
- Dev (modern): `yarn start`

## Key Conventions

- UI uses PatternFly components for the chat interface, not MUI
- The frontend has two variants: modern (new Backstage plugin API) and legacy (old API)

## Architecture (only non-obvious parts)

- `packages/app` uses the new Backstage frontend plugin API; `packages/app-legacy` uses the old API for backwards compatibility
- Backend uses Knex for database access; migrations live in `plugins/intelligent-assistant-backend/migrations/`
- `packages/` is strictly for the dev environment; do not add application code there

## PR Conventions

- All commits must have an `Assisted-by: <model>` footer below the sign offs

---
> Source: [redhat-developer/rhdh-plugins](https://github.com/redhat-developer/rhdh-plugins) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-25 -->
