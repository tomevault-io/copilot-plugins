## chiri-blog

> This document provides guidelines for AI coding agents working on the Chiri blog project.

# AGENTS.md - Chiri Blog Coding Guidelines

This document provides guidelines for AI coding agents working on the Chiri blog project.

## Project Overview

This is a minimalist blog theme built with **Astro 5.x**, using TypeScript, MDX, and deployed to Netlify. The project uses pnpm as the package manager.

## Build/Lint/Test Commands

```bash
# Development
pnpm dev                  # Start dev server

# Build
pnpm build                # Run prebuild script + astro build

# Linting & Formatting
pnpm lint                 # Run ESLint
pnpm lint:fix             # Run ESLint with auto-fix
pnpm format               # Format with Prettier
pnpm format:check         # Check formatting without changes

# Type Checking
tsc --noEmit              # Run TypeScript type check

# Utilities
pnpm new                  # Create new blog post
pnpm update-theme         # Update theme from upstream
```

**Note:** There are no test files in this project. CI runs: format:check, lint, tsc --noEmit, and build.

## Directory Structure

```
src/
├── components/           # Astro components
│   ├── examples/         # Example components (Callout, Tag, etc.)
│   ├── layout/           # Layout components (Header, Footer, BaseHead)
│   ├── ui/               # UI components (LinkCard, TOC, ThemeToggle)
│   └── widgets/          # Widget components (PostList, FormattedDate)
├── content/              # Astro Content Collections
│   ├── about/            # About page (Markdown)
│   └── posts/            # Blog posts (Markdown/MDX)
├── layouts/              # Page layouts (BaseLayout, IndexLayout, PostLayout)
├── pages/                # Routes and API endpoints
├── plugins/              # Custom remark/rehype plugins (.mjs)
├── styles/               # CSS stylesheets
├── types/                # TypeScript type definitions
├── utils/                # Utility functions
├── config.ts             # Theme configuration
└── content.config.ts     # Content collection schemas
```

## Code Style Guidelines

### Formatting (Prettier)

- **No semicolons** (`semi: false`)
- **Single quotes** (`singleQuote: true`)
- **No trailing commas** (`trailingComma: "none"`)
- **Print width:** 120 characters
- **Tab width:** 2 spaces
- **Plugin:** `prettier-plugin-astro` for `.astro` files

### TypeScript

- Use `strictNullChecks: true`
- Extends `astro/tsconfigs/strict`
- Path alias: `@/` maps to `src/`

```typescript
// Good
import { themeConfig } from '@/config'
import type { ThemeConfig } from '@/types'

// Bad
import { themeConfig } from '../../config'
```

### Imports

1. External packages first
2. Then internal imports using `@/` alias
3. Types should use `import type` syntax

```typescript
import { defineConfig } from 'astro/config'
import mdx from '@astrojs/mdx'

import { themeConfig } from '@/config'
import type { LayoutProps } from '@/types'
```

### Naming Conventions

- **Files:** kebab-case for utilities (`date.ts`, `image-config.ts`)
- **Components:** PascalCase (`PostList.astro`, `LinkCard.astro`)
- **Plugins:** kebab-case with prefix (`remark-toc.mjs`, `rehype-copy-code.mjs`)
- **Types:** PascalCase with descriptive names (`ThemeConfig`, `LayoutProps`)
- **Functions:** camelCase (`formatDate`, `getSortedFilteredPosts`)
- **Constants:** SCREAMING_SNAKE_CASE for true constants (`MONTHS_EN`, `VALID_SEPARATORS`)

### Types

- Define types in `src/types/` directory with barrel export from `index.ts`
- Use interfaces for object shapes, type aliases for unions/primitives
- Export types from categorized files (e.g., `config.types.ts`, `content.types.ts`)

```typescript
// src/types/config.types.ts
export interface SiteInfo {
  website: string
  title: string
  author: string
  description: string
  language: string
}

export type DateFormat = 'YYYY-MM-DD' | 'MM-DD-YYYY' | 'DD-MM-YYYY' | 'MONTH DAY YYYY' | 'DAY MONTH YYYY'
```

### Astro Components

- Use frontmatter (`---`) section for imports, props, and logic
- Define `Props` type using `type Props = ...`
- Access props via `Astro.props`
- Use `<style>` for scoped styles, `<style is:global>` for global styles
- Use `<style is:inline>` for styles that need to be inlined

```astro
---
import { themeConfig } from '@/config'
import type { LayoutProps } from '@/types'

type Props = LayoutProps

const { type = 'page' } = Astro.props
---

<div data-type={type}>
  <slot />
</div>

<style>
  div {
    padding: 1rem;
  }
</style>
```

### Error Handling

- Use try-catch for async operations
- Silently handle non-critical errors in UI components
- Use optional chaining (`?.`) and nullish coalescing (`??`) operators
- Avoid `console.log`, use `console.warn` or `console.error` if needed (ESLint warns on console)

```typescript
try {
  const response = await fetch(url)
  if (!response.ok) {
    throw new Error('Failed to fetch')
  }
} catch {
  // Handle error silently or return fallback
  return null
}
```

### ESLint Rules

- `no-console`: warn (allow `warn` and `error`)
- `@typescript-eslint/no-unused-vars`: warn (ignore args prefixed with `_`)
- `@typescript-eslint/no-explicit-any`: warn
- `astro/no-set-html-directive`: off

### Remark/Rehype Plugins

- Use `.mjs` extension for plugins
- Export default function that returns the transform function
- Use `unist-util-visit` for AST traversal
- Store computed data in `file.data.astro.frontmatter`

```javascript
import { visit } from 'unist-util-visit'

export default function remarkPlugin() {
  return function (tree, file) {
    visit(tree, 'node-type', (node) => {
      // Transform logic
    })

    if (!file.data.astro) file.data.astro = {}
    if (!file.data.astro.frontmatter) file.data.astro.frontmatter = {}
    file.data.astro.frontmatter.customData = computedValue
  }
}
```

### Content Collections

- Posts go in `src/content/posts/` as `.md` or `.mdx` files
- Required frontmatter: `title`, `pubDate`
- Optional frontmatter: `image`

```yaml
---
title: My Post Title
pubDate: 2024-01-01
image: /images/cover.jpg # optional
---
```

### CSS Variables

Use CSS variables defined in the theme for consistent styling:

- Colors: `--text-primary`, `--text-secondary`, `--text-tertiary`, `--border`, `--selection`
- Typography: `--font-size-s`, `--font-size-m`, `--font-weight-bold`
- Code: `--astro-code-background`

## Configuration

Main theme configuration is in `src/config.ts`. Modify this file to customize:

- Site info (title, author, description, language)
- General settings (content width, layout, animations)
- Date format settings
- Post settings (TOC, reading time, link cards)

## Key Dependencies

- **Astro 5.x** - Static site generator
- **@astrojs/mdx** - MDX support
- **rehype-katex/remark-math** - Math equation support
- **sharp** - Image processing
- **feed** - RSS/Atom feed generation

## CI/CD

GitHub Actions runs on push/PR to main:

1. Format check (`pnpm format:check`)
2. Lint (`pnpm lint`)
3. Type check (`tsc --noEmit`)
4. Build (`pnpm build`)

Auto-merge is enabled for Dependabot PRs and PRs with `automerge` label.

---
> Source: [fishcat37/chiri-blog](https://github.com/fishcat37/chiri-blog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
