## dev-contract-shields-frontend

> <!-- BEGIN:nextjs-agent-rules -->

<!-- BEGIN:nextjs-agent-rules -->
# This is NOT the Next.js you know

This version has breaking changes — APIs, conventions, and file structure may all differ from your training data. Read the relevant guide in `node_modules/next/dist/docs/` before writing any code. Heed deprecation notices.
<!-- END:nextjs-agent-rules -->

# DevContractShield Frontend Agent Guidelines

## Project Overview

- **Framework**: Next.js 16.2.1 (App Router)
- **UI Library**: React 19
- **Language**: TypeScript 5
- **Styling**: Tailwind CSS 4
- **Linting**: ESLint 9 with `eslint-config-next`
- **Package Manager**: npm

## Build Commands

```bash
# Development
npm run dev              # Start dev server at http://localhost:3000

# Production
npm run build            # Build for production
npm run start            # Start production server

# Linting
npm run lint             # Run ESLint on all files

# Type checking
npx tsc --noEmit         # Run TypeScript compiler without emitting
```

## Code Style Guidelines

### TypeScript

- Use **strict mode** (enabled in tsconfig.json)
- Always use explicit types for function parameters and return values
- Prefer `interface` over `type` for object shapes
- Use `Readonly<T>` for immutable types
- Avoid `any` - use `unknown` when type is truly unknown

### Imports

- Use path aliases: `@/*` maps to project root
- Order imports: external packages → internal modules → CSS/assets
- Always import types with `import type { TypeName }` for type-only imports
- Use named exports over default exports where possible

### Naming Conventions

| Element | Convention | Example |
|---------|------------|---------|
| Files | kebab-case | `user-profile.tsx` |
| Components | PascalCase | `UserProfile` |
| Functions | camelCase | `fetchUserData` |
| Constants | SCREAMING_SNAKE | `MAX_RETRIES` |
| Types/Interfaces | PascalCase | `UserProfileProps` |
| CSS variables | kebab-case | `--color-primary` |

### React/Next.js Conventions

- Server Components by default; add `'use client'` directive only when needed
- Use `Metadata` export for page metadata
- Use `next/image` for all images
- Use `next/font` for Google Fonts
- Always provide `alt` text for images
- Use semantic HTML elements
- Pass `priority` prop to above-fold images

### Error Handling

- Always handle API errors with try/catch blocks
- Provide user-friendly error messages
- Use TypeScript's type narrowing for runtime checks
- Never expose sensitive error details to users

### Tailwind CSS

- Use Tailwind v4 syntax with `@import "tailwindcss"`
- Define theme variables in `@theme inline` block
- Use dark mode with `dark:` prefix
- Use Tailwind's color scale consistently (e.g., `text-zinc-50`)

## File Structure

```
app/                    # Next.js App Router pages
  layout.tsx           # Root layout
  page.tsx             # Home page
  globals.css          # Global styles
components/            # Reusable React components (create when needed)
lib/                   # Utility functions and helpers (create when needed)
public/                # Static assets
docs/                  # Documentation
```

## Linting Rules

ESLint is configured with:
- `eslint-config-next/core-web-vitals` - Core Web Vitals best practices
- `eslint-config-next/typescript` - TypeScript-specific rules

Ignored directories: `.next/`, `out/`, `build/`

## Best Practices

1. **Before committing**: Run `npm run lint` to check for issues
2. **Before deploying**: Run `npm run build` to verify production build
3. **Type safety**: Run `npx tsc --noEmit` to catch type errors early
4. **Component isolation**: Keep components focused and single-responsibility
5. **Performance**: Use React Server Components by default, client components only when needed
6. **Accessibility**: Ensure proper ARIA attributes and keyboard navigation

## Important Notes

- No test framework is currently configured; do not add tests unless explicitly requested
- This is a new project - conventions may evolve
- Follow existing code patterns when adding new files

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mijelDeve) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
