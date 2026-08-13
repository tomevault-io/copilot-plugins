## shopping-state-known-fragility

> Triage checklist and named silent-failure traps for shopping plan + Supabase round-trip bugs


# Shopping state — read this before guessing

**Most bugs in this surface look like timing problems but are silent-write problems.** Get one piece of real evidence before changing code.

## Triage checklist (do this first)

1. **Browser console** — paste the user's stack trace verbatim, don't paraphrase. Today's `recipeId is not defined` crash fell in 30 seconds from a stack trace.
2. **Network tab** — what's the body of the `save_shopping_state` request and response? "No request" is itself evidence.
3. **Supabase tables** — query `plan.selected_recipes` / `plan.selected_items` directly. "Server has X" vs "server has nothing" halves the suspect list.

If none of those three exist, ask for them. Do not speculate further.

## Named silent-failure modes (grep these first)

Each one can swallow a write or hand back stale data **with no toast and no rejection**:

- `skipDuplicateRemotePlanSave` — `JSON.stringify` compare in `persistShoppingPlan`; skips the RPC entirely when the next normalized plan stringifies identical to the previous. A real user change that the normalizer flattens looks like a duplicate.
- `shoppingStateRemoteWriteSuppressed` — `queueSaveShoppingStateToDataService` early-returns at the top. Set true during hydrate apply; if user input fires inside that window, the save never queues.
- `shoppingPlanRemoteSaveInFlight` — gates hydrate apply. Useful, but a stuck counter would silently block hydrates.
- `shoppingStateRemoteApplyGeneration` — supersedes in-flight applies. Same risk.
- `recipeDetailResolvedCache` / `recipeDetailInflight` — in-memory cache in `supabaseAdapter.js`. Read-back after a write returns the **pre-write** entry unless explicitly invalidated. `saveRecipe` invalidates; other writes that affect a cached recipe may not.

## Anti-patterns from past failed attempts

- "Must be a timing/ordering issue" — often the data was never sent at all. Check the network tab before touching hydrate logic.
- "Add a toast on the catch path" — toasts only fire on rejection. They do not catch silent skips, suppressed queues, hung fetches, or succeed-with-wrong-body.
- "Mirror servings to roots instead of merged" — assumes the persist + RPC pipeline is intact. Fix the pipeline first; the root/merged distinction is downstream.

## Codebase landmarks

- `js/main.js` is ~26k lines. Use `rg` / `grep` aggressively. Two near-identical closures with different variable names exist in the same file family — scope errors like the May 2026 `recipeId` typo are easy.
- `walkRecipe` exists **twice** in `supabaseAdapter.js` with different parameter conventions (`recipeId` vs `normalizedRecipeId`). If you edit one, look at the other.
- `loadRecipeDetail` cache must be invalidated on every write that changes its return value. `saveRecipe` does this via `invalidateRecipeDetailCache`. Pattern-match if you add a new write.

## Recent fix landmarks

- `19387c9` — await-hydrate on Recipes boot, in-flight save guard
- `b3414aa` — `loadRecipeDetail` cache invalidation after `save_recipe`; `walkRecipe` scope fix

## The one-sentence moral

**Get one piece of real evidence — stack trace, network body, or DB row — before changing code.**

Companion narrative: `docs/agent-handoff-shopping-state.md`.

---
> Source: [spoonfloor/favorite-eats](https://github.com/spoonfloor/favorite-eats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
