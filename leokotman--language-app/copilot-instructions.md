## language-app

> Keep test coverage at or above threshold on code changes


# Test coverage

When you add or change functionality (or touch code that affects coverage):

1. **Add or update tests** so that the changed code is covered, including **branch coverage** (both sides of conditionals, switch cases, optional chaining, etc.).
2. **Run coverage** before considering the change done: `npm run coverage`. To run format, lint, and coverage in one go before committing: `npm run precommit`.
3. **Meet the project threshold:** All four metrics (statements, branches, functions, lines) must be **≥ 70%** (see `vite.config.ts` → `coverage.thresholds` and `docs/TEST_COVERAGE.md`). If any metric is below 70%, add or adjust tests until the threshold is met.

Branches are often the metric that fails first; when adding tests, cover both outcomes of conditionals and handle edge cases (e.g. null/undefined, empty array, error path).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leokotman) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
