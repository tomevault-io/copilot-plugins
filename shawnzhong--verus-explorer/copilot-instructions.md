## verus-explorer

> - Keep functions short. Split when they grow or mix concerns.

# AGENTS.md

- Keep functions short. Split when they grow or mix concerns.
- No duplication — extract a shared helper the second time you'd write the same logic.
- Delete deprecated code in the same change that obsoletes it. No "just in case."
- Refactor freely: rename, move, split, merge, change signatures. No backwards compatibility, no shims, no re-exports for old names. Update every call site.
- Think first, then pick the real fix — even if it's two layers up.
- No fallbacks for impossible cases. No flags or scaffolding for hypothetical futures.
- Run `make` after changes to confirm the wasm build is green before handing off.
- Only `wasm32-unknown-unknown` is a supported target. Don't add `#[cfg(not(target_arch = "wasm32"))]` branches, native fallbacks, or dual-target stubs — if it wouldn't run in the browser, it doesn't need to compile.

---
> Source: [ShawnZhong/verus-explorer](https://github.com/ShawnZhong/verus-explorer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
