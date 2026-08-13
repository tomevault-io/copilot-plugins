## shopping-list-no-adhoc-lines

> Shopping List has no UX to add free-text checklist rows; do not imply otherwise in docs, tests, or migration QA


# Shopping List — no “add a line” flow

**Product fact:** The **Shopping List** screen (`shoppingList.html`, `loadShoppingListPage` in `js/main.js`) does **not** let the user type and add a brand-new free-text row. Rows come from **Catalog + Plan → generation**, plus **edits, checks, removals, and overrides** on those rows.

**Schema vs UI:**

- **`list.manual_rows`** is a **Postgres table** for server-side list session data (and RPCs like `append_manual_shopping_list_row`). It is **not** documentation that end users compose ad-hoc lines on the Shopping List UI.
- **Durable “extra” shopping intent** belongs in **`plan.selected_items`** (Items / planner flows), not in imaginary Shopping List free-text entry.

**Agents must not:**

- Suggest manual QA, migration chunks, or user-facing copy that assumes “add another line,” “typed-only row,” or “manual line” **on the Shopping List page**.
- Describe multi-device success as requiring users to **add** manual checklist lines unless that UI is explicitly implemented and linked.

**Allowed:** References to `list.manual_rows`, merge logic, and narrow RPCs as **implementation / migration / server** concerns.

Canonical detail: `docs/catalog-plan-list-supabase.md` (list schema + UX note). Related rule: `.cursor/rules/shopping-variant-editor-known-issue.mdc` (different surface).

---
> Source: [spoonfloor/favorite-eats](https://github.com/spoonfloor/favorite-eats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
