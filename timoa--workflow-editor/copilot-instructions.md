## 03-testing

> Unit testing and property-based fuzz testing with Vitest and fast-check


# Testing

## Runner and Config

- **Vitest** only. Config in `vitest.config.ts`. Tests: `src/**/*.test.ts`, `src/**/*.test.tsx`.
- Run: `pnpm test` (single run), `pnpm test:watch` (watch). Use `node` environment unless you need `jsdom` for components.

## What to Test

- **Workflow parsing/serialization**: All public functions in `lib/` (e.g. `parseWorkflow`, `serializeWorkflow`). Test valid YAML, invalid YAML, missing `jobs`, round-trips, and edge cases (needs, uses, with, env).
- **Pure logic and types**: Any function that maps or validates workflow data.
- **React components**: Use Vitest + React Testing Library when testing UI that depends on props or user interaction. Prefer testing behavior, not implementation details.

## Structure

- Colocate tests: `parseWorkflow.test.ts` next to `parseWorkflow.ts`, or in a `__tests__` folder under `src/lib` if preferred.
- One describe per module or major function; nested describe for groups of cases.
- Descriptive names: `it('parses minimal workflow', ...)`, `it('returns errors when jobs is missing', ...)`.

## Assertions

- Use `expect()` from Vitest. Prefer specific matchers: `toEqual`, `toContain`, `toHaveLength`, `toMatch`. For errors, assert on message content or length: `expect(errors.length).toBeGreaterThan(0)`.
- For parse results, assert both `workflow` shape and `errors` when relevant.

## Examples

```typescript
// Good: minimal workflow + round-trip
it('round-trips minimal workflow', () => {
  const { workflow } = parseWorkflow(minimalWorkflow)
  const yaml = serializeWorkflow(workflow)
  const { workflow: again, errors } = parseWorkflow(yaml)
  expect(errors).toEqual([])
  expect(again.name).toBe(workflow.name)
})
```

- Avoid tests that only check "doesn't throw". Assert on return value or side effects.

## Property-Based Fuzz Testing

Use **[fast-check](https://fast-check.dev/)** for fuzz tests alongside regular unit tests. Fuzz tests live in `src/lib/workflow.fuzz.test.ts` (and sibling `*.fuzz.test.ts` files for other modules).

**When to add/update fuzz tests:**
- When adding or changing a public function in `lib/` that accepts arbitrary string or object input.
- When fixing a crash or unexpected-throw bug — add a property that would have caught it.
- When adding new fields to `Workflow`, `WorkflowJob`, or `WorkflowStep` — update the arbitraries in `workflow.fuzz.test.ts` so round-trip tests cover the new shape.

**Key invariants to always fuzz:**
1. **Safety**: the function never throws on arbitrary input.
2. **Shape**: the return value always has the expected structure.
3. **Round-trip stability**: `serialize → parse → serialize` produces the same result.

```typescript
import * as fc from 'fast-check'

// Safety invariant — never throw
it('never throws on arbitrary string input', () => {
  fc.assert(
    fc.property(fc.string(), (input) => {
      expect(() => parseWorkflow(input)).not.toThrow()
    }),
    { numRuns: 500 }
  )
})

// Round-trip — job IDs survive serialize/parse
it('job names survive a round-trip', () => {
  fc.assert(
    fc.property(workflowArbitrary, (workflow) => {
      const yaml = serializeWorkflow(workflow)
      const { workflow: parsed } = parseWorkflow(yaml)
      expect(Object.keys(parsed.jobs).sort()).toEqual(Object.keys(workflow.jobs).sort())
    }),
    { numRuns: 200 }
  )
})
```

- Keep arbitraries in the same fuzz test file; export them if other fuzz tests need them.
- Use `{ numRuns: 500 }` for safety invariants and `{ numRuns: 200 }` for heavier round-trip properties.

---
> Source: [timoa/workflow-editor](https://github.com/timoa/workflow-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-25 -->
