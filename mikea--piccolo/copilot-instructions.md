## piccolo

> This file governs how AI agents must behave when working on this repository. These rules are not suggestions. Violating them produces a codebase that cannot be trusted.

# AGENTS.md — Rules for AI Agents Working on Piccolo

This file governs how AI agents must behave when working on this repository. These rules are not suggestions. Violating them produces a codebase that cannot be trusted.

---

## The Prime Directive: Spec and Code Are One

**The specification in `specs/` is the ground truth for this project. The implementation must match the spec exactly. The spec must reflect the implementation exactly. They are the same thing expressed in two languages.**

Any divergence between the spec and the code — however small — is a defect. A correct implementation of the wrong spec is wrong. A correct spec that no implementation follows is useless.

---

## Rule 1: Update the spec before or alongside every code change

If you change a JSRPC method signature, update `specs/api.md`.
If you change how sessions are stored, update `specs/session-format.md`.
If you change a data flow, update `specs/data-flows.md`.
If you add a new component, add it to `specs/overview.md` and `specs/api.md`.

**Never commit code that makes the spec inaccurate. Never.**

---

## Rule 2: Update the code when the spec changes

If a spec document is updated — whether by you or by a human — find every affected implementation file and update it. Do not leave stale code that contradicts a current spec.

After updating any spec file, ask: *does any existing code now contradict this?* If yes, fix it before stopping.

---

## Rule 3: Verify sync after every change

After completing any task that touches either spec or code, perform a sync check:

1. Re-read the relevant spec section(s).
2. Re-read the affected implementation file(s).
3. Confirm that every interface, type, method name, parameter, and return type in the spec exists identically in the code, and vice versa.
4. If any discrepancy is found, fix it immediately — do not defer it.

This check is not optional. Do not mark a task complete until you have done it.

---

## Rule 4: `specs/api.md` is the single source of truth for all interfaces

All JSRPC interfaces, shared types, event unions, and result types are defined once in `specs/api.md`. No other spec document may redefine them — only reference them. No implementation file may define an interface that contradicts `specs/api.md`.

When in doubt about a type or interface, `specs/api.md` wins.

---

## Rule 5: No implementation detail may exist only in code

If you make an architectural decision during implementation — a caching strategy, a retry policy, a storage layout, a fallback behaviour — that decision must be documented in the relevant spec file before or at the same time as the code is written.

Code is not documentation. A decision that lives only in code will be lost, misunderstood, or contradicted by the next change.

---

## Rule 6: Treat spec drift as a build failure

If you discover that the spec and the code have diverged — even in a part of the project you were not asked to touch — you must fix the divergence or explicitly surface it as a blocking issue before proceeding with your current task.

Spec drift is never "someone else's problem to fix later." It compounds.

---

## Rule 7: Place helper functions at the end of implementation files

When adding or refactoring helper functions in source files, place them after the main class/function implementations (i.e. helper functions last in the file), unless language constraints require otherwise.

---

## Verification Checklist

Run this checklist after every non-trivial change:

- [ ] Every type and interface I added or changed exists in `specs/api.md` (if it is a JSRPC surface) or the relevant spec file.
- [ ] Every method I added, removed, or renamed is reflected in the spec.
- [ ] Every spec section I updated has a corresponding implementation change (or the implementation already matches).
- [ ] No spec document contains a type definition that contradicts another spec document.
- [ ] `specs/overview.md` accurately lists all components that exist in the implementation.
- [ ] The document index in `specs/overview.md` links to all current spec files.

---

## Why this matters

Piccolo is intentionally AI-implemented. The human's role is to drive direction and verify correctness — not to manually reconcile spec and code after the fact. If an AI agent allows the spec and code to drift, the human has no reliable source of truth to verify against. The entire model breaks down.

Keeping spec and code in sync is not a housekeeping task. It is the core discipline that makes AI-driven development trustworthy.

## Code verification

- run `just all` to verify all the changes before finishing work

## Type Discipline

- Avoid `unknown` in application/domain typing when a concrete SDK or API type exists.
- For AI message parts, use exported SDK types (e.g. `AssistantContent`, `ReasoningPart`, `ToolCallPart`, `ToolResultPart`) re-exported from `@piccolo/api`.
- Type guard helpers must return concrete predicates (`value is SomeSpecificPartType`), not broad/object-shaped fallbacks.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mikea)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mikea)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
