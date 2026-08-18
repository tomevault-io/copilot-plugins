## beautiful-ui-vue

> When selecting, reusing, copying, or changing a showcase component, read

# Beautiful UI Vue component catalog

When selecting, reusing, copying, or changing a showcase component, read
`.agents/skills/use-beautiful-ui-vue/SKILL.md` first.

Treat `component-registry.json` as the source of truth for the component inventory.
Query it instead of guessing:

- `npm run ui -- count` returns the current component count.
- `npm run ui -- list` lists every component and source file.
- `npm run ui -- show <id>` explains one component and its usage.
- `npm run ui -- source <id>` prints its copyable Vue source.
- `npm run agent:generate` refreshes the hosted Agent guide, catalog, and source files.

When adding, renaming, or removing a component, update `component-registry.json`
in the same change. Never edit generated files under `public/agent` manually.
Run `npm run build` before reporting completion.

---
> Source: [tycoding/beautiful-ui-vue](https://github.com/tycoding/beautiful-ui-vue) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
