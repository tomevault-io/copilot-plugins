## writing-tests

> Writing tests with Bun

# Writing tests (Bun)

- Prefer adding tests near the code you changed (co-located) before creating new suites.
- For bug fixes: add a failing test first, then implement the fix.
- Keep tests deterministic: avoid real network calls; prefer fixtures/mocks.
- Run tests in the relevant workspace (or repo root) with `bun test` / `bun run test` where available.

---
> Source: [nech-ai/proxed](https://github.com/nech-ai/proxed) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
