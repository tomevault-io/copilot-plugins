## favorite-eats-amount-kit

> Canonical name for the unified ingredient amount parse + format module


# `favoriteEatsAmountKit` (locked)

**Name:** The single “brain” for recipe ingredient amounts—**parsing** messy input into numbers when possible and **formatting** numbers for display—must be called **`favoriteEatsAmountKit`**.

**Exposure:** `window.favoriteEatsAmountKit` — `js/favoriteEatsAmountKit.js`.

**Integration:** Load after `js/utils.js` (and ideally `js/unitQuantityFormat.js` for grid glyphs). Load before `js/quantityDisplayPolicy.js` if you want `formatGlyphForAmount` to delegate at first paint; otherwise delegation applies once the kit script has run.

**Do not** introduce parallel global names for the same responsibility (e.g. a second “amount brain”). Extend this kit and its tests instead.

---
> Source: [spoonfloor/favorite-eats](https://github.com/spoonfloor/favorite-eats) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
