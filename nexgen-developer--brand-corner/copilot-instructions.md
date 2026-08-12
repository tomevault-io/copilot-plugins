## flair-globo-infinite-scroll

> Flair badges on collection infinite scroll (Globo Filter)


# Flair + Globo infinite scroll

## Problem

Collection pages with infinite scroll show Flair badges on early products but not on products loaded later. Globo renders empty placeholders (`[data-flair-product-badge][data-product-id]`). `FlairApp.refreshProductBadges()` on `globoFilterRenderCompleted` alone misses appended nodes and Flair's ~250 product refresh limit.

## Required files (Brand Corner theme)

1. **`snippets/flair-collection-scroll.liquid`** — batch-fetches `flair-product-badges` via Section API, MutationObserver on product grids, retries `FlairApp.refreshProductBadges()`, exposes `window.refreshFlairCollectionBadges(scope)`.

2. **`layout/theme.liquid`** — render the snippet (do not use a bare `globoFilterRenderCompleted` listener only):
   ```liquid
   {% render 'flair-collection-scroll' %}
   ```

3. **`assets/main-*.js`** — in `fetchAndRenderNextPage`, append to `#js:results` or `#gf-products`, then call `window.refreshFlairCollectionBadges(resultsContainer)`.

## Do not edit

Flair-generated files: `sections/flair-product-badges.liquid`, `snippets/flair-product-badges.liquid`, `sections/flair-banners.liquid`, `snippets/flair-banners.liquid`.

## Port to another store (same theme)

Copy the three files/changes from `nexgen-developer/brand-corner` `main`. Grid IDs are `#gf-products` (Globo) and `#js:results` (native pagination).

## Test

Long collection → scroll to bottom → badges on first, middle, and last products.

## Commits

Use human-style messages only; no agent/cursor references.

---
> Source: [nexgen-developer/brand-corner](https://github.com/nexgen-developer/brand-corner) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
