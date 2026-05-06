## testing

> Testing conventions — Vitest + happy-dom


# Testing

- **Vitest 4.x** + **happy-dom** for DOM tests
- Pragmatic — one `it()` covers a full scenario, not individual edge cases
- Mock data in `tests/utils/mocks.ts` (17 fake logs, no real data)
- Helpers in `tests/utils/` — test files contain only assertions
- All UI text must be tested for both PL and EN key parity
- Run: `npm test` (single), `npm run test:watch` (watch)

---
> Source: [maciejaszex/ndexplorer](https://github.com/maciejaszex/ndexplorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
