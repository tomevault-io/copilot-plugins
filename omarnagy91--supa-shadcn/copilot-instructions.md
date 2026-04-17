## supa-shadcn

> Main project rules and context for AI coding assistants


# Supa-Shadcn Template - AI Assistant Guidelines

This is a **template repository** for building web applications using **Next.js**, **Supabase**, and **shadcn/ui** with multiple component registries. This template is optimized for "vibe coding" - rapid prototyping and development with AI assistance.

## Project Overview

- **Framework**: Next.js 16 (App Router) with React 19
- **Backend**: Supabase (PostgreSQL, Auth, Storage, Realtime, Edge Functions)
- **UI Components**: shadcn/ui with 60+ component registries
- **Styling**: Tailwind CSS 4 with CSS variables
- **Icons**: Lucide React
- **Package Manager**: pnpm (preferred) or npm/yarn

## Project Structure

```
.
├── app/                    # Next.js App Router pages and layouts
│   ├── layout.tsx         # Root layout
│   ├── page.tsx           # Home page
│   └── globals.css        # Global styles and Tailwind imports
├── components/            # React components (create as needed)
│   └── ui/               # shadcn/ui components (auto-generated)
├── lib/                   # Utility functions and configurations
│   ├── client.ts         # Supabase browser client
│   ├── server.ts         # Supabase server client
│   ├── middleware.ts     # Supabase middleware
│   └── utils.ts          # Utility functions (cn, etc.)
├── .cursor/               # Cursor IDE rules and configurations
│   └── rules/            # Specific coding rules (Supabase, SQL, etc.)
├── components.json        # shadcn/ui configuration with registries
└── package.json           # Dependencies and scripts
```

## Component Registry System

This project uses **shadcn/ui** with **60+ component registries** configured. When suggesting or adding components:

1. **Check available registries first** - Use `npx shadcn@latest add <component-name>` to search across all registries
2. **Prefer official shadcn components** - The default registry (`@shadcn`) should be preferred unless a specialized component is needed
3. **Use registry-specific components when appropriate**:
   - `@supabase` - Supabase-specific UI components
   - `@magicui` - Advanced animations and effects
   - `@aceternity` - Premium animated components
   - `@blocks` - Pre-built page sections
   - `@formcn` - Form-specific components
   - `@assistant-ui` - AI chat interfaces
   - And 50+ more specialized registries

4. **Component Selection Guidelines**:
   - For basic UI: Use `@shadcn` (default)
   - For animations: Check `@magicui`, `@aceternity`, `@motion-primitives`
   - For forms: Check `@formcn`, `@shadcn`
   - For data display: Check `@shadcn`, `@blocks`
   - For AI features: Check `@assistant-ui`, `@ai-elements`
   - For Supabase integration: Check `@supabase`
   - For commerce: Check `@commercn`, `@billingsdk`
   - For specialized needs: Search registries using shadcn CLI

5. **Adding Components**:
   ```bash
   # Search across all registries
   npx shadcn@latest add <component-name>
   
   # Add from specific registry
   npx shadcn@latest add @registry/component-name
   ```

## Supabase Integration

### Client Setup

- **Browser Client**: Use `lib/client.ts` - `createClient()` for client components
- **Server Client**: Use `lib/server.ts` - `createClient()` for server components and API routes
- **Middleware**: Use `lib/middleware.ts` for authentication middleware

### Environment Variables

Required environment variables (create `.env.local`):
```
NEXT_PUBLIC_SUPABASE_URL=your-project-url
NEXT_PUBLIC_SUPABASE_PUBLISHABLE_OR_ANON_KEY=your-anon-key
```

### Database Guidelines

- Follow rules in `.cursor/rules/create-migration.mdc` for migrations
- Follow rules in `.cursor/rules/create-rls-policies.mdc` for RLS policies
- Follow rules in `.cursor/rules/create-db-functions.mdc` for database functions
- Follow rules in `.cursor/rules/postgres-sql-style-guide.mdc` for SQL style

### Realtime

- Follow rules in `.cursor/rules/use-realtime.mdc` for Realtime implementation
- Prefer `broadcast` over `postgres_changes` for scalability
- Use private channels with proper RLS policies

### Edge Functions

- Follow rules in `.cursor/rules/writing-supabase-edge-functions.mdc`
- Use Deno runtime with npm: and jsr: specifiers
- Place functions in `supabase/functions/` directory

## Coding Standards

### TypeScript

