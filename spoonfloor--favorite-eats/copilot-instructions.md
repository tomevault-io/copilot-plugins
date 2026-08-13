## high-before-low

> Prioritize structural fixes over patches; self-check for blind spots and local maxima before finishing


# High-before-low fix ethic

When fixing or extending this project, **always address higher-order problems before lower-order ones**.

## Priority order (high → low)

1. **Architecture & single source of truth** — unify duplicated state, APIs, and phase machines before adding more bridges or per-flow one-offs.
2. **Agreed product anatomy** — preflight / inflight / complete structure, scorecards, CTAs, and copy spec across Add, Clean, and Convert.
3. **Production parity** — wire real flows to the same APIs and UI patterns debug uses; don't leave debug-only paths as the reference implementation.
4. **Correctness & regressions** — broken toggles, missing logs, wrong counts, reset-on-new-run bugs.
5. **Polish & deferrables** — CSS tweaks, cache-bust params, copy nits, nice-to-have refactors.

## How to apply it

- Before implementing a fix, ask: *Is there a higher-level fix that would make this patch unnecessary?*
- Do not stack tactical patches on top of known architectural debt unless the user explicitly asks for a quick stopgap.
- When multiple issues exist, propose or execute **high items first**; mention deferred low items only if relevant.
- Prefer one canonical module (e.g. shared log feed, unified reset, single debug runner) over a third parallel code path.
- Minimize scope for the current task, but **do not** choose a minimal diff that perpetuates duplication when unification is the agreed next step.

## Self-check (required)

Agents **must** review their own work for blind spots and local maxima before treating a task as complete.

**Blind spots** — unverified assumptions: sibling flows still broken, production/debug divergence, silent skips, or patterns you didn't grep for.

**Local maxima** — solutions that optimize the immediate diff at the cost of the system: symptom fixes, a third parallel helper, or a minimal change that perpetuates known debt.

When you find either:

1. **Revise** toward the higher-order fix when scope and user intent allow.
2. If you must ship narrower, **state what you checked**, what you deferred, and why — do not present a local optimum as full resolution.

Self-check targets **system-level** mistakes (wrong layer, missed unification), not endless polish or unrelated refactors.

## Red flags (stop and go higher)

- New `*State.logLines` or debug-only sync helpers
- A fourth reset/toggle helper for the same overlay pattern
- Debug and production diverging on the same screen
- Fixing a symptom in one flow while siblings still use the old pattern
- Declaring done without asking whether a sibling flow or shared module still uses the old pattern
- Shipping a patch you know unification would supersede, without calling that out

---
> Source: [spoonfloor/favorite-eats](https://github.com/spoonfloor/favorite-eats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
