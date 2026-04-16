## finportal-web

> This document provides guidelines for AI assistants working on this codebase.


# AI Instructions for FinPortal Project

This document provides guidelines for AI assistants working on this codebase.

## Project Overview

**FinPortal** is a Next.js 15 financial portal application with:
- TypeScript for type safety
- Tailwind CSS 4 for styling
- shadcn/ui for components
- Route groups for layout organization
- Context-based breadcrumb system

## Architecture Principles

### 1. Route Groups
- Use `(auth)` for public authentication pages
- Use `(portal)` for protected main application pages
- Route group folders use parentheses: `app/(portal)/`
- Route groups don't affect URLs: `(portal)/dashboard` → `/dashboard`

### 2. Layout System
- **AuthLayout**: Centered, no sidebar - for login/register
- **MainLayout**: Sidebar + header + breadcrumbs - for main app
- Layouts are enforced at route group level in `layout.tsx`
- Pages should NOT wrap themselves in layouts

### 3. Breadcrumb System
- Uses React Context (`contexts/breadcrumb-context.tsx`)
- Pages set breadcrumbs via `useBreadcrumbs()` hook
- MainLayout reads and displays breadcrumbs from context
- This ensures consistent layout across all portal pages

## File Structure Rules

```
app/
├── (auth)/              # Public pages
│   ├── login/
│   │   └── page.tsx
│   └── layout.tsx       # Wraps with AuthLayout
│
├── (portal)/            # Protected pages
│   ├── dashboard/
│   │   └── page.tsx
│   └── layout.tsx       # Wraps with BreadcrumbProvider + MainLayout
│
├── layout.tsx           # Root layout
└── page.tsx             # Root page (redirects)

components/
├── layouts/             # Layout wrappers
├── navigation/          # Navigation components
├── auth/                # Auth components
└── ui/                  # shadcn/ui components

contexts/
└── breadcrumb-context.tsx  # Breadcrumb state

config/
└── navigation.ts        # Navigation menu config

types/
├── index.ts             # Type exports
└── navigation.ts        # Navigation types
```

## Code Patterns

### Adding a New Portal Page

```tsx
// app/(portal)/your-page/page.tsx
"use client"

import { useEffect } from "react"
import { useBreadcrumbs } from "@/contexts/breadcrumb-context"

export default function YourPage() {
  const { setBreadcrumbs } = useBreadcrumbs()

  useEffect(() => {
    setBreadcrumbs([
      { label: "Parent", href: "/parent" },
      { label: "Your Page" },
    ])
  }, [setBreadcrumbs])

  return (
    <div>
      {/* Just content - MainLayout is automatic */}
    </div>
  )
}
```

**Key Points:**
- Add `"use client"` directive
- Use `useBreadcrumbs()` hook
- Set breadcrumbs in `useEffect`
- Return only content (no layout wrapper)
- Include `setBreadcrumbs` in dependency array

### Adding a New Component

```tsx
// components/category/component-name.tsx

interface ComponentNameProps {
  title: string
  children: React.ReactNode
}

export function ComponentName({ title, children }: ComponentNameProps) {
  return (
    <div>
      <h2>{title}</h2>
      {children}
    </div>
  )
}
```

**Key Points:**
- Use PascalCase for component names
- Define TypeScript interfaces for props
- Export as named export
- Use kebab-case for file names

### Updating Navigation

```typescript
// config/navigation.ts
import { YourIcon } from "lucide-react"

export const navMain: NavSection[] = [
  {
    title: "Your Section",
    url: "/your-page",
    icon: YourIcon,
    items: [
      { title: "Sub Item", url: "/your-page/sub" },
    ],
  },
]
```

## Naming Conventions

- **Files**: `kebab-case.tsx` (e.g., `user-profile.tsx`)
- **Components**: `PascalCase` (e.g., `UserProfile`)
- **Functions**: `camelCase` (e.g., `getUserData`)
- **Types/Interfaces**: `PascalCase` (e.g., `UserProfile`)
- **Constants**: `UPPER_SNAKE_CASE` (e.g., `API_URL`)
- **Pages**: Always `page.tsx`
- **Layouts**: Always `layout.tsx`

## Import Rules

### Always Use Path Alias
```tsx
// ✅ Correct
import { Button } from "@/components/ui/button"
import { useBreadcrumbs } from "@/contexts/breadcrumb-context"
import { NavSection } from "@/types"

// ❌ Wrong
import { Button } from "../../components/ui/button"
```

### Import Order
1. React imports
2. Third-party libraries
3. Internal components (with `@/`)
4. Types
5. Styles

```tsx
import { useEffect, useState } from "react"
import { useRouter } from "next/navigation"
import { Button } from "@/components/ui/button"
import { useBreadcrumbs } from "@/contexts/breadcrumb-context"
import type { User } from "@/types"
```

## Common Mistakes to Avoid

### ❌ Don't Wrap Pages in MainLayout
```tsx
// ❌ WRONG
export default function Page() {
  return (
    <MainLayout breadcrumbs={[...]}>
      <div>Content</div>
    </MainLayout>
  )
}

// ✅ CORRECT
export default function Page() {
  const { setBreadcrumbs } = useBreadcrumbs()
  useEffect(() => {
    setBreadcrumbs([...])
  }, [setBreadcrumbs])
  
  return <div>Content</div>
}
```

