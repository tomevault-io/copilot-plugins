## module-entrypoints

> Prefer named module entry files over index files


# Module Entrypoints

- Avoid creating new `index.ts` or `index.js` files for module entrypoints.
- Prefer a named entry file that matches the module or folder name, such as `llm.ts`, `cacheManager.ts`, or `componentName.js`.
- When adding or refactoring shared modules, keep exports in the named entry file instead of introducing an `index.*` barrel.
- If an existing area already uses a named entry file convention, extend that convention rather than adding an `index.*` alongside it.

---
> Source: [grebmann1/workbench](https://github.com/grebmann1/workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
