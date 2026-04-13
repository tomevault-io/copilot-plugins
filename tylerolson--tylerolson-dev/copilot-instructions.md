## tylerolson-dev

> Personal portfolio website built with SvelteKit 5, displaying GitHub repository stats. The site fetches and caches data from the GitHub API, presenting deployed projects and all public repositories.

# Agent Guidelines for tylerolson.dev

## Project Overview

Personal portfolio website built with SvelteKit 5, displaying GitHub repository stats. The site fetches and caches data from the GitHub API, presenting deployed projects and all public repositories.

## Tech Stack

- **Framework**: SvelteKit 2 with Svelte 5 (runes syntax)
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS 4 with Catppuccin Mocha theme
- **Formatting**: Prettier with svelte and tailwindcss plugins
- **Package Manager**: pnpm

## Commands

```bash
# Development
pnpm dev          # Start dev server
pnpm build        # Production build
pnpm preview      # Preview production build

# Type checking & linting
pnpm check        # Run svelte-check (types + lint)
pnpm check:watch  # Watch mode for type checking

# Formatting
pnpm format       # Format all files (writes changes)
pnpm lint         # Check formatting (read-only)

# Sync (run before check/build)
pnpm prepare      # Sync SvelteKit types
```

**Note**: No test framework is configured. To add tests, consider installing Vitest.

## Code Style

### Formatting

- **Indentation**: Tabs
- **Quotes**: Single quotes
- **Trailing commas**: None
- **Print width**: 100 characters
- **Config**: `.prettierrc` for all settings

### TypeScript

- **Strict mode enabled** - no implicit any, strict null checks
- Use `type` instead of `interface` for type aliases (unless extending needed)
- Always import types explicitly: `import type { Foo } from '...'` or `import { type Foo } from '...'`
- Use `$types` for SvelteKit-generated types: `import type { PageProps } from './$types'`

### Svelte 5 Runes

```typescript
// Props
let { prop1, prop2 }: { prop1: string; prop2: number } = $props();

// Reactive state
let count = $state(0);

// Derived values
let doubled = $derived(count * 2);

// Effects (when side effects needed)
$effect(() => {
	console.log(count);
});
```

### SvelteKit Conventions

- **Server files**: `+page.server.ts`, `+server.ts` in route directories
- **Client files**: `+page.svelte`, `+layout.svelte`
- **Shared types**: Define in `src/lib/types.ts`
- **Server utilities**: Place in `src/lib/server/` directory
- **Path alias**: Use `$lib` for imports from `src/lib/`

### File Organization

```
src/
├── lib/
│   ├── components/    # Reusable UI components
│   ├── server/        # Server-only utilities (GitHub API, etc.)
│   └── types.ts       # Shared TypeScript types
├── routes/            # SvelteKit routes
│   ├── +page.svelte   # Route page
│   ├── +page.server.ts # Server-side load function
│   └── +server.ts     # API endpoints
├── app.css            # Global styles, Tailwind config
└── hooks.server.ts    # Server hooks
```

### Import Order

1. Svelte built-ins (`svelte/animate`, `svelte/transition`)
2. Type imports
3. `$lib` imports
4. Relative imports

### HTML & Templates

- Use semantic HTML elements
- Include `rel="noopener noreferrer"` on external links
- Add `aria-label` for icon-only links
- Svelte 5 uses `{#snippet}` and `{@render}` instead of `<slot>`

### CSS & Styling

- Use Tailwind utility classes in component classes
- Use Catppuccin color variables: `text-ctp-*`, `bg-ctp-*`, `border-ctp-*`
- Animation classes defined in `app.css`: `fade-in-up`, `fade-in-up-1` through `fade-in-up-4`
- Custom fonts: `font-mono` (JetBrains Mono), `font-sans` (Outfit)

### Error Handling

- Wrap async operations in try/catch blocks
- Log errors with context: `console.error('[context] message:', err)`
- Return sensible defaults on error (empty arrays, fallback values)

### Naming Conventions

- **Components**: PascalCase (`BigCard.svelte`, `SectionHeader.svelte`)
- **Functions/variables**: camelCase (`fetchAllRepos`, `reposCache`)
- **Constants**: PascalCase with descriptive names (`CACHE_TTL`)
- **Types/interfaces**: PascalCase (`RepoData`, `RepoStats`)

## GitHub API Integration

The site uses Octokit for GitHub API calls with a 6-hour cache (configurable via `CACHE_TTL` in `src/lib/server/github.ts`). The `RepoData` type maps to GitHub's repository API response format.

## Adding New Features

1. Create component in `src/lib/components/`
2. Import and use in appropriate route
3. Run `pnpm check` to verify types
4. Run `pnpm format` before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tylerolson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
