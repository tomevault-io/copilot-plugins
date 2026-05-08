## no-flowblock-renderer

> Do not use FlowBlock/manual canvas flow renderer; use Blitz/engine-driven HTML+CSS rendering in this project.


# Rendering: no FlowBlock fallback

Do **not** implement or route rendering through `FlowBlock`-based/manual canvas text layout for page rendering.

Rules:

- Do **not** add new usage of `FlowBlock` in primary render paths.
- Do **not** introduce fallback code that paints page content via handcrafted block/word-wrap rendering.
- Treat `FlowBlock` renderer as legacy/debug-only and avoid touching it unless explicitly requested.
- For page rendering, use the **Blitz / Stylo-driven** pipeline so CSS comes from a real engine path.

If rendering fails, debug the engine path (resource fetches, async WebGPU path, stylesheet loading) instead of switching back to manual rendering.

---
> Source: [pdufour/browserbrowserbrowser](https://github.com/pdufour/browserbrowserbrowser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
