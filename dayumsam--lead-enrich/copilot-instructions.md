## tests-required

> Require automated tests for all feature and behavior changes


# Tests Are Required

For every feature, bug fix, refactor, or behavior change, add or update automated tests in `tests/`.

- Write tests first when possible (red -> green -> refactor).
- Do not mark work complete unless `npm test` passes.
- If behavior changes, tests must assert the new behavior.
- If a bug is fixed, include a regression test reproducing the original failure.

## Minimum expectations per change

1. At least one test covers the primary success path.
2. At least one test covers an edge/error case if applicable.
3. Test names describe behavior, not implementation details.

## Examples

```js
// BAD: vague
it('works', () => {});

// GOOD: behavior-focused
it('rejects invalid CRM URLs in settings', () => {});
```

---
> Source: [dayumsam/lead-enrich](https://github.com/dayumsam/lead-enrich) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