### ❌ Don't Forget "use client" for Hooks
```tsx
// ❌ WRONG
import { useBreadcrumbs } from "@/contexts/breadcrumb-context"

export default function Page() {
  const { setBreadcrumbs } = useBreadcrumbs() // Error!
}

// ✅ CORRECT
"use client"

import { useBreadcrumbs } from "@/contexts/breadcrumb-context"

export default function Page() {
  const { setBreadcrumbs } = useBreadcrumbs()
}
```

### ❌ Don't Forget Dependencies in useEffect
```tsx
// ❌ WRONG
useEffect(() => {
  setBreadcrumbs([...])
}, []) // Missing setBreadcrumbs

// ✅ CORRECT
useEffect(() => {
  setBreadcrumbs([...])
}, [setBreadcrumbs])
```

### ❌ Don't Include Route Group in URLs
```tsx
// ❌ WRONG
<Link href="/(portal)/dashboard">Dashboard</Link>

// ✅ CORRECT
<Link href="/dashboard">Dashboard</Link>
```

## TypeScript Guidelines

### Always Define Types
```tsx
// ✅ Good
interface PageProps {
  params: { id: string }
  searchParams: { [key: string]: string | string[] | undefined }
}

export default function Page({ params, searchParams }: PageProps) {
  // ...
}
```

### Use Type Imports When Possible
```tsx
import type { User } from "@/types"
import type { NavSection } from "@/types/navigation"
```

### Avoid `any`
```tsx
// ❌ Avoid
const data: any = await fetch(...)

// ✅ Better
interface ApiResponse {
  data: User[]
  total: number
}

const response: ApiResponse = await fetch(...)
```

## Component Guidelines

### Use Composition
```tsx
// ✅ Good - Composable
<Card>
  <CardHeader>
    <CardTitle>Title</CardTitle>
  </CardHeader>
  <CardContent>Content</CardContent>
</Card>

// ❌ Avoid - Monolithic
<Card title="Title" content="Content" />
```

### Keep Components Small
- Single responsibility principle
- Extract reusable logic into hooks
- Split large components into smaller ones

### Use Proper Event Handlers
```tsx
// ✅ Good
<Button onClick={handleClick}>Click</Button>

// ❌ Avoid
<Button onClick={() => setCount(count + 1)}>Click</Button>
```

## Styling Guidelines

### Use Tailwind Classes
```tsx
// ✅ Preferred
<div className="flex items-center gap-4 p-4">

// ❌ Avoid inline styles
<div style={{ display: 'flex', gap: '16px' }}>
```

### Use cn() for Conditional Classes
```tsx
import { cn } from "@/lib/utils"

<div className={cn(
  "base-classes",
  isActive && "active-classes",
  variant === "primary" && "primary-classes"
)}>
```

## When Making Changes

### Before Adding New Features
1. Check if similar functionality exists
2. Review `config/navigation.ts` for menu structure
3. Check `types/` for existing type definitions
4. Look in `components/` for reusable components

### When Refactoring
1. Maintain the route group structure
2. Keep breadcrumb context pattern
3. Don't break the layout hierarchy
4. Update types if data structures change
5. Update documentation if architecture changes

### When Fixing Bugs
1. Check if it's a client/server component issue
2. Verify imports use `@/` alias
3. Check useEffect dependencies
4. Verify route group parentheses
5. Check if breadcrumbs are set correctly

## Testing Checklist

When implementing changes, verify:
- [ ] TypeScript compiles without errors (`npx tsc --noEmit`)
- [ ] ESLint passes (`npm run lint`)
- [ ] Dev server runs (`npm run dev`)
- [ ] Production build succeeds (`npm run build`)
- [ ] All routes are accessible
- [ ] Breadcrumbs display correctly
- [ ] Sidebar navigation works
- [ ] Responsive on mobile

## Git Commit Guidelines

Follow the project's commit message format:
```
type(scope): subject

Examples:
feat(portal): add settings page
fix(breadcrumb): resolve context error
docs(readme): update installation steps
refactor(navigation): extract menu config
style(dashboard): improve card spacing
```

Types: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`

## Branch Naming

Follow the user's defined rules:
```
feature/s1/issue-title
fix/s1/issue-title
docs/s1/issue-title
refactor/s1/issue-title
```

Where `s1` is the sprint number.

## Resources

- [Quick Start Guide](./docs/QUICK_START.md) - How-to for common tasks
- [Architecture Overview](./docs/ARCHITECTURE.md) - System design
- [Project Structure](./STRUCTURE.md) - Detailed folder structure
- [Next.js 15 Docs](https://nextjs.org/docs)
- [shadcn/ui Docs](https://ui.shadcn.com)

## Summary

**Key Principles:**
1. Use route groups for layout organization
2. Context-based breadcrumbs (never pass as props)
3. Pages return content only (layouts are automatic)
4. Always use `@/` import alias
5. TypeScript everything
6. Follow naming conventions
7. Keep components small and focused

When in doubt, check existing code patterns in the codebase!

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Theplus1) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
