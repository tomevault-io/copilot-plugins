## sveltekit

> SvelteKit + TypeScript + Svelte 5 conventions


# SvelteKit

This is an **Apply to Specific Files** rule. It attaches when Svelte or
SvelteKit route/server files are in context.

- Use SvelteKit + TypeScript + Svelte 5 runes. Do not add another UI framework.
- Put server-only modules in `$lib/server/`. Client modules must not import
  `$lib/server/*`.
- Share types from `$lib`. Do not export the store from a client-safe module.
- REST handlers are `src/routes/api/**/+server.ts`. Use `json` and `error`
  from `@sveltejs/kit`.
- UI components live in `$lib/components/` with `$props()` and `on*` callbacks.
- Colocate scoped `<style>` in the same `.svelte` file.
- Named exports. Explicit return types on exported functions.
- Tests are `*.spec.ts` next to the module. Do not start `npm run dev`.

---
> Source: [damien-xai/cursor-rules-skills-mcp-hooks](https://github.com/damien-xai/cursor-rules-skills-mcp-hooks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-14 -->
