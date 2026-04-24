## prop-model

> Whenever you create or modify a feature, bugfix, or any functional code:

# CLAUDE.md

## Test Generation Policy
Whenever you create or modify a feature, bugfix, or any functional code:
1. Generate corresponding tests in the `backend/tests/` directory
2. Tests should cover happy path, edge cases, and failure cases
3. For NBA model code specifically, include tests for:
   - Data validation (player stats, game data, odds)
   - Model prediction accuracy assertions
   - API response structure
4. Run the tests after generating them and fix any failures before finishing
5. Do not consider a task complete until tests pass

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mithun-s14) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
