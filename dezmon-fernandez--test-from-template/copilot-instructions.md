## test-from-template

> This file provides guidance to Claude Code when working with React 19 + Supabase projects.

# React 19 + Supabase MVP Template

This file provides guidance to Claude Code when working with React 19 + Supabase projects.

## Technology Stack

| Layer | Technology | Version |
|-------|------------|---------|
| **Frontend** | React | 19.x |
| **Build** | Vite | 6.x |
| **Package Manager** | pnpm | 9.x |
| **Routing** | TanStack Router | Latest |
| **Data Fetching** | TanStack Query | v5 |
| **UI Components** | shadcn/ui | Latest |
| **Styling** | Tailwind CSS | v4 |
| **Forms** | React Hook Form + Zod | Latest |
| **Backend** | Supabase | Latest |
| **Linting** | Biome | 2.0+ |
| **Payments** | Stripe | Latest |
| **AI** | Vercel AI SDK | 5.x |
| **Testing** | Vitest + Testing Library | Latest |

## Architecture: Vertical Slices

```
src/
├── routes/           # TanStack Router file-based routes (thin)
├── features/         # Self-contained vertical slices
│   └── [feature]/
│       ├── __tests__/
│       ├── components/
│       ├── hooks/
│       ├── schemas/
│       ├── types/
│       └── index.ts  # Public API
└── shared/           # UI components, utils (NO business logic)
```

**Import Rules:**
- Routes → Features ✅
- Features → Shared ✅
- Features → Features ✅
- Shared → Features ❌ NEVER

## Development Commands

```bash
# Development
pnpm dev              # Start dev server
pnpm build            # Production build

# Code Quality
pnpm biome check .    # Lint and format
pnpm tsc --noEmit     # Type check

# Testing
pnpm test             # Run tests (watch)
pnpm test --run       # Run once (CI)
pnpm test src/features/[feature]  # Test specific feature

# Supabase
supabase start        # Local Supabase
supabase db reset     # Reset with migrations
supabase gen types typescript --local > src/shared/types/database.types.ts
```

## Testing

### Per-Phase Testing (CRITICAL)
| Phase | Run Tests? | Command |
|-------|------------|---------|
| Hooks | **YES** | `pnpm test src/features/[feature]/__tests__/use-*` |
| Components | **YES** | `pnpm test src/features/[feature]/__tests__/` |
| Routes | **YES** | `pnpm test --run` (full suite) |

### Testing Stack
- Vitest - Test runner
- @testing-library/react - Component testing
- @testing-library/user-event - User interactions

## Code Patterns

### React 19
```typescript
function Component(): ReactElement {
  return <div>...</div>;
}
// Don't manually memoize - React Compiler handles it
```

### TanStack Query v5
```typescript
const { data, isPending, error } = useQuery({ ... });
// isPending not isLoading, gcTime not cacheTime
```

### Supabase
```typescript
import { createBrowserClient } from '@supabase/ssr';
// Always index RLS policy columns
```

## PRP Workflow

**New App**: Edit `PRPs/INITIAL.md` → `/generate-react-supabase-prp PRPs/INITIAL.md`

**New Feature**: Edit `PRPs/FEATURE.md` → `/generate-react-supabase-prp PRPs/FEATURE.md`

**Quick Feature**: `/generate-react-supabase-prp "add dark mode toggle"`

Then execute: `/execute-react-supabase-prp PRPs/[generated-file].md`

## Common Gotchas

- **TanStack Query v5**: `isLoading` → `isPending`
- **TanStack Router**: Dynamic params use `$`: `$projectId.tsx`
- **Supabase**: RLS returns empty array, not error
- **Stripe**: Secret key ONLY in Edge Functions
- **AI SDK 5**: Import from `@ai-sdk/react`, use `message.parts`
- **Tailwind v4**: No config file - use `@theme` in CSS
- **Testing**: Run tests after each phase, don't batch to the end

## Environment Variables

### Client (.env.local)
```
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
VITE_STRIPE_PUBLISHABLE_KEY=
```

### Edge Functions (supabase secrets set)
```
STRIPE_SECRET_KEY=
STRIPE_WEBHOOK_SECRET=
OPENAI_API_KEY=
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dezmon-fernandez)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dezmon-fernandez)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
