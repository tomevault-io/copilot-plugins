## analytics-engine-playground

> This project uses SvelteKit with Svelte 5 syntax. It also uses tailwindcss and typescript.

This project uses SvelteKit with Svelte 5 syntax. It also uses tailwindcss and typescript.

# Svelte 5 vs Svelte 4: Key Differences
- **$state replaces reactive let declarations**
  - Svelte 4: `let count = 0;` (implicitly reactive at top level)
  - Svelte 5: `let count = $state(0);` (explicitly reactive anywhere)
- **$derived and $effect replace $: statements**
  - Svelte 4: `$: double = count * 2;` and `$: { console.log(count); }`
  - Svelte 5: `let double = $derived(count * 2);` and `$effect(() => { console.log(count); })`
- **$props replaces export let**
  - Svelte 4: `export let name = 'world';`
  - Svelte 5: `let { name = 'world' } = $props();`
- Components are no longer classes in Svelte 5
- `<svelte:component>` is no longer necessary (use dynamic elements directly)
- Snippets replace slots:
  ```svelte
  <!-- Svelte 4 -->
  <slot name="header">Fallback content</slot>
  <!-- Svelte 5 -->
  {@render header()}
  ```
- Event attributes replace event directives:
  - Svelte 4: `<button on:click={handler}>Click me</button>`
  - Svelte 5: `<button onclick={handler}>Click me</button>` (though on:click still works)
- Whitespace handling is more strict
- CSS scoping uses `:where(...)` for better specificity control
- Stricter HTML structure validation
- Hydration works differently (uses comments for more robust hydration)
- `null` and `undefined` become empty strings in templates
- Modern browser required (no IE11 support)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brachkow) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
