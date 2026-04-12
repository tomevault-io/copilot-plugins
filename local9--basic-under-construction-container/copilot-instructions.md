## basic-under-construction-container

> description: Core frontend guidance for Antony Cook’s personal gallery (SvelteKit 5, Svelte runes, Tailwind CSS, shadcn-svelte, Lucide Svelte).

---
description: Core frontend guidance for Antony Cook’s personal gallery (SvelteKit 5, Svelte runes, Tailwind CSS, shadcn-svelte, Lucide Svelte).
alwaysApply: true
---

# Frontend stack and mindset

- You are a **frontend expert** in the latest SvelteKit 5, Svelte 5 runes, Tailwind CSS, and shadcn-svelte. Assume modern browser support.
- Prefer **Svelte 5 runes** over legacy patterns:
  - Use `$state` for local state, `$derived` for computed values, and `$effect` for side-effects instead of `$:` and `onMount` where practical.
  - Use `$props<T>()` in Svelte components to declare and type props in TypeScript.
- Favor **small, focused components** in `$lib/components` and `$lib/server` over large pages.

# Svelte and SvelteKit patterns

- Use **file-based routing** with `+page.svelte`, `+page.server.ts`, `+layout.svelte`, and `+layout.server.ts` as per SvelteKit docs.
- Prefer **typed load functions** and `PageServerLoad` when fetching data, keeping data access in `+page.server.ts` and thin UI in Svelte components.
- Avoid legacy APIs or patterns that predate Svelte 5 unless there is a compelling reason.

# Tailwind, shadcn-svelte, and design system

- Use **Tailwind CSS for layout and spacing**, reserving custom CSS for gallery-specific behavior and animations defined in `@layer components` in `src/app.css`.
- When using shadcn-svelte:
  - Treat shadcn components as the base UI kit, customizing via Tailwind utility classes and design tokens.
  - Keep components colocated under `$lib/components` following the `components.json` aliases.

# Icons (Lucide Svelte)

- Use **Lucide Svelte** icons from `@lucide/svelte`:
  - Import icons as PascalCase components, e.g. `import { Camera, Instagram, Facebook } from '@lucide/svelte';`.
  - Pass standard Lucide props like `size`, `strokeWidth`, and `class`, combining with Tailwind classes for sizing and color.
- Encapsulate repeated icon/link combinations in reusable components (for example, `IconLink.svelte`) that accept a Lucide component via `$props()`.


---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Local9)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Local9)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
