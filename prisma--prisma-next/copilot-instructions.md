## no-tautological-tests

> Avoid tautological tests that only restate fixture input


# Avoid Tautological Tests

Tests must verify behavior, not mirror object shape passed by the test itself.

## Rule

- Prefer assertions that validate transformation, branching, integration behavior, error handling, or side effects.
- Do not add tests that only confirm an object still contains fields copied directly from the test input.
- If coverage pressure encourages low-value assertions, tune coverage configuration instead of adding meaningless tests.

## Examples

**❌ Avoid**

```typescript
const value = makeThing({ a: 1, b: 2 });
expect(value).toMatchObject({ a: 1, b: 2 });
```

When `makeThing()` only stores the same input fields, this adds little signal and mostly tests fixture setup.

**✅ Prefer**

```typescript
const value = makeThing({ precision: 12 });
expect(value.normalize()).toBe('numeric(12)');
expect(() => makeThing({ precision: -1 })).toThrow();
```

These assertions validate behavior and error paths that can actually regress.

---
> Source: [prisma/prisma-next](https://github.com/prisma/prisma-next) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
