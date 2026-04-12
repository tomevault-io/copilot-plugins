## largolabs-blog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `pnpm dev` - Start development server with Turbopack
- `pnpm build` - Build for production
- `pnpm start` - Start production server
- `pnpm lint` - Run ESLint

## Architecture Overview

This is a Next.js 15 personal blog/portfolio site using the App Router with MDX support for blog posts.

### Key Technologies
- **Next.js 15** with App Router
- **MDX** for blog posts with frontmatter metadata
- **Tailwind CSS** with custom design system
- **Radix UI** components with shadcn/ui
- **TypeScript** with strict configuration
- **next-themes** for dark/light mode
- **Recharts** for data visualization

### Project Structure
- `app/` - Next.js App Router pages
  - `blog/[slug]/` - Dynamic blog post routes
  - `blog/posts/` - MDX blog post files
- `components/` - Reusable React components
  - `ui/` - shadcn/ui component library
- `lib/utils.ts` - Utility functions (cn for class merging)
- `hooks/` - Custom React hooks

### Blog System
- Blog posts are stored as `.mdx` files in `app/blog/posts/`
- Posts include frontmatter: `title`, `publishedAt`, `summary`
- Dynamic routing handles `/blog/[slug]` pages
- `BlogPosts` component lists all posts sorted by date
- Blog utilities should be in `app/blog/utils.ts` (currently missing)

### Styling & UI
- Uses Tailwind CSS with CSS variables for theming
- Custom font configuration with Roboto and Roboto Mono
- Sidebar layout with collapsible navigation
- Theme provider supports system/light/dark modes
- Components follow shadcn/ui patterns

### Key Features
- Responsive sidebar navigation
- Theme toggle functionality
- PDF viewer component for resume
- Experience chart visualization
- Search functionality
- Breadcrumb navigation

## Important Notes

- MDX configuration in `next.config.mjs` enables `.md` and `.mdx` page extensions
- Path aliases use `@/*` pointing to root directory
- Strict TypeScript configuration with ES2017 target
- Uses `suppressHydrationWarning` for theme provider compatibility

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jlargs64)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jlargs64)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
