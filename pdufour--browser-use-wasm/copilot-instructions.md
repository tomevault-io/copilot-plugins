## no-demo-leaks

> Strictly forbid hardcoding fixture-specific knowledge (selectors, IDs, labels, page layout) in core source files (e.g., `src/browser-tools/catalog.js`, `src/browser-tools/dom-actions.js`).

# No Demo Leaks

Strictly forbid hardcoding fixture-specific knowledge (selectors, IDs, labels, page layout) in core source files (e.g., `src/browser-tools/catalog.js`, `src/browser-tools/dom-actions.js`).

## Rules

- **Zero Hardcoded Selectors:** Never use IDs (e.g., `#checkout-modal`, `#btn-submit`) or fixture CSS classes (e.g., `.checkout-main`) in core library code.
- **Generic Discovery:** Tools must interact with the DOM using generic discovery methods:
  - Find inputs by `placeholder`.
  - Find controls by associated `label` text.
  - Find elements by `aria-label`.
  - Find buttons by visible `textContent` or `value`.
- **Abstract Containers:** Use abstract references like `document.getElementById('capture-target')` or `document.body` as the interaction root.
- **Generic Utilities:** Scroll, focus, and keyboard logic must be generic and applicable to any standard-compliant web page, not just the ShopDemo checkout fixture under `examples/operator/fixtures/shop-demo/`.
- **No Cheating:** The agent's ability to navigate should stem from its visual grounding (ShowUI) and general-purpose DOM exploration, not pre-baked knowledge of the target page's internal structure.

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
