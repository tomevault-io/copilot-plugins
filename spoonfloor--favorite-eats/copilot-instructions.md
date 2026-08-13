## favorite-eats-recipe-ingredient-amount-model

> Canonical recipe ingredient amount semantics


# Recipe Ingredient Amount Model

Recipe ingredient amount semantics live in
`window.favoriteEatsRecipeIngredientAmountModel` (`js/recipeIngredientAmountModel.js`).

Use this model for:

- recipe editor prefill/display
- recipe save payload construction
- Items recipe-derived quantities
- Shopping List plan-row quantities

Do **not** add new `quantity_max -> quantity_min -> quantity` precedence logic.

Canonical invariant:

- Scalar quantity wins over stale min/max endpoints.
- Explicit ranges use endpoint data and shop by max.
- Plain text amounts clear endpoints and do not produce shopping quantities.

The server boundary must preserve the same invariant via
`catalog.canonicalize_recipe_amount_columns()` on recipe ingredient tables.

---
> Source: [spoonfloor/favorite-eats](https://github.com/spoonfloor/favorite-eats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
