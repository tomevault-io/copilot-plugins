## tests

> - When writing tests, focus on decoupling each `it` or `test` block to ensure that they can run independently and concurrently. Tests must never depend on the action or outcome of previous or subsequent tests.

# Writing tests

- When writing tests, focus on decoupling each `it` or `test` block to ensure that they can run independently and concurrently. Tests must never depend on the action or outcome of previous or subsequent tests.
- When writing tests, especially in the __tests__/server directory, ensure that you avoid `pruneDatabase` calls.

---
> Source: [litefuse/litefuse](https://github.com/litefuse/litefuse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
