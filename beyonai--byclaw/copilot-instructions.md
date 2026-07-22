## byclaw

> The default **main** agent text is authored in [`templates/main-agents.md`](templates/main-agents.md) and **inlined into `dist/index.js` at build time** (`esbuild --loader:.md=text`). Runtime only needs the single bundle file; no `templates/` directory beside `dist/`.

# Main workspace instructions (canonical template)

The default **main** agent text is authored in [`templates/main-agents.md`](templates/main-agents.md) and **inlined into `dist/index.js` at build time** (`esbuild --loader:.md=text`). Runtime only needs the single bundle file; no `templates/` directory beside `dist/`.

Edit `templates/main-agents.md` and run `npm run build` to refresh the embedded copy. Use plugin config `mainAgentsMdPath` to point at an external file instead.

---
> Source: [beyonai/ByClaw](https://github.com/beyonai/ByClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-22 -->
