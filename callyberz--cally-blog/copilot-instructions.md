## cally-blog

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a personal blog and portfolio website built with Next.js 13.5.6 (App Router), TypeScript, and Tailwind CSS. The blog features technical articles written in MDX format and managed by Contentlayer.

## Common Commands

```bash
# Development
pnpm install          # Install dependencies
pnpm dev             # Start development server at http://localhost:3000
pnpm build           # Build for production
pnpm start           # Start production server
pnpm lint            # Run ESLint

# Content Management
# Blog posts are MDX files in /posts/ directory
# After adding/editing posts, restart dev server to regenerate Contentlayer data
```

## Architecture

### Content Management System
- **Contentlayer**: Processes MDX files from `/posts/` directory
- **Post Schema**: Requires `title`, `date`, and `description` frontmatter
- **URLs**: Generated as `/posts/[slug]` where slug is the filename
- Configuration in `contentlayer.config.ts`

### Routing Structure (App Router)
- `/src/app/page.tsx` - Homepage with blog posts and personal intro
- `/src/app/posts/[slug]/page.tsx` - Dynamic blog post pages
- `/src/app/about/`, `/education/`, `/skills/`, `/work/` - Portfolio sections

### Component Architecture
- **UI Components**: shadcn/ui components in `/src/components/ui/`
- **Common Components**: Typography components in `/src/components/common/`
- **Core Components**: Container, Navigation, PostCard, ThemeSwitch
- **Data**: Personal information centralized in `/src/lib/data.ts`

### Styling System
- **Tailwind CSS**: Utility-first CSS with custom theme configuration
- **Dark/Light Theme**: Implemented with `next-themes`
- **Typography**: Uses `@tailwindcss/typography` for blog content
- **Animations**: Framer Motion for smooth transitions

## Key Technologies

- **Next.js 13.5.6** with App Router
- **TypeScript 5.8.3** for type safety
- **Contentlayer 0.3.4** for MDX processing
- **Tailwind CSS 3.4.17** with shadcn/ui components
- **Motion (Framer Motion) 12.17.3** for animations

## Development Notes

### Adding Blog Posts
1. Create new `.mdx` file in `/posts/` directory
2. Include required frontmatter: `title`, `date`, `description`
3. Restart development server for Contentlayer to process changes

### Component Development
- Follow existing patterns in `/src/components/`
- Use shadcn/ui components from `/src/components/ui/`
- Maintain consistent styling with Tailwind classes
- Support both light and dark themes

### Content Structure
- Personal data (work, education, skills, projects) in `/src/lib/data.ts`
- Static assets in `/public/images/`
- Typography components provide consistent styling across pages

### No Testing Framework
Currently no test configuration exists in the project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/callyberz)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/callyberz)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
