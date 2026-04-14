## ohmydashboard

> | Item | Convention | Example |

# Code Quality Rules

## Naming

| Item | Convention | Example |
|---|---|---|
| Files | kebab-case | `smart-input-field.tsx` |
| Components | PascalCase | `SmartInputField` |
| Hooks | camelCase with `use` prefix | `useDebounce` |
| Store hooks | camelCase with `use` prefix + `Store` suffix | `useThemeStore` |
| Constants | UPPER_SNAKE_CASE | `MAX_RETRIES` |
| Types/Interfaces | PascalCase | `StudentFilters` |
| Barrel files | Always `index.ts` | — |

## File Size Limits

| Type | Max Lines | Action |
|---|---|---|
| Component | 300 | Split into sub-components |
| Hook | 100 | Extract utilities |
| Utility | 200 | Split by concern |
| Page | 200 | Extract to components |
| Store | 100 | Split by domain |

## TypeScript

- Explicit interfaces for all component props
- Export types alongside components: `export type { FooProps }`
- Use `import type` for type-only imports
- No `any` — use `unknown` and narrow
- Zod schemas for runtime validation (in `core/validation/`)

## Styling

- ALL colors via theme tokens — zero hardcoded hex/rgb/hsl
- Use Tailwind semantic classes (`bg-primary`, `text-muted-foreground`)
- Use `cn()` from `@/core/utils` for class merging
- See `config/theme/types.ts` for available tokens

## Components

- Named exports only (no `export default` except Next.js pages)
- `'use client'` directive only when needed (events, hooks, state)
- Composition over massive prop APIs
- Every component has explicit TypeScript interface

## Testing

- Tests co-located: `module.test.ts` next to `module.ts`
- Use custom `render` from `@/test/utils` for component tests
- UTC dates for timezone safety
- `toBeCloseTo` for floating-point comparisons

## Imports

```tsx
// ✅ Correct
import { Button } from '@/components/ui/button'
import { cn } from '@/core/utils'
import type { NavItem } from '@/types'

// ❌ Wrong
import { Button } from '../../../components/ui/button'
import { cn } from '../../lib/utils'
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrjw717) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
