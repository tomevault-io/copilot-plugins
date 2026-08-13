## shopping-variant-editor-known-issue

> Do not fix shopping variant row Shift+Enter/focus issues in main.js unless user explicitly asks


# Shopping variant row UI — out of scope

In `loadShoppingItemEditorPage` inside `js/main.js`, **do not** attempt fixes for:

- Shift+Enter to insert a new named variant row
- Focus jitter / `requestAnimationFrame` / `setTimeout` refocus loops
- Empty variant rows disappearing on blur
- `preventAutoDeleteOnInitialBlur` or similar guards

**Unless** the user explicitly asks to work on that behavior in the current task.

**Why:** Known issue — prior QA failed; easy to regress adjacent blur, duplicate detection, and deprecation UI. Unrelated to Supabase catalog migration (browser SQLite tail is complete).

Canonical write-up: `docs/migration-sweep.md` → section **Known issues / out of scope for agents**.

---
> Source: [spoonfloor/favorite-eats](https://github.com/spoonfloor/favorite-eats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
