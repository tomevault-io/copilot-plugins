## luml

> This is a **Vue/Vite app only**. Follow these rules strictly:

# Frontend build rules

This is a **Vue/Vite app only**. Follow these rules strictly:

- **No TypeScript emit.** TypeScript is used for type-checking only (`vue-tsc --noEmit`). Vite handles transpilation.
- **No declaration generation.** Do not enable or run anything that produces `.d.ts` files.
- **No `.d.ts` generation.**
- **No `.vue.js` output.** Never produce compiled `.vue.js` files next to `.vue` sources.
- **Never run raw `tsc` emit.** Do not run `tsc` to emit output. Type-check only (`tsc --noEmit` / `vue-tsc --noEmit`); build via Vite (`vite build`).
- **Keep generated files confined to `dist/`.** No build artifacts anywhere else in the source tree.

---
> Source: [luml-ai/luml](https://github.com/luml-ai/luml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
