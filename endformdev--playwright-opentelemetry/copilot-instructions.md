## playwright-opentelemetry

> Always run unit tests from workspace root: `pnpm tsc`

# Agent Guidelines for playwright-opentelemetry

Always run unit tests from workspace root: `pnpm tsc`
Always typecheck from workspace root: `pnpm test`
Never make types.ts files.

## For trace-reporter

### Build & Test Commands
- `pnpm test:e2e` - Run e2e tests (playwright)
- `pnpm typecheck` - Type checking with tsc
- `pnpm format` - Format code with Biome

### Running Single Tests
- E2E test: `pnpm test:e2e example.spec.ts` or `pnpm test:e2e --grep "test name"`

---
> Source: [endformdev/playwright-opentelemetry](https://github.com/endformdev/playwright-opentelemetry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-24 -->
