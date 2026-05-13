## testing

> - Use Jest with React Testing Library for frontend


# Testing Conventions

## Framework

- Use Jest with React Testing Library for frontend
- Mock external services, never make real API calls in tests

## Structure

- One test file per source file, co-located next to it.
  - Server / utility tests: `*.test.ts` (runs in the `node` Jest project).
  - Client component tests: `*.test.tsx` (runs in the `jsdom` Jest project).
- No `__tests__` folders — co-location only.
- Group tests with describe blocks by method or behavior.
- Use arrange-act-assert pattern.

## Shared test helpers

- Shared test utilities (render wrappers, fixture factories, mocks) live under `core/test-utils/` and are imported via the `@core/test-utils/*` alias.
- Create this folder on first need — avoid empty scaffolding.

## Examples to follow

---
> Source: [GuRuGuMaWaRu/nextjs_job-prep_ai](https://github.com/GuRuGuMaWaRu/nextjs_job-prep_ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
