## frontend

> - **Speed over purity** - Ship features fast, refactor later

# Frontend Architecture - Vertical Slices

## 🎯 Philosophy: PRAGMATIC OVER PERFECT
- **Speed over purity** - Ship features fast, refactor later
- **Simple patterns** - Avoid over-engineering 
- **Consistency** - Follow established patterns for AI predictability

## 📁 Structure
```
src/
├── features/          # Self-contained feature slices
├── shared/           # Reusable components/hooks
├── api/              # 🤖 ORVAL GENERATED (DO NOT EDIT)
└── app/              # Next.js App Router
```

## 🏗️ Feature Slice Pattern
```
features/feature-name/
├── index.ts          # Exports (import { X } from '@/features/auth')
├── components/       # Feature components
├── hooks/           # Wraps api/hooks with business logic
└── [name]-context.tsx # Feature state (if needed)
```

## 🤖 Orval Integration - CRITICAL
- **Generated**: `api/hooks/` (React Query) + `api/models/` (DTOs)
- **Regenerate**: `npm run api:generate` after backend changes
- **Pattern**: Features wrap generated hooks, add business logic
- **🚨 NEVER EDIT**: Generated files will be overwritten

## 🔧 Tech Stack
- **Next.js 15** + **React 19** + **TypeScript 5**
- **React Query** + **shadcn/ui** + **Tailwind CSS**
- **Axios** (JWT auth) + **SignalR** (real-time)

## 📋 Patterns

### Imports
```typescript
// External first
import { useState } from 'react';
// Internal with aliases
import { Button } from '@/shared/components/ui/button';
import { useAuth } from '@/features/auth';
import { useUsers } from '@/features/users'; // ✅ Cross-feature OK
```

### Components
```typescript
export function Component({ prop }: Props) {
  // Hooks first - use namespaced, not destructured
  const data = useData();
  const form = useForm();
  
  // Event handlers
  const handleClick = () => data.refresh();
  
  // Render
  return <div>{content}</div>;
}
```

### Hook Usage - CRITICAL PATTERN
**Use namespaced hooks, NOT destructuring for cleaner code:**

```typescript
// ❌ BAD: Destructuring chaos
const { data, isLoading, error, hasData, create, update, delete: deleteItem, isCreating, isUpdating, isDeleting } = useItems();
const { name, setName, description, setDescription, isVisible, open, close, submit } = useForm();

// ✅ GOOD: Clean namespaced
const items = useItems();
const form = useForm();

// Usage becomes crystal clear:
<button onClick={items.create} disabled={items.isCreating}>
  {items.isCreating ? 'Creating...' : 'Create'}
</button>
<input value={form.name} onChange={form.setName} />
```

**Benefits:**
- **5 lines instead of 20+** - massive reduction in component noise
- **Clear origins** - `items.create()` vs mystery `create()` function
- **Better IntelliSense** - type `items.` to see all available methods
- **No naming conflicts** - multiple hooks can have `isLoading`
- **Easier refactoring** - hook API changes don't break destructuring

### Code as Poetry
**Write code that reads like poetry - simple, clear, purposeful:**

```typescript
function MyComponent() {
  // Clean setup - like verses in a poem
  const data = useData();
  const form = useForm();
  const animations = useAnimations();

  // Guard clauses - clear and direct
  if (data.isLoading) return <Loading />;
  if (data.error) return <Error />;

  // Pure composition - every line has purpose
  return (
    <div>
      <Header />
      <CreateForm data={data} form={form} />
      <ItemsList data={data} animations={animations} />
    </div>
  );
}
```

**Poetry Principles:**
- **Rhythm** - Setup → Guard → Compose (natural flow)
- **Clarity** - Every variable name tells its story
- **Purpose** - No line exists without reason
- **Simplicity** - Complex is the enemy of readable

### API Hook Wrapping
```typescript
// features/users/hooks/use-users.ts
import { useGetUsers } from '@/api/hooks'; // Generated

export function useUsers() {
  const query = useGetUsers();
  
  // Add business logic here
  return {
    ...query,
    displayUsers: query.data?.map(u => ({
      ...u,
      fullName: `${u.firstName} ${u.lastName}`
    }))
  };
}
```

## 🎯 Key Rules
- ✅ Never edit `api/hooks/` or `api/models/`
- ✅ Features CAN import from other features (be pragmatic)
- ✅ Use Server Components by default, 'use client' when needed
- ✅ JWT from localStorage, axios interceptors for auth
- ✅ Wrap generated hooks in feature-specific hooks
- ✅ Keep it simple - avoid over-engineering
- ✅ Consistency over cleverness
- ✅ keep the design clear, do not overuse emojis, make it feel real, and clean



## 🚨 Critical Gotchas
- **Orval regeneration** wipes `api/` folder - never edit directly
- **'use client'** needed for hooks, state, event handlers
- **SignalR connections** need cleanup in useEffect
- **JWT expiry** - handle 401s gracefully
- **Loading states** - always handle isLoading/error from React Query

---
> Source: [WilliamAvHolmberg/vibecodementor](https://github.com/WilliamAvHolmberg/vibecodementor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-06 -->
