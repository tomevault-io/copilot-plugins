## qafiyah

> This file provides coding standards and architectural rules for AI coding agents working in the Qafiyah repository. Follow these conventions in all generated or modified code.

# Agent Guidelines

This file provides coding standards and architectural rules for AI coding agents working in the Qafiyah repository. Follow these conventions in all generated or modified code.

## TypeScript

- `type` > `interface`. No `enum`/`any`. `readonly` everywhere. Named exports only.
- Enums → `as const` + derived union. Discriminated unions > boolean flags/optional fields.
- Branded types: `type UserId = string & { readonly __brand: unique symbol }`.
- Narrow via `unknown` + guards; no casts. Exhaustive `switch` with `never` fallback.
- Make invalid states unrepresentable. Lookup tables > `if/else`. `satisfies` > annotations when inference matters.

## Logic

- Pure by default. One abstraction level/fn. Inject deps as args; no globals/singletons.
- Compute derived; never store it. Single source of truth. Mutations at edges, pure transformations in core.
- Abstract only when: repeats 3×, hard to test, types allow invalid states, or immutable updates hurt.
- Delete dead code.

## Architecture

- Validate and normalize at entry points; trust types downstream. Fail loudly at boundaries.
- One primary export/file. 4+ exports → split. Co-locate until shared (2+ places → `types/`|`utils/`). `lowercase-kebab.ts`.
- Dependencies flow inward. Core has no knowledge of infra/transport/framework. High cohesion, low coupling; one reason to change per module. Prefer push (events/callbacks) over pull.
- Design interfaces for the caller: easy correct use, hard incorrect use. Stable interfaces > stable implementations. Never expose internals. Outputs: structured and predictable.

## Naming

- Skip abstraction if the name doesn't come naturally. Reveal intent (`getUsersWithExpiredPlans`). Domain = vocabulary.
- Consistent across boundaries. Booleans: `is`/`has`/`can`. Fns: verb+noun. Handlers: `handle`; props: `on`.
- No abbreviations unless universal (`req`, `res`, `id`). Similar names → similar behavior.
- Exceptions: React/DOM (`ref`, `props`), canvas `ctx`, Result `ok`/`err`, event params, loop `i`, dep-required names.

## Errors

- `Result<T,E>` for fallible logic; `throw` for truly unexpected. Log with enough context to reproduce.

## Testing

- Behavior not implementation. One concept/test. Unit → pure fns; integration → side-effectful flows.
- Mock only at boundaries (network/DB/time); never mock what you own.

## Comments

- Explain why, not what. If a comment explains what the code does, the code needs better naming.
- Last resort. Delete outdated ones, wrong comments are worse than none.
- TODOs must reference an issue or owner; orphaned TODOs get deleted.

---
> Source: [alwalxed/qafiyah](https://github.com/alwalxed/qafiyah) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-17 -->
