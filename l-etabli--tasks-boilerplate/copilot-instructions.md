## tasks-boilerplate

> - Use descriptive test names

# Testing

- Use descriptive test names
- Write unit tests for usecases, tested against in memory repositories
- We call test for adapters implementions, integration tests (for example, tests that would test a Postgres adapter of a repository). They should focus on the adapter in isolation.
- Follow Arrange-Act-Assert pattern (but do not comment those steps)
- Do not use mocks, use explicit dependency injection
- Test edge cases

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/l-etabli) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
