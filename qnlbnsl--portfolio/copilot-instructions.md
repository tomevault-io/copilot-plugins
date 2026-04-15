## portfolio

> You are working on a SvelteKit portfolio project using modern web technologies. Follow these rules strictly.

You are working on a SvelteKit portfolio project using modern web technologies. Follow these rules strictly.

## Technology Stack (Required Versions)

- Svelte 5.0+ with runes only
- Tailwind CSS v4.1.3+ only
- TypeScript 5.0+
- Vite 6.2.5+
- DaisyUI 5.0.17+
- Flowbite Svelte 0.48.6+
- SvelteKit 2.16.0+
- Use ES modules only

## Documentation Access

Before suggesting code, search the indexed documentation using MCP tools to verify correct syntax and patterns for the specific technology versions used in this project.

## Svelte 5 Rules (Critical)

Use Svelte 5 runes exclusively. Never suggest legacy patterns.

Component props:

```
let { title, description } = $props();
```

Reactive state:

```
let count = $state(0);
```

Computed values:

```
let doubled = $derived(count * 2);
```

Side effects:

```
$effect(() => { console.log(count); });
```

Event handlers:

```
<button onclick={handleClick}>
```

Never use:

- `export let` syntax
- `$:` reactive statements
- `onMount`/`onDestroy` for simple effects

## Tailwind CSS v4 Rules (Critical)

Use Tailwind v4 syntax only. Never suggest v3 patterns.

CSS imports:

```
@import 'tailwindcss';
@plugin "@tailwindcss/typography";
```

CSS variables for theming:

```
--color-primary: 241 73% 67%;
--p: 241 73% 67%;
```

Reference CSS variables:

```
bg-[hsl(var(--color-primary))]
```

Never use:

- `@apply` directive
- `@variants` or `@responsive`
- Hardcoded color classes like `bg-red-500`

## TypeScript Rules

Use strict TypeScript with modern features.

Component props with types:

```
interface CardProps { title: string; }
let { title } = $props<CardProps>();
```

Union types for variants:

```
type Size = 'sm' | 'md' | 'lg';
```

## Component Architecture

File organization:

- Components in `src/lib/components/`
- Utilities in `src/lib/utils/`
- MDSvex content uses `.svx` extension

Component patterns:

- Use descriptive names: `CardLarge.svelte`
- TypeScript interfaces for props
- Proper accessibility attributes
- Export from index files

## Content and Documentation

MDSvex files require frontmatter:

```
---
title: 'Title'
description: 'Description'
categories: ['category']
publishedDate: '2023-08-15'
---
```

Add JSDoc comments for complex logic and props.

## Development Workflow

Use these commands:

- `npm run dev` - development server
- `npm run build` - production build
- `npm run check` - type checking
- `yarn` for package management

## Error Prevention

Never suggest:

- Svelte 4 syntax
- Tailwind v3 patterns
- CommonJS require() statements
- Deprecated package versions

Always verify suggestions match the technology stack versions before providing code.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/qnlbnsl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
