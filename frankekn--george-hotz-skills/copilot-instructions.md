## george-hotz-skills

> Radical-simplicity coding guidelines distilled from George Hotz's public philosophy. The reflex: **complexity is the enemy — remove before you add.** Merge with project-specific instructions as needed.

# AGENTS.md

Radical-simplicity coding guidelines distilled from George Hotz's public philosophy. The reflex: **complexity is the enemy — remove before you add.** Merge with project-specific instructions as needed.

**Tradeoff:** these bias toward deletion and narrow interfaces over speed and extensibility. For trivial tasks, use judgment. Never delete a real invariant (money, auth, data-integrity, lifecycle) to "simplify" — that's a bug, not a cleanup.

## 1. Complexity is the enemy

**More capability at equal or lower complexity is the only real win.** Adding a feature by adding a layer is a loss.

- Solve the actual request with the least machinery that works.
- No speculative abstractions, configurability, or future-proofing nobody asked for.
- Smell: a new manager/handler/factory/adapter to do one thing → stop, ask what to remove instead.

## 2. You have never refactored enough

**Line count is a number you drive down. The best change is often a negative diff.**

- Delete-first: before touching a subsystem, ask "can this not exist?"
- Collapse duplicated logic to one source of truth; delete the glue that synced the copies.
- Prove something is dead before deleting it (check the callers). LOC is debt, not output.

## 3. Wide interfaces mean your abstraction is wrong

**A wide interface is a design signal, not a styling nit.**

- When an interface grows wide, especially toward five arguments, fix the boundary — split the responsibility, or pass one well-shaped value.
- Don't paper over it with an options/`**kwargs` bag so the count "looks" smaller.

## 4. Understand the whole stack — nothing is magic

**Don't reason about what code "should" do — look at what it does.**

- Build the smallest runnable unit, print/inspect its real inputs and outputs, check against expectation, then stack more on top.
- Read the definition and the callsites before concluding.
- Don't declare code "works" or "is safe" without seeing it; don't cargo-cult a pattern you don't understand.
- AI-generated code must be read and validated line by line; speed is not correctness.

---

**These guidelines are working if:** diffs trend smaller, the agent proposes deleting/collapsing before adding, interfaces stay narrow, and claims come with an observed value instead of "this should work."

---
> Source: [frankekn/george-hotz-skills](https://github.com/frankekn/george-hotz-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
