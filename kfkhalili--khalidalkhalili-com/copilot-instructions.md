## functional-and-typed

> Functional programming and strict typing; avoid any/unknown


# Functional Programming & Strict Typing

Follow these in all code (TypeScript/JavaScript and other languages where applicable).

## Functional programming

- Prefer **pure functions**: same inputs → same outputs, no side effects. Do I/O and mutation at the edges.
- Prefer **immutability**: don’t mutate existing data; return new values (e.g. new objects/arrays).
- Prefer **declarative** style (map, filter, reduce, composition) over imperative loops where it improves clarity.
- Keep **functions small and focused**; compose them instead of writing large procedures.
- Avoid **shared mutable state**; pass data in and return new state.

## Typing (TypeScript)

- Use **strict typing** everywhere. Enable strict mode and strict null checks if not already.
- **Do not use `any`.** Use a concrete type, a generic, or a narrow union. If something is truly dynamic, type it as narrowly as possible (e.g. `Record<string, number>` or a specific interface) instead of `any`.
- **Avoid `unknown`** unless you are immediately narrowing (type guards, assertions after checks). Prefer a concrete type or a union.
- Prefer **explicit return types** on functions and **explicit types** on parameters and variables when the type is not trivial or obvious from context.

## Examples

```ts
// Prefer: pure, typed, immutable
function addDebt(current: number, delta: number): number {
  return Math.max(0, Math.min(100, current + delta));
}

// Avoid: any, mutation, side effects in “logic”
function updateState(s: any) {
  s.debt += 1;  // mutation
  return s;
}
```

Apply these principles in both new code and when modifying existing code.

---
> Source: [kfkhalili/khalidalkhalili.com](https://github.com/kfkhalili/khalidalkhalili.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
