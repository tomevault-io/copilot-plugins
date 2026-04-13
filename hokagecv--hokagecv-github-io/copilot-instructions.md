## hokagecv-github-io

> Personal website and blog built with Astro. Contains blog posts, projects, code snippets, and dynamic OG image generation.

# AGENTS.md - hokageCV.github.io

Personal website and blog built with Astro. Contains blog posts, projects, code snippets, and dynamic OG image generation.

## Project Overview

- Framework: Astro 5.x
- Package Manager: pnpm (v10.4.1)
- Language: TypeScript (strict mode)
- Styling: CSS with custom properties (Utopia fluid typography, CSS variables)
- Content: Astro Content Collections (blogs, projects, snippets)

## Build Commands

```bash
pnpm dev          # Start development server (default on folder open)
pnpm start        # Alias for pnpm dev
pnpm build        # Build for production (outputs to dist/)
pnpm preview      # Preview production build locally
pnpm astro        # Run Astro CLI directly
```

### Running Single Test

No test framework is configured - there are no tests in this project.

## Project Structure

```
src/
├── components/       # Astro components (PascalCase, grouped by feature)
│   ├── blogs/        # Blog-related components
│   ├── projects/     # Project-related components
│   ├── snippets/     # Snippet-related components
│   ├── tags/         # Tag components
│   ├── theme/        # Theme switching components
│   ├── contact/      # Contact form components
│   ├── header/       # Header components
│   └── *.astro       # Shared components
├── content/          # Astro Content Collections
│   ├── blogs/        # Blog posts (Markdown, organized by year)
│   ├── projects/     # Project entries (Markdown)
│   └── snippets/     # Code snippet entries (Markdown)
├── data/             # Static data files (JS/TS)
├── layouts/          # Astro layouts
├── pages/            # Astro pages (file-based routing)
│   ├── blogs/[...slug].astro
│   ├── projects/[...slug].astro
│   ├── snippets/[...slug].astro
│   ├── tags/[tag].astro
│   └── rss.xml.ts
├── styles/           # Global CSS files
├── utils/            # Utility functions (camelCase TS files)
└── types.ts          # Shared TypeScript types

public/
├── fonts/            # Font files for OG image generation
└── ...               # Static assets

astro.config.mjs     # Astro configuration
tsconfig.json        # TypeScript configuration
```

## Code Style Guidelines

### Imports

```typescript
// Use @/ alias for src/ imports
import BlogList from '@/components/blogs/BlogList.astro'
import { getBlogList } from '@/utils/collection'
import type { TagsType } from '@/types'

// Third-party imports
import { DateTime } from 'luxon'
import { getCollection } from 'astro:content'

// Node.js imports
import fs from "node:fs/promises";
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Astro Components | PascalCase | `BlogPost.astro`, `ProjectCard.astro` |
| TypeScript Files | camelCase | `date.ts`, `collection.ts`, `open-graph.ts` |
| TypeScript Types | PascalCase | `TagsType`, `BlogPost` |
| CSS Files | kebab-case | `global.css`, `scrollbar.css` |
| CSS Variables | kebab-case | `--space-l`, `--base-background` |
| Functions | camelCase | `getBlogList()`, `readableDate()` |
| Constants | camelCase | `MAX_NUMBER_OF_SIMILAR_POSTS` |
| Collections | camelCase | `blogList`, `projectList` |

### Astro Components

Frontmatter uses TypeScript:
```astro
---
import Base from '@/layouts/Base.astro'
import { readableDate } from '@/utils/date'

const { data, render } = Astro.props
const { Content } = await render()
---

<div>
  <h1>{data.title}</h1>
  <Content />
</div>

<style>
  /* Component-scoped styles */
  div {
    display: flex;
  }
</style>
```

### TypeScript

- Strict mode enabled (`strict: true`, `strictNullChecks: true`)
- Use explicit types for function parameters and return values
- Use `type` imports for type-only imports (`import type { ... }`)
- **Verbatim module syntax**: Use `import type` for types only

### CSS Styling

- Use CSS custom properties from `src/styles/global.css`
- Fluid typography via Utopia scale (`--step--5` to `--step-5`)
- Spacing scale via `--space-2xs` to `--space-8xl`
- Theme switching via `html[data-theme='dark']` / `html[data-theme='light']`
- Use `:root` for CSS variable definitions

### Error Handling

- Use try-catch with logging for async operations
- Use Astro's `logger.warn()` / `logger.error()` in integration hooks
- Handle missing files gracefully with informative error messages

### Content Collections

Define content in `src/content/` with frontmatter matching schemas:

Blog frontmatter:
```markdown
---
title: "Post Title"
description: "Post description"
publishedDate: "2024-01-15"
draft: false
tags: ["javascript", "backend"]
---
```

Project frontmatter:
```markdown
---
title: "Project Name"
repoLink: "https://github.com/..."
liveLink: "https://..."
cover:
  image: "/path/to/image.png"
  alt: "Description"
---
```

Snippet frontmatter:
```markdown
---
title: "Snippet Title"
publishedDate: "2024-01-15"
tags: ["ruby", "rails"]
---
```

## Architecture Notes

### Path Aliases

Configured in `tsconfig.json`:
- `@/*` → `src/*`
- `~/assets/*` → `src/assets/*`

### Astro Configuration

Key settings in `astro.config.mjs`:
- Site URL: `https://hokageCV.github.io`
- Trailing slash: `never`
- Integrations: MDX, Partytown, Sitemap, Expressive Code
- Expressive Code theme: `solarized-dark`

### Dynamic Routing

- `src/pages/blogs/[...slug].astro` - Individual blog posts
- `src/pages/projects/[...slug].astro` - Individual projects
- `src/pages/snippets/[...slug].astro` - Individual snippets
- `src/pages/tags/[tag].astro` - Tag-filtered pages

### OG Image Generation

Custom Astro integration (`src/utils/open-graph.ts`) generates OpenGraph images for blog posts during build. Font file required at `public/fonts/JetBrainsMono-Regular.ttf`.

## Git Workflow

- Branch: `main` (triggers deploy to GitHub Pages)
- CI/CD: GitHub Actions workflow in `.github/workflows/deploy.yaml`
- Dist and node_modules are gitignored

## Common Tasks

### Add a new blog post
1. Create `src/content/blogs/YYYY/slug.md`
2. Add frontmatter with title, description, publishedDate, tags
3. Write content in Markdown/MDX

### Add a new project
1. Create `src/content/projects/slug.md`
2. Add frontmatter with title, cover, optional links
3. Update `src/data/projects.ts` with project order

### Modify theme/styles
1. Edit `src/styles/global.css` for global styles and CSS variables
2. Component styles go in `<style>` blocks within `.astro` files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hokageCV) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
