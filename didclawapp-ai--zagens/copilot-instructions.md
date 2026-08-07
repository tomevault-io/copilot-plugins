## code-organization

> File size, module boundaries, and where to put new code


# Code organization

- **Soft cap ~1000 lines** per implementation file (`.rs`, `.tsx`, `.ts`). Prefer **splitting** (submodules, `mod foo`, smaller components, `lib/*` helpers) **before** a file grows far past that.
- **New work:** add **new files** or focused modules instead of appending large blocks to already-large sources.
- **Legacy monoliths:** some paths (e.g. very large `runtime_api.rs`–style files) are **grandfathered**. Do **not** one-shot rewrite them; only **incrementally** extract when you already change that area.
- **Layout:** follow each crate’s existing directories (`crates/<name>/src/`, `web-ui/src/components/`, etc.); match naming and import style of neighboring code.

---
> Source: [didclawapp-ai/zagens](https://github.com/didclawapp-ai/zagens) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
