## phishmonger

> npm run tauri:dev      # Tauri dev mode

# AGENTS.md - PhishMonger Development Guide

## Build Commands

```bash
# Development server
npm run dev

# Production build
npm run build

# Preview production build
npm run preview

# Lint with ESLint
npm run lint

# Tauri commands (desktop app)
npm run tauri:dev      # Tauri dev mode
npm run tauri:build    # Tauri production build
```

## Testing

**No test framework is currently configured.** The project does not have unit tests set up. When adding tests, consider:
- Vitest (bundled with Vite)
- React Testing Library for component tests
- Place tests in `src/**/*.test.ts` or `src/**/*.spec.ts`

## Code Style Guidelines

### Imports

**Order (grouped with blank lines between):**
1. React imports
2. Third-party library imports
3. Absolute imports from project root (`@/` or direct paths)
4. Relative imports
5. Type imports (`import type { ... }`)
6. CSS imports (always last)

**Examples:**
```typescript
// React imports
import { useState, useEffect, useCallback } from 'react'
import { useHotkeys } from 'react-hotkeys-hook'

// Third-party
import { v4 as uuidv4 } from 'uuid'

// Local components
import { HTMLInput } from './components/HTMLInput'
import { CampaignManager } from './components/campaign/CampaignManager'

// Types
import type { Campaign } from './types/campaign'
import type { ScoringData } from './types/scoring'

// CSS (always last)
import './index.css'
```

### TypeScript Conventions

- **Always use `import type`** for type-only imports to enable better tree-shaking
- **Strict mode enabled** - no implicit any, strict null checks
- **Prefer `interface` over `type`** for object shapes
- Use `type` for unions, tuples, and mapped types
- Explicit return types on exported functions
- Use `const` for immutable variables, `let` only when necessary

### Naming Conventions

| Category | Convention | Example |
|----------|-----------|---------|
| Components | PascalCase | `CampaignManager`, `PhishImportModal` |
| Hooks | camelCase, prefix `use` | `useCampaigns`, `useUndoRedo` |
| Functions | camelCase | `loadAnnotations`, `saveScoring` |
| Types/Interfaces | PascalCase | `Campaign`, `Annotation` |
| Constants | UPPER_SNAKE_CASE | `STORAGE_KEY`, `CAMPAIGNS_KEY` |
| LocalStorage keys | kebab-case, prefix | `phishmonger-campaigns` |
| File names | PascalCase (components), camelCase (utils) | `CampaignManager.tsx`, `storage.ts` |

### Error Handling

**Always wrap LocalStorage and external operations:**
```typescript
try {
  localStorage.setItem(key, value)
} catch (error) {
  console.error('Failed to save:', error)
  return defaultValue
}
```

**Graceful fallbacks:** Return empty objects/arrays rather than throwing for data loading functions.

### Comments

- Use JSDoc for exported functions with `@param` and `@returns`
- Section comments for code organization: `// Campaign management`, `// Effects`
- Document "why", not "what" - code should be self-documenting

### Formatting

- Indent: 2 spaces
- Semicolons: optional (be consistent within a file)
- Single quotes for strings
- Trailing commas in multi-line object/array literals
- Max line length: ~100 characters (soft limit)

### React Patterns

- **Functional components only** - no class components
- **Custom hooks** for reusable logic (`useCampaigns`, `useUndoRedo`)
- **Destructured props** in component parameters
- **`useCallback`** for event handlers passed to children
- **`useMemo`** for expensive computations

```typescript
// Good
export function MyComponent({ prop1, prop2 }: MyProps) {
  const handleClick = useCallback(() => {
    // handler logic
  }, [dep])

  return <div onClick={handleClick}>{prop1}</div>
}
```

### State Management

- React hooks for local state
- LocalStorage for persistence (via utility functions in `utils/storage.ts`)
- No global state library - lift state up as needed

### Project Structure

```
src/
  components/          # React components
    campaign/          # Campaign-related components
    import/            # Import modal components
    library/           # Technique library components
    preview/           # Preview mode components
    visualizer/        # Visualization controls
  hooks/               # Custom React hooks
  types/               # TypeScript type definitions
  utils/               # Utility functions
  data/                # Static JSON data
  extensions/          # TipTap extensions
```

### Key Patterns

**UUID Generation:** Always use `crypto.randomUUID()` - never manual construction.

**Date Format:** ISO 8601 strings: `new Date().toISOString()`

**Type Exports:** Export types separately from values:
```typescript
export { CampaignManager }
export type { CampaignInput }
```

**CSS:** Use CSS Modules for component styles (`*.module.css`) or global `index.css`.

### Dependencies

**Key libraries in use:**
- React 19, TypeScript 5.9
- TipTap (rich text editor)
- Tauri (desktop app wrapper)
- Vite (build tool)

**Before adding new dependencies:** Check if functionality exists in existing libraries.

### Pre-commit Checklist

- [ ] `npm run lint` passes with no errors
- [ ] `npm run build` succeeds
- [ ] TypeScript compiles without errors
- [ ] No `console.log` left in production code (use `console.error` for errors)

---
> Source: [zarguell/phishmonger](https://github.com/zarguell/phishmonger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
