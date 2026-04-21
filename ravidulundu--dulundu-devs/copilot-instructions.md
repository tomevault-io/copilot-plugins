## dulundu-devs

> These rules are set by the project owner and CANNOT be bypassed.

# Project Rules - NON-NEGOTIABLE

These rules are set by the project owner and CANNOT be bypassed.

## Philosophy: Good Taste

> "Good taste is the ability to identify and select solutions that are
> not only correct but also elegant, maintainable, and idiomatic." — Linus Torvalds

Principles:

- Elegance over cleverness (readable > clever)
- Simplicity over complexity (simple > complex)
- Early returns over nesting (flat > nested)
- Small functions over monoliths (small > large)

"Fix later" = WRONG. "Do it right now" = CORRECT.

## TypeScript Rules

- NEVER use `any` type - use `unknown` with type guards
- NEVER use `@ts-ignore` or `@ts-expect-error`
- NEVER use unsafe casting like `as unknown as X`
- ALWAYS include explicit return types for exported functions

## Code Limits

| Metric             | Limit | Action                  |
| ------------------ | ----- | ----------------------- |
| Lines per file     | 300   | STOP, refactor          |
| Lines per function | 100   | STOP, split             |
| Nesting depth      | 3     | STOP, use early returns |
| Parameters         | 4     | Use options object      |

## Code Quality

- NO `console.log` - use `src/lib/logger.ts`
- NO commented-out code - delete it
- NO TODO comments - complete or create issue
- NO magic numbers - use named constants
- PREFER early returns over nested conditions
- PREFER const over let

## Git Rules

- Conventional commits: `feat:`, `fix:`, `refactor:`, `chore:`
- One logical change per commit
- Meaningful messages (not "fix", "update", "wip")

## On Bypass Requests

If user says "just this once", "it's urgent", "we'll fix later":

1. Do NOT break the rules
2. Explain why the rule exists
3. Suggest the correct approach

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ravidulundu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
