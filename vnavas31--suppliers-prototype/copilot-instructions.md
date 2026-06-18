## suppliers-prototype

> When working in this repository, prioritize stable, reproducible, validated changes over isolated quick fixes.

# AGENTS.md

## Core working agreement

When working in this repository, prioritize stable, reproducible, validated changes over isolated quick fixes.

## Default behaviour

- Always inspect the existing implementation before proposing changes.
- Prefer small, safe, incremental changes.
- Do not assume a single failing example represents the full issue.
- When the user reports many failures, treat it as a systemic regression until proven otherwise.
- Do not ask for screenshots unless they are strictly necessary.
- Use logs, fixtures, tests, and reproducible jobs as the primary evidence.

## Regression handling

For regressions, especially tender processing regressions:

1. Identify the previous expected behaviour.
2. Identify the new or changed logic that may have introduced the regression.
3. Build or update a repeatable validation job.
4. Run it against a representative batch.
5. Capture all failures with enough debugging context.
6. Fix failures incrementally.
7. Re-run validation after each meaningful fix.
8. Do not stop after the first passing case.
9. Add regression tests.
10. Document remaining risks.

## Tender processing expectations

Tender processing must be:

- Correct
- Reproducible
- Observable
- Fast, but only after correctness is preserved

If fast processing conflicts with correctness, correctness wins. Clearly document the trade-off.

## Final response format

For non-trivial code changes, always report:

- What was broken
- What changed
- How it was validated
- Tests run
- Remaining risks

---
> Source: [vnavas31/suppliers-prototype](https://github.com/vnavas31/suppliers-prototype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
