## shadcn-studio-svelte

> - **Reactivity:** Use Runes! `$state`, `$derived`, `$effect`. NO `$:`.

# Repository Guide for Agents

## Svelte 5 Conventions

- **Reactivity:** Use Runes! `$state`, `$derived`, `$effect`. NO `$:`.
- **Props:** Use `let { prop } = $props();` instead of `export let prop`.
- **Events:** Use `onclick={handler}` instead of `on:click`.
- **Snippets:** Use `{#snippet}` and `{@render}` instead of `<slot>`.
- **Async:** Use `<svelte:boundary>` and `{#await}`.
- **Component Id:** Use `$props.id()` instead of `crypto.randomUUID()`.

## General Rules

- **Conciseness:** Be extremely concise.
- **Types:** Do NOT write explicit return types unless necessary.

## btca

Trigger: user says "use btca" (for codebase/docs questions).

Run:

- btca ask --resource <resource> --question "<question>"

Available <resource>: svelte, tailwindcss, runed, motion-svelte

---
> Source: [EpicAlbin03/shadcn-studio-svelte](https://github.com/EpicAlbin03/shadcn-studio-svelte) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
