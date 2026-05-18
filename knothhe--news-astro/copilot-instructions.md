## news-astro

> This file provides guidelines for AI agents working in this Astro blog theme repository.

# AGENTS.md

This file provides guidelines for AI agents working in this Astro blog theme repository.

## Project Overview

Astro-based typography theme for blogs. Uses pnpm, TypeScript, ESLint (@-config), UnoCSS, and follows Astro bestantfu/eslint practices.

## Build Commands

| Command          | Description                             |
| ---------------- | --------------------------------------- |
| `pnpm dev`       | Start dev server with type checking     |
| `pnpm build`     | Build for production with type checking |
| `pnpm preview`   | Preview production build                |
| `pnpm lint`      | Run ESLint                              |
| `pnpm lint:fix`  | Run ESLint with auto-fix                |
| `pnpm typecheck` | Run TypeScript type checking only       |

## Code Style Guidelines

### TypeScript

- Use TypeScript for all `.ts` files and Astro frontmatter
- Enable `strict: true` in tsconfig (extends `astro/tsconfigs/strict`)
- Use type imports: `import type { Post } from '~/types'`
- Define types in `src/types/` directory
- Use path alias `~/*` for imports from `src/`

### ESLint

- Configuration extends `@antfu/eslint-config`
- Uses `eslint-plugin-astro` for `.astro` files
- Uses `eslint-plugin-format` for formatting
- Pre-commit hook runs `eslint --fix` via lint-staged

### Imports

```typescript
// Type imports first
import type { Post } from '~/types'

// Then external imports
import { getCollection } from 'astro:content'
import dayjs from 'dayjs'

// Then internal aliases
import { themeConfig } from '~/.config'
import { getPosts } from '~/utils'
```

### Naming Conventions

- **Variables/functions**: camelCase (`getPosts`, `formatDate`)
- **Constants**: UPPER_SNAKE_CASE for config values, PascalCase for objects
- **Types/Interfaces**: PascalCase (`Post`, `ThemeConfig`)
- **Files**: kebab-case for components (`site-footer.astro`), PascalCase for utilities (`Utils.ts`)
- **Component props**: camelCase in Astro templates

### Astro Components

- Frontmatter at top with `---` delimiter
- Imports ordered: theme config → components → styles
- Use `class:list` for conditional classes
- Place `<slot />` where children go
- Put scripts at bottom of `<body>`
- Use `transition-swup-*` classes for swup animations

### Error Handling

- Use early returns for validation checks
- Return fallback values rather than throwing in utilities
- Use `import.meta.env.PROD` for environment-specific behavior

### Styling

- UnoCSS utility classes in `class` attributes
- Use `lg:` prefix for responsive desktop styles
- Prefer utility classes over custom CSS when possible
- Custom CSS in `<style>` blocks for complex animations

### Functions

- Use async/await for data fetching
- Explicit return types for exported functions
- Default parameters for optional arguments
- Helper functions at module bottom (not exported)

## Important Paths

| Path              | Description            |
| ----------------- | ---------------------- |
| `src/pages/`      | Astro pages and routes |
| `src/layouts/`    | Page layouts           |
| `src/components/` | Reusable components    |
| `src/utils/`      | Utility functions      |
| `src/types/`      | TypeScript types       |
| `src/styles/`     | Global CSS             |
| `src/.config/`    | Theme configuration    |

## Theme Configuration

Theme settings are in `src/.config/index.ts`. Key sections:

- `site`: Website metadata
- `appearance`: Theme, locale settings
- `features`: Optional features (RSS, etc.)

## Special Features

- i18n support via `src/i18n.ts` with multiple locales
- RSS feed generation at `/atom.xml`
- Math rendering with KaTeX
- Swup page transitions

## Running Single Tests

No test framework configured. Run lint/typecheck to verify code:

```bash
pnpm lint && pnpm typecheck
```

## Daily News Report Skill

A skill exists in `.agents/skills/daily-news-report/` for generating daily tech news reports. It's a standalone skill and not related to the main Astro theme functionality.

## Posts Content

Markdown files are stored in `src/content/posts/` directory.

### File Naming Convention

- Format: `YYYY-MM-DD-news-report.md`
- Example: `2026-01-22-news-report.md`

### Frontmatter Metadata

All posts must include the following frontmatter fields:

```yaml
---
title: Daily News Report（YYYY-MM-DD）
pubDate: YYYY-MM-DD
description: Brief description of the report
categories: [daily-news, tech, AI, development]
draft: false
---
```

| Field         | Required | Description                                 |
| ------------- | -------- | ------------------------------------------- |
| `title`       | Yes      | Post title with date format                 |
| `pubDate`     | Yes      | Publication date in YYYY-MM-DD format       |
| `description` | Yes      | Brief description in quotes                 |
| `categories`  | Yes      | Array of category tags                      |
| `draft`       | Yes      | Boolean, set to `false` for published posts |

### Daily News Report Content Structure

Articles generated by the daily news skill follow this structure:

```markdown
> 本日筛选自 X 个信息源，共收录 Y 条高质量内容
> 生成耗时: ~Z 分钟 | 版本: v3.0
>
> **Sources**: source1, source2, source3, ...

---

## 1. News Title

**摘要**: Brief summary...

**要点**:

1. Key point 1
2. Key point 2
3. Key point 3

**来源**: [Link Text](URL)

**关键词**: `keyword1` `keyword2` `keyword3`

**评分**: ⭐⭐⭐⭐⭐ (5/5)

---
```

### News Entry Format

- Use `##` for main title
- Separate entries with `---`
- Each entry must have: summary, key points, source, keywords, rating
- Use emoji stars for rating (1-5)
- Include source URL in markdown link format

---
> Source: [knothhe/news-astro](https://github.com/knothhe/news-astro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
