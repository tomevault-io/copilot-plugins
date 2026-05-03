## bigmama

> > **Purpose**: This file helps AI agents work effectively in this codebase.

# BigMama - Agent Guidelines

> **Purpose**: This file helps AI agents work effectively in this codebase.

## Project Overview

**BigMama** is a Hebrew-language (RTL) family productivity app built with React 18, TypeScript, Vite, and Firebase.

## Build & Development Commands

| Command | Description |
|---------|-------------|
| `npm run dev` | Start Vite dev server |
| `npm run build` | Type check + build for production |
| `npm run lint` | Run ESLint on `.ts,.tsx` files |
| `npm test` | Run all tests (Vitest) |
| `npm test -- src/path/to/file.test.tsx` | Run a single test file |
| `npm test -- --watch` | Run tests in watch mode |
| `npm run emulators` | Start Firebase emulators |

## Tech Stack

- **Framework**: React 18 + TypeScript 5 (strict mode)
- **Bundler**: Vite 4 with `@` path alias to `src/`
- **Backend**: Firebase (Auth, Firestore)
- **Testing**: Vitest + React Testing Library + jsdom
- **Linting**: ESLint with TypeScript, React Hooks, React Refresh plugins
- **Icons**: `lucide-react`
- **Styling**: Vanilla CSS + CSS Modules + CSS variables

## Directory Structure

```
src/
├── components/       # React components
│   ├── common/       # Shared (Layout, Header, Modals)
│   └── [feature]/    # Feature-specific (e.g., requests/)
├── context/          # React Contexts
├── hooks/            # Custom hooks (useAuth, useTasks, etc.)
├── pages/            # Route components
├── services/         # Firebase service layer
├── types/            # TypeScript interfaces
├── utils/            # Helper functions
└── styles/           # Global CSS + variables
```

## Code Style Guidelines

### TypeScript
- **Strict mode enabled** - no `any` types unless absolutely necessary
- Define interfaces in `src/types/models.ts`
- Use explicit return types for exported functions
- Prefer `interface` over `type` for object shapes

### Imports
- Use `@/` alias for all internal imports (e.g., `@/types/models`)
- Group imports: React, external libs, internal (`@/`), relative
- Example:
```typescript
import React, { useState } from 'react';
import { collection } from 'firebase/firestore';
import { Task } from '@/types/models';
import { useAuth } from '@/hooks/useAuth';
```

### React Components
- Use functional components with explicit `Props` interfaces
- Prefer `React.FC<Props>` pattern
- Co-locate CSS files (e.g., `RequestCard.tsx` + `RequestCard.css`)
- Extract logic into custom hooks in `src/hooks/`

### Naming Conventions
| Type | Convention | Example |
|------|------------|---------|
| Components | PascalCase | `TaskCard.tsx` |
| Hooks | camelCase with `use` | `useTasks.ts` |
| Services | camelCase | `tasks.ts` |
| Types/Interfaces | PascalCase | `Task`, `UserRole` |
| CSS files | Same as component | `TaskCard.css` |
| Test files | `.test.tsx` suffix | `TaskCard.test.tsx` |

### Firebase Patterns
- **Never call Firestore directly in components**
- Create service functions in `src/services/[feature].ts`
- Use `onSnapshot` for real-time data in hooks
- Always clean up subscriptions:
```typescript
useEffect(() => {
  const unsubscribe = subscribeToTasks(familyId, setTasks);
  return () => unsubscribe();
}, [familyId]);
```

### Error Handling
- Use try/catch in async service functions
- Log errors with `console.error('Context:', error)`
- Show user-friendly alerts in Hebrew for critical errors

### CSS / RTL
- Use CSS variables from `src/styles/variables.css`
- Use logical properties: `margin-inline-start` instead of `margin-left`
- RTL-first: The app is Hebrew-language (`dir="rtl"` in HTML)

### Testing
- Co-locate tests: `Component.tsx` → `Component.test.tsx`
- Use `describe` + `it` blocks with clear descriptions
- Mock Firebase services in tests

## Development Workflow

1. **Spec**: Check `specs/implementation_plan.md`
2. **Model**: Add types to `src/types/models.ts`
3. **Service**: Implement in `src/services/`
4. **Hook**: Expose via `src/hooks/`
5. **UI**: Build components/pages
6. **Test**: Write tests, run `npm test`
7. **Verify**: `npm run build` must pass, check at 375px mobile width

## Critical Rules

- ✅ Always use `@/` path alias for imports
- ✅ Always clean up Firebase subscriptions in useEffect
- ✅ Always run `npm run build` after changes
- ✅ Always test at mobile width (375px)
- ❌ No Firestore calls in components (use services)
- ❌ No `any` types without justification
- ❌ No physical directional CSS (left/right) - use logical properties

## Git Workflow

Use conventional commits:
```
feat: add task voting system
fix: resolve calendar RTL alignment
test: add tests for RequestCard
refactor: extract date utilities
```

## Definition of Done

- [ ] `npm run build` passes without errors
- [ ] `npm test` passes
- [ ] Works at 375px mobile width
- [ ] RTL layout looks correct
- [ ] No console errors/warnings
- [ ] Committed with meaningful message

---
> Source: [solo3/BigMama](https://github.com/solo3/BigMama) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
