## no-src-css-mod

> Strictly avoid modifying the source document's CSS or injecting any styles (including `<style>` tags) into the capture target or its clones.

# No Source Document CSS Modification

Strictly avoid modifying the source document's CSS or injecting any styles (including `<style>` tags) into the capture target or its clones.

## Critical Mandates
- **Never** modify the fixture's CSS files (e.g., `examples/operator/fixtures/shop-demo/fixture.css`) to fix rendering issues.
- **Never** inject `<style>` tags or inline styles into the live document before or during capture.
- **Never** use SnapDOM plugins that inject styles into cloned nodes.
- **Never** perform recursive normalization passes that loop through all elements to lock dimensions or fonts.

## Acceptable Approaches
- Use SnapDOM's built-in options and configuration.
- Adjust the *container* of the capture target (e.g., the viewport or iframe container) only if necessary and if it doesn't affect the site's own rendering.
- Optimize the rendering pipeline (canvas handling, bitmapping) without touching the DOM.
- Ensure the environment (DPR, viewport size) is stable and integer-aligned using non-destructive means.

---
> Source: [pdufour/browser-use-wasm](https://github.com/pdufour/browser-use-wasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-29 -->