- Use strict TypeScript configuration
- Prefer explicit types over `any`
- Use proper type definitions for Supabase queries
- Generate types using Supabase CLI: `npx supabase gen types typescript --local > types/supabase.ts`

### React/Next.js

- Use Server Components by default, Client Components when needed (`'use client'`)
- Follow Next.js 16 App Router conventions
- Use proper async/await patterns in Server Components
- Use React 19 features appropriately

### Styling

- Use Tailwind CSS utility classes
- Use CSS variables for theming (defined in `app/globals.css`)
- Follow shadcn/ui component patterns
- Use `cn()` utility from `lib/utils.ts` for conditional classes

### File Naming

- Components: PascalCase (e.g., `UserProfile.tsx`)
- Utilities: camelCase (e.g., `formatDate.ts`)
- Pages: Next.js conventions (e.g., `page.tsx`, `layout.tsx`)
- Database: snake_case for tables and columns

## AI Assistant Best Practices

### When Generating Code

1. **Always check existing patterns** - Look at similar components/files before creating new ones
2. **Use proper imports** - Import from `@/` aliases (configured in `tsconfig.json`)
3. **Follow project structure** - Place files in appropriate directories
4. **Add proper TypeScript types** - Don't use `any` unless absolutely necessary
5. **Include error handling** - Add try/catch blocks and proper error messages
6. **Use Supabase best practices** - Follow the rules in `.cursor/rules/` directory
7. **Component selection** - Choose appropriate components from available registries
8. **Responsive design** - Ensure components work on mobile and desktop
9. **Accessibility** - Use semantic HTML and ARIA attributes when needed
10. **Performance** - Use Server Components, proper caching, and optimize images

### When Adding Features

1. **Check for existing implementations** - Search the codebase first
2. **Follow established patterns** - Maintain consistency with existing code
3. **Update related files** - If adding a new page, update navigation if needed
4. **Add proper types** - Create TypeScript types for new data structures
5. **Consider RLS policies** - If adding database tables, add appropriate RLS policies
6. **Add error boundaries** - Use React error boundaries for better error handling
7. **Test locally** - Ensure code builds and runs without errors

### Component Selection Decision Tree

When user needs a component:

1. **Identify the use case**:
   - Basic UI element? → `@shadcn`
   - Animated/visual effect? → `@magicui`, `@aceternity`, `@motion-primitives`
   - Form input/validation? → `@formcn`, `@shadcn`
   - Data table/list? → `@shadcn`, `@blocks`
   - AI/chat interface? → `@assistant-ui`, `@ai-elements`
   - Supabase integration? → `@supabase`
   - Payment/billing? → `@billingsdk`, `@commercn`
   - Specialized feature? → Search registries

2. **Search for component**:
   ```bash
   npx shadcn@latest add <component-name>
   ```

3. **If not found, check specialized registries** based on use case

4. **If still not found, suggest creating custom component** using shadcn patterns

## Common Patterns

### Creating a New Page

```tsx
// app/feature/page.tsx
import { createClient } from '@/lib/server'

export default async function FeaturePage() {
  const supabase = await createClient()
  // Fetch data...
  
  return (
    <div>
      {/* Page content */}
    </div>
  )
}
```

### Creating a Client Component

```tsx
// components/FeatureComponent.tsx
'use client'

import { useState } from 'react'
import { createClient } from '@/lib/client'

export function FeatureComponent() {
  const [state, setState] = useState()
  const supabase = createClient()
  
  // Component logic...
  
  return <div>{/* Component JSX */}</div>
}
```

### Using shadcn Components

```tsx
import { Button } from '@/components/ui/button'
import { Card, CardHeader, CardTitle, CardContent } from '@/components/ui/card'

export function MyComponent() {
  return (
    <Card>
      <CardHeader>
        <CardTitle>Title</CardTitle>
      </CardHeader>
      <CardContent>
        <Button>Click me</Button>
      </CardContent>
    </Card>
  )
}
```

## Important Notes

- **Always ensure the project builds** before considering tasks complete
- **Static generation errors** related to missing Supabase env vars are expected in development
- **Use pnpm** as the preferred package manager (lock file is pnpm-lock.yaml)
- **Follow existing code patterns** - maintain consistency
- **Check for TypeScript errors** - fix any type issues
- **Test in development** - run `pnpm dev` to verify changes

## Getting Help

- Check `.cursor/rules/` for specific guidelines (Supabase, SQL, etc.)
- Review `components.json` for available registries
- Check Next.js 16 docs for App Router patterns
- Check Supabase docs for backend features
- Check shadcn/ui docs for component usage

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/omarnagy91) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
