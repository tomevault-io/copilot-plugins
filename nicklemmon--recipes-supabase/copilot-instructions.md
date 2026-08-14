## recipes-supabase

> - Before considering a task complete, run QA scripts:

# Instructions for AI agents

- Before considering a task complete, run QA scripts:

```bash
npm run qa
```

This file provides guidance to Claude Code (claude.ai/code) when working with code in this
repository.

## Agent Communication Standards

When communicating with the user (chat responses, commit messages, PR descriptions, comments),
follow plain-language principles grounded in:

- **ISO 24495-1:2023**: information should be relevant, findable, understandable, and usable.
- **W3C Cognitive Accessibility Guidance**: clear words, literal language, short text, separate
  steps, short critical paths, and no reliance on memory. It explicitly considers ADHD, but is
  advisory rather than required for WCAG conformance.
- **US Plain Writing Act**: communication must be understandable on the first reading.
- **JAN ADHD guidance**: recommends written, structured, step-by-step instructions. It is
  accommodation guidance, not a standard.

## Development Commands

- `npm start` - Start development server with Vite
- `npm run build` - Build production bundle
- `npm run serve` - Preview production build locally
- `npm run type-check` - Run TypeScript type checking

## Architecture Overview

This is a React-based family recipes app using:

- **TanStack Router** for file-based routing with type-safe navigation
- **Supabase** for backend database and authentication
- **Tailwind CSS v4** for styling
- **Zod** for runtime type validation
- **Base UI Components** for accessible UI primitives

### Key Architecture Patterns

**Authentication Flow**:

- Root route loader fetches session state from Supabase
- `_private` route group handles auth guards and redirects
- Public routes allow unauthenticated access

**Data Layer**:

- API functions in `src/api/` handle Supabase queries
- All database responses validated with Valibot schemas
- Type definitions in `src/types/` define data models

**Route Structure**:

- Nested routes follow URL pattern: `/recipes/$category/$subcategory/$recipe`
- Route files use TanStack Router conventions (`_private`, `$param`)
- `routeTree.gen.ts` auto-generated, don't edit manually

**Component Organization**:

- Reusable UI components in `src/components/`
- Page-specific components co-located with routes
- Consistent component props via `src/types/props.ts`

### Coding preferences

- Global components in `src/components/` should have a short JSDoc comment above the component
  function describing its purpose (see `EmptyCell` for an example)
- Prefer `type` over `interface` in TypeScript files
- Run the `prettier` command against all files before finishing work
- Place test files in `__tests__` directories (e.g., `src/components/__tests__/`,
  `src/helpers/__tests__/`)
- Use `<Inline>` for horizontal wrapping layouts instead of raw `flex` divs; use `<Stack>` for
  vertical ones

### Environment Setup

Requires `.env` file with Supabase credentials:

```
VITE_SUPABASE_PROJECT_URL="<project-url>"
VITE_SUPABASE_CLIENT_KEY="<client-key>"
```

### Database Schema

Main tables: `recipes`, `categories`, `subcategories`

- Recipes have hierarchical category/subcategory relationships
- Recipe slugs are unique within category/subcategory scope
- Rating field (1-5) used for favorites functionality

---
> Source: [nicklemmon/recipes-supabase](https://github.com/nicklemmon/recipes-supabase) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
