## sim-components

> Component patterns and structure for React components


# Component Patterns

## Structure Order

```typescript
'use client' // Only if using hooks

// Imports (external → internal)
// Constants at module level
const CONFIG = { SPACING: 8 } as const

// Props interface
interface ComponentProps {
  requiredProp: string
  optionalProp?: boolean
}

export function Component({ requiredProp, optionalProp = false }: ComponentProps) {
  // a. Refs
  // b. External hooks (useParams, useRouter)
  // c. Store hooks
  // d. Custom hooks
  // e. Local state
  // f. useMemo
  // g. useCallback
  // h. useEffect
  // i. Return JSX
}
```

## Rules

1. `'use client'` only when using React hooks
2. Always define props interface
3. Extract constants with `as const`
4. Semantic HTML (`aside`, `nav`, `article`)
5. Optional chain callbacks: `onAction?.(id)`

## Component Extraction

**Extract when:** 50+ lines, used in 2+ files, or has own state/logic

**Keep inline when:** < 10 lines, single use, purely presentational

---
> Source: [simstudioai/sim](https://github.com/simstudioai/sim) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
