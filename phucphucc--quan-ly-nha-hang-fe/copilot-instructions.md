## quan-ly-nha-hang-fe

> - **Framework**: Next.js 16, React 19

# FoodHub Frontend - Agent Guidelines

## Project Overview
- **Framework**: Next.js 16, React 19
- **Language**: TypeScript (strict mode)
- **Styling**: Tailwind CSS v4, clsx, tailwind-merge
- **State**: Zustand
- **Data Fetching**: TanStack Query + Axios
- **Forms**: React Hook Form + Zod
- **UI**: Radix UI primitives + custom shadcn-like components
- **Testing**: Vitest + React Testing Library

## Commands
```bash
npm run dev           # Dev server
npm run build        # Production build
npm run lint         # ESLint
npm run type-check   # TypeScript check
npm run test         # Run all tests
npm run test:watch   # Watch mode
npm run test -- src/file.test.tsx      # Single test file
npm run test -- --run -t "pattern"    # Tests matching pattern
```

## Code Style

### Imports (auto-sorted by ESLint)
1. React/Next → 2. External libs → 3. @/ imports → 4. ./ imports
```typescript
import { useState } from 'react'
import { useRouter } from 'next/navigation'
import { cn } from '@/lib/utils'
import { Button } from '@/components/ui/button'
import styles from './Component.module.css'
```
**No barrel files**: `import { Button } from '@/components/ui/button'` (not from index)

### Naming
- Components: PascalCase (`OrderList`)
- Hooks: `useXxx` (`useCartStore`)
- Types: PascalCase (`Employee`)
- Files: kebab-case (`order-list.tsx`)

### TypeScript
- No `any` - use `unknown` and narrow
- Explicit return types for exported functions

### Styling
```typescript
import { cn } from '@/lib/utils'
<div className={cn('base', isActive && 'active', className)} />
```

### JSX Literals
**CẤM text trực tiếp trong JSX** - Use constants:
```typescript
import { UI_TEXT } from '@/lib/UI_Text'
<p>{UI_TEXT.LOGIN.WELCOME}</p>
```
Allowed: `:`, ` `, `: `, `-`, ` - `, `—`, `•`, `·`, `đ`, `.`, `/`

### Error Handling
```typescript
import { getErrorMessage } from '@/lib/error'
try { await apiCall() } 
catch (err) { toast.error(getErrorMessage(err)) }
```

## Patterns

### Zustand Store
```typescript
import { create } from 'zustand'
type State = { items: Item[]; addItem: (item: Item) => void }
export const useStore = create<State>()((set) => ({
  items: [],
  addItem: (item) => set(s => ({ items: [...s.items, item] })),
}))
```

### Form (React Hook Form + Zod)
```typescript
import { zodResolver } from '@hookform/resolvers/zod'
import { useForm } from 'react-hook-form'
const schema = z.object({ name: z.string().min(1) })
type FormData = z.infer<typeof schema>
const form = useForm<FormData>({ resolver: zodResolver(schema) })
```

### TanStack Query
```typescript
import { useQuery } from '@tanstack/react-query'
export function useOrders() {
  return useQuery({ queryKey: ['orders'], queryFn: () => axios.get('/orders') })
}
```

### Next.js
- Server Components by default, `'use client'` only when needed
- Lazy load: `const X = dynamic(() => import('./X'), { ssr: false })`

## Directory Structure
```
src/
├── app/              # Next.js App Router
├── components/
│   ├── features/    # Domain components (menu/, order/, etc.)
│   ├── shared/     # Shared components
│   └── ui/         # Base UI (button, input, dialog...)
├── hooks/           # Custom hooks
├── lib/             # Utils, zod-schemas, configs
├── providers/      # React contexts
├── services/       # API functions
├── store/          # Zustand stores
└── types/          # TypeScript types
```

## Performance
Follow Vercel best practices (see `.agents/skills/vercel-react-best-practices/`):
- Avoid waterfalls: use `Promise.all()` for parallel fetches
- Avoid barrel file imports
- Minimize client component props
- Use `React.cache()` for server-side deduplication

---
> Source: [PhucPhucc/quan-ly-nha-hang-fe](https://github.com/PhucPhucc/quan-ly-nha-hang-fe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-01 -->
