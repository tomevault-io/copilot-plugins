## svelte-wxt-standards

> Svelte 5 + WXT + Tailwind v4 coding standards; runes-only, no comments, readability first


# Svelte WXT standards

## Svelte / WXT / stack

- Use PascalCase for component file names (e.g. MyComponent.svelte).
- Use semantic HTML where possible.
- Use Svelte stores for global state; TypeScript for type safety.
- Svelte 5 runes only: never use legacy reactive statements ($:) or reactive `let`.
- Use $state() for reactive local state, $derived() for computed values, $effect() for side effects, $props() for component props.
- No comments; rely on clear naming and structure.
- Use Tailwind v4 classes instead of custom styles.

## General

- Follow the user's requirements carefully and fully.
- Prefer readable, correct, and complete code; avoid todos and placeholders.
- Reference file names; be concise; follow project structure and formatting.
- If the answer is uncertain or unknown, say so instead of guessing.
- When asked, produce or update architecture diagrams from project structure and requirements.

---
> Source: [sm17p/amfig](https://github.com/sm17p/amfig) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
