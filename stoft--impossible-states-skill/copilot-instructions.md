## impossible-states-skill

> Guides designing types and domain models so invalid states are impossible. Use when modeling state machines, invariants, optional vs required fields, or when editing TypeScript (.ts/.tsx), Gleam (.gleam), Elm, or F#. Covers discriminated unions, opaque/branded types, smart constructors, and valid state transitions.


# Impossible states

Use this skill when designing or reviewing types and domain models so that invalid or inconsistent states cannot be represented. Follow the workflow: load the required references, then load any optional references that match the task.

## When to use this skill

- Designing or refactoring types (state, status, optional fields, invariants).
- Modeling state machines, workflows, or multi-step processes.
- Reviewing code for “impossible states” (e.g. cash payment with card number, door open and locked).
- Working in TypeScript/TS, Gleam, Elm, F#, or similar typed languages.

## Must-read core references (required)

Before implementing or reviewing domain models for impossible states, read and keep in context:

- [references/overview.md](references/overview.md) — What “impossible states” means and why it matters.
- [references/impossible-data-structures.md](references/impossible-data-structures.md) — Optional fields, state/status modeling, mutually exclusive/inclusive fields, illegal combinations.

## Load when the task requires them

Read only when the user’s task involves that area:

- **Async/loading/error UI** → [references/hidden-failure-state.md](references/hidden-failure-state.md)
- **Non-empty collections, intervals, positive numbers, context dependency** → [references/constrained-values.md](references/constrained-values.md)
- **Hiding internals (IDs, private fields)** → [references/private-data.md](references/private-data.md)
- **State machines, workflows, valid transitions** → [references/valid-transitions.md](references/valid-transitions.md)
- **TypeScript/TS** → [references/typescript-techniques.md](references/typescript-techniques.md)
- **Gleam** → [references/gleam-techniques.md](references/gleam-techniques.md)

## When unsure

Prefer modeling with sum types / discriminated unions so each variant has only the fields that make sense for that variant. Avoid optional fields that create invalid combinations; use smart constructors or opaque/branded types for constrained values.

---
> Source: [stoft/impossible-states-skill](https://github.com/stoft/impossible-states-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
