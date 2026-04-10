## milanesa

> > A Next.js 16 app showcasing songs learned on bass guitar, with sheet music rendering via VexFlow.

# AGENTS.md - Milanesa Project

> A Next.js 16 app showcasing songs learned on bass guitar, with sheet music rendering via VexFlow.

## Build & Run Commands

```bash
# Package manager: pnpm (preferred based on overrides in package.json)
pnpm install         # Install dependencies
pnpm dev             # Development server (http://localhost:3000)
pnpm build           # Production build
pnpm start           # Start production server
pnpm lint            # Run ESLint
```

## Testing

No test framework is currently configured. When adding tests:
- Use Vitest (recommended for Next.js) or Jest
- Place tests in `__tests__/` directories or use `.test.ts(x)` / `.spec.ts(x)` suffixes

## Project Structure

```
app/
  ├── page.tsx              # Homepage - song grid
  ├── layout.tsx            # Root layout with Inter font
  ├── globals.css           # Tailwind v4 + CSS variables (oklch colors)
  ├── about/page.tsx        # About page
  ├── songs/[slug]/page.tsx # Dynamic song detail page
  ├── api/route.ts          # API endpoint for song data
  └── db/data.json          # Song data (JSON database)
components/
  ├── ui/                   # shadcn/ui primitives (Card, Breadcrumb)
  ├── song-card.tsx         # Song grid card component
  ├── song-list.tsx         # Song grid container
  ├── header.tsx            # Breadcrumb navigation
  └── title.tsx             # Page title component
lib/
  └── utils.ts              # cn() utility (clsx + tailwind-merge)
```

## Code Style Guidelines

### TypeScript

- **Strict mode enabled** - all `strict` compiler options active
- **No type suppression** - never use `as any`, `@ts-ignore`, `@ts-expect-error`
- **Prefer interfaces** for props: `interface ComponentProps { ... }`
- **Use explicit return types** for exported functions when complex

### Imports

```typescript
// Order: React/Next → External libs → Internal absolute (@/) → Relative
import { useState } from "react"
import { Calendar, Music } from "lucide-react"
import { cn } from "@/lib/utils"
import Link from "next/link"

// Path aliases configured:
// @/* → ./*
// @/components/* → ./components/*
```

### Component Patterns

```typescript
// Client components: explicit "use client" at top
"use client"

// Props: interface with explicit types
interface SongCardProps {
    song: Song
    className?: string
    index: number
}

// Function components (not arrow for top-level exports)
export function SongCard({ song, className, index }: SongCardProps) { ... }

// Default export for page components
function About() { ... }
export default About
```

### Styling

- **Tailwind CSS v4** with CSS-first configuration
- **CSS Variables**: Use oklch color space variables defined in `globals.css`
- **cn() utility**: Always use for conditional/merged classes
- **Semantic colors**: `text-foreground`, `bg-background`, `text-muted-foreground`, etc.

```typescript
// Correct: use cn() for conditional classes
className={cn(
    "base-classes",
    condition && "conditional-classes",
    className
)}

// Tailwind classes in use: rounded-2xl, hover:scale-[1.02], shadow-primary/20
```

### Naming Conventions

| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `SongCard`, `BreadcrumbList` |
| Files | kebab-case | `song-card.tsx`, `song-list.tsx` |
| Hooks | camelCase with use prefix | `useState`, `usePathname` |
| Variables | camelCase | `isHovered`, `formatDate` |
| Constants | UPPER_SNAKE or camelCase | depends on scope |
| Interfaces | PascalCase | `SongCardProps`, `Song` |

### Error Handling

- **API routes**: Return explicit error responses with status codes
- **Data fetching**: Handle missing data gracefully with fallback UI
- **No empty catch blocks** - always handle or log errors

```typescript
// API route example
if (!slug) return new Response('Slug is required', { status: 400 })

// Page data handling
if (!song) return <div>Song not found</div>
```

### React Patterns

- **Server Components by default** - only add "use client" when needed
- **State management**: useState for local UI state
- **Event handlers**: inline arrow functions for simple cases
- **Conditional rendering**: ternary for simple, && for single branch

### Next.js Specifics

- **App Router** (Next.js 16) - all routes in `app/` directory
- **Dynamic routes**: `[slug]` folder pattern
- **API routes**: `route.ts` files with named exports (GET, POST, etc.)
- **Fonts**: Use `next/font/google` for optimization
- **Third-party embeds**: Use `@next/third-parties` (e.g., YouTubeEmbed)

### Data Flow

- **Static data**: JSON file in `app/db/data.json`
- **API access**: Internal fetch to `/api?slug=xxx` for dynamic pages
- **Environment-aware URLs**: Check `process.env.NODE_ENV` for base URL

## Dependencies Overview

| Package | Purpose |
|---------|---------|
| `accidentaljs` | VexFlow sheet music renderer |
| `framer-motion` / `motion` | Animations |
| `lucide-react` | Icons |
| `@radix-ui/react-slot` | Primitive composability |
| `class-variance-authority` | Component variants |
| `clsx` + `tailwind-merge` | Class name utilities |

## ESLint Configuration

Uses flat config (`eslint.config.mjs`):
- `eslint-config-next/core-web-vitals` - Next.js performance rules
- `eslint-config-next/typescript` - TypeScript rules

## Git Workflow

- **Don't commit** unless explicitly requested
- **No force pushes** to main
- **Verify changes** with `pnpm lint` before committing

## Key Decisions for Agents

1. **Visual changes** (colors, spacing, animations) → delegate to frontend-ui-ux-engineer
2. **Logic changes** (data flow, API, state) → handle directly
3. **Always run** `pnpm lint` after changes
4. **Match existing patterns** - this is a disciplined codebase
5. **Spanish UI** - the app is in Spanish (Argentina locale: es-ES)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lautaroroldan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/lautaroroldan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
