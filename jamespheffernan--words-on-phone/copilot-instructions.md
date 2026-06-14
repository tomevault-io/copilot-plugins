## words-on-phone

> - Every change must start with a failing Vitest or Cypress test.

# Cursor Rules for Words on Phone

- Every change must start with a failing Vitest or Cypress test.
- PRs must be ≤ 200 lines of code (excluding tests/docs).
- Commit only when all tests are green.
- PR template must confirm: npm run test && npx cypress run pass.
- Use vertical slices: UI, logic, tests, docs in one PR. 
- Before implementing any gameplay changes, consult docs/ruleset.md to ensure compatibility with core game mechanics (7-point scoring, hot-potato passing, timer-based rounds, clue-giving restrictions). If a change conflicts with established rules or introduces new gameplay elements, request user confirmation and clarify how it fits within the ruleset framework.

---
> Source: [jamespheffernan/words-on-phone](https://github.com/jamespheffernan/words-on-phone) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-14 -->
