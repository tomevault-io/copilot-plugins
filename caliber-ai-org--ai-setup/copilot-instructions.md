## testing

> Vitest testing patterns for Caliber


- Vitest config: `vitest.config.ts` · Setup: `src/test/setup.ts`
- Run all: `npm run test` · Single: `npx vitest run src/scoring/__tests__/accuracy.test.ts`
- Coverage: `npm run test:coverage` (v8 provider)
- LLM calls globally mocked in `src/test/setup.ts` — never call real providers
- Use `vi.mock()` for `fs`, `child_process` · `vi.mocked()` for type-safe assertions
- Tests colocated: `src/llm/__tests__/config.test.ts` tests `src/llm/config.ts`
- Pattern: `describe` → `beforeEach(vi.clearAllMocks)` → `it` with `expect`
- Environment vars: save in `beforeEach`, restore in `afterEach`

---
> Source: [caliber-ai-org/ai-setup](https://github.com/caliber-ai-org/ai-setup) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-19 -->
