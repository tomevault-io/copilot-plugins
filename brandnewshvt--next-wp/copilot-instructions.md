## next-wp

> This file provides guidance to OpenCode when working with code in this repository.

# AGENTS.md

This file provides guidance to OpenCode when working with code in this repository.

## Skill Usage Requirements

**MANDATORY: Check and use relevant skills BEFORE any action or response.**

### Project Skills (Priority - check first)

| Skill | When to Use | Trigger Phrases |
|-------|-------------|-----------------|
| `frontend-design` | Building web components, pages, UI artifacts | "build a component", "create a page", "design this" |
| `copywriting` | Marketing copy for pages | "write copy for", "improve this copy", "headline help", "CTA" |
| `vercel-react-best-practices` | React/Next.js code review & optimization | "optimize", "performance", "review code", "best practices" |
| `web-design-guidelines` | UI/UX compliance review | "review my UI", "check accessibility", "audit design" |

### Superpowers Skills (General workflows)

| Skill | When to Use |
|-------|-------------|
| `superpowers:brainstorming` | ANY creative work (features, components, functionality) |
| `superpowers:test-driven-development` | Implementing features or bugfixes |
| `superpowers:systematic-debugging` | Bugs, test failures, unexpected behavior |
| `superpowers:writing-plans` | Multi-step tasks from specs/requirements |
| `superpowers:executing-plans` | Executing a written implementation plan |
| `superpowers:verification-before-completion` | Before claiming work is complete |
| `superpowers:requesting-code-review` | Completing tasks or major features |
| `superpowers:dispatching-parallel-agents` | 2+ independent tasks |
| `superpowers:subagent-driven-development` | Implementation plans with independent subtasks |

### Skill Priority Order
1. **Project skills first** - domain-specific guidance for this codebase
2. **Process skills second** (brainstorming, debugging) - determine HOW to approach
3. **Implementation skills third** (TDD, plans) - guide execution

### Red Flag Thoughts (STOP if you think these)
- "This is just a simple question" → Questions are tasks. Check for skills.
- "I need more context first" → Skill check comes BEFORE clarifying questions.
- "Let me explore the codebase first" → Skills tell you HOW to explore.
- "This doesn't need a formal skill" → If a skill exists, use it.
- "I know React best practices" → Use `vercel-react-best-practices` anyway.
- "The design looks fine" → Use `web-design-guidelines` to verify.

## Project Memory Management

**MANDATORY: Update project memory after completing each task/prompt.**

Use `memory_set` or `memory_replace` to update the `project:project` memory block with:
- Important discoveries about the codebase
- New conventions established during the task
- Gotchas and issues encountered
- Architectural decisions made
- Commands or patterns that were useful

This ensures context persists across sessions and helps future agents work more effectively.

## Build Commands
- `pnpm run dev` - Start development server with turbo mode
- `pnpm run build` - Build for production
- `pnpm run start` - Start production server  
- `pnpm run lint` - Run ESLint to check code quality

## Architecture Overview

This is a headless WordPress starter using Next.js 15 App Router with TypeScript. Key architectural patterns:

### Data Layer
- All WordPress API interactions go through `lib/wordpress.ts`
- Type definitions in `lib/wordpress.d.ts` define Post, Page, Category, Tag, Author, Media interfaces
- Error handling uses custom `WordPressAPIError` class
- Functions use Next.js cache tags for granular revalidation (e.g., `tags: ['posts', `post-${slug}`]`)

### Routing Structure
- Dynamic routes: `/posts/[slug]`, `/pages/[slug]`
- Archive pages: `/posts`, `/posts/authors`, `/posts/categories`, `/posts/tags`
- API routes: `/api/revalidate` (webhook), `/api/og` (OG images)

### Component Patterns
- Server Components for data fetching with parallel `Promise.all()` calls
- URL-based state management for search and filters
- Debounced search (300ms) with `useSearchParams`
- Pagination with 9 posts per page default

### Revalidation System
- WordPress plugin sends webhooks on content changes
- Next.js endpoint validates webhook secret and calls `revalidateTag()`
- Default cache duration: 1 hour (`revalidate: 3600`)

## Code Style

### TypeScript
- Use strict typing with interfaces defined in `lib/wordpress.d.ts`
- Prefer type annotations over type assertions
- Use type inference when the type is obvious

### Naming Conventions
- React components: PascalCase (e.g., `PostCard.tsx`)
- Functions and variables: camelCase
- Types and interfaces: PascalCase
- Constants: UPPERCASE_SNAKE_CASE for true constants

### File Structure
- Page components: `/app/**/*.tsx`
- Reusable UI components: `/components/**/*.tsx`  
- API and utility functions: `/lib/**/*.ts`
- WordPress data functions must use cache tags for proper revalidation

### Error Handling
- Use `try/catch` blocks for API calls
- Utilize `WordPressAPIError` class for consistent API error handling

## Environment Variables
Required environment variables (see `.env.example`):
- `WORDPRESS_URL` - Full URL of WordPress site
- `WORDPRESS_HOSTNAME` - Domain for image optimization
- `WORDPRESS_WEBHOOK_SECRET` - Secret for webhook validation

## Key Dependencies
- Next.js 16.1.6 with React 19.2.4
- TypeScript with strict mode
- Tailwind CSS with shadcn/ui components
- React Hook Form for form handling
- Lucide React for icons

## Frontend Style Guide

**MANDATORY: Read `prompt.xml` before any frontend work.**

The `prompt.xml` file contains the complete frontend design system specification:

### Design Systems
1. **Craft Design System** (`@/components/craft`) - Layout primitives & prose
   - `Section`, `Container`, `Box` for page structure
   - `Article`, `Prose` for WordPress content
   - Built-in typography handling

2. **shadcn/ui** (`@/components/ui/*`) - Interactive UI components
   - Button, Input, Badge, Form, Select, Sheet, etc.
   - Never recreate these - use existing components

### Design Style: Minimalist Monochrome
- **Colors**: Pure black (#000000) and white (#FFFFFF) only
- **Typography**: Serif fonts (Playfair Display, Source Serif 4)
- **Corners**: Zero border-radius everywhere (sharp 90° corners)
- **Shadows**: None - depth through color inversion and borders
- **Animations**: Instant (100ms max transitions)

### Quick Reference

| Need | Use |
|------|-----|
| Page layout | `Section`, `Container` from craft |
| Grid/flex | `Box` from craft |
| WordPress content | `Article` or `Prose` from craft |
| Buttons, forms | shadcn/ui components |
| Cards | Custom div with `border-2 border-foreground p-6` |

### Non-Negotiables
1. Oversized hero typography (8xl-9xl)
2. Heavy horizontal rules between sections
3. Zero border-radius on ALL elements
4. Color inversion for emphasis (no accent colors)
5. No shadows, ever

---
> Source: [BRANDNEWSHVT/next-wp](https://github.com/BRANDNEWSHVT/next-wp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-19 -->
