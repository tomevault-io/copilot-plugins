## arboles-enlaces

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project state

Freshly scaffolded, minimal SvelteKit app (`sv create --template minimal --types ts`). Only the default welcome page exists so far (`src/routes/+page.svelte`); there is no application logic, tests, or lint config yet. When adding features, follow the SvelteKit conventions below rather than assuming an existing structure.

## Commands

- `npm run dev` — start the Vite dev server (append `-- --open` to open a browser)
- `npm run build` — production build (output depends on the auto-selected adapter)
- `npm run preview` — serve the production build locally
- `npm run check` — type-check `.svelte`/`.ts` via `svelte-check`; run this as the "does it compile" gate (there is no separate lint/test runner). `npm run check:watch` for watch mode.
- `npm run prepare` — runs `svelte-kit sync` to regenerate `.svelte-kit/` (types, tsconfig). Run this after changing `svelte.config`/routes if type errors reference stale generated files.

`.npmrc` sets `engine-strict=true`, so Node/npm engine mismatches will fail installs.

## Architecture & conventions

- **Svelte 5 runes mode is forced** for all first-party code (see `vite.config.ts`; `node_modules` is exempted). Use runes (`$props`, `$state`, `$derived`, `$effect`) — legacy `export let` / reactive `$:` syntax will not work in project files.
- **Tailwind v4** is wired through the `@tailwindcss/vite` plugin, not a `tailwind.config.js`. Styles are activated by `@import 'tailwindcss';` in `src/routes/layout.css`, which `+layout.svelte` imports. There is no PostCSS config; configure Tailwind via CSS (`@theme`, etc.) rather than a JS config.
- **Routing** is file-based under `src/routes/` (`+page.svelte`, `+layout.svelte`, `+server.ts`, `+page.ts`/`+page.server.ts`, etc. per SvelteKit).
- **Shared code** goes in `src/lib/` and is imported via the `$lib` alias (e.g. `import x from '$lib/...'`).
- **Adapter**: uses `@sveltejs/adapter-auto`, which picks a deployment target at build time. `.gitignore` hints at Vercel/Netlify/Cloudflare targets; switch to a specific adapter in `svelte.config`/`vite.config` if you settle on one.
- **`src/app.html`** is the page shell (`%sveltekit.head%` / `%sveltekit.body%`); `data-sveltekit-preload-data="hover"` enables link preloading on hover.
- TypeScript is `strict` with `checkJs` on, so `.js` files are also type-checked.

---
> Source: [marianacortes0/arboles-enlaces](https://github.com/marianacortes0/arboles-enlaces) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-08 -->
