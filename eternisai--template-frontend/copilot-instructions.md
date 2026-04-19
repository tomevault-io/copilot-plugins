## template-frontend

> - ✅ Always use `export default function ComponentName() {}`

# Cursor AI Rules for TanStack Start Project

## Key Rules to Remember

- ✅ Always use `export default function ComponentName() {}`
- ✅ Use `@/` imports, never relative `../`
- ✅ Organize features in `src/features/feature-name/`
- ✅ No `any` types, prefer `unknown`
- ✅ Alphabetical import ordering
- ✅ Use pnpm only, never npm/yarn

## Project Structure

```
src/
├── features/
│   ├── shared/
│   │   └── components/
│   └── feature-name/
│       ├── index.ts
│       ├── types.ts      # Feature types (directly in root)
│       ├── components/
│       ├── hooks/        # Includes context providers
│       ├── lib/          # Configuration and utilities
│       └── utils/
├── routes/
├── test/
└── styles.css
```

## 🚨 MANDATORY Code Quality Workflow

**ALWAYS run these commands after making significant changes:**

```bash
pnpm typecheck  # ✅ CRITICAL - Check TypeScript errors
pnpm lint       # ✅ CRITICAL - Fix code quality issues
pnpm test       # ✅ CRITICAL - Ensure tests pass
```

**Pre-commit hooks enforce these automatically, but run manually during development!**

## Code Style Guidelines

### React Components

- Use functional components with TypeScript
- Export components as default exports
- Use proper TypeScript types, avoid `any`
- Organize imports alphabetically

### Feature Export Pattern

Each feature must have an `index.ts` file that exports all public modules:

```typescript
// src/features/feature-name/index.ts
export * from './components'
export * from './hooks'
export * from './types'
export * from './utils'
export { default as FeatureProvider } from './hooks/FeatureProvider'
```

This enables clean imports: `import { useFeature, FeatureComponent } from '@/features/feature-name'`

### Imports

- Use absolute imports with `@/` prefix
- Never use relative imports like `../`
- Group imports: external libraries first, then internal modules

### File Naming

- Components: PascalCase (e.g., `Header.tsx`)
- Hooks: camelCase starting with `use` (e.g., `useAuth.ts`)
- Utils: camelCase (e.g., `formatDate.ts`)
- Types: camelCase with `.types.ts` suffix

### TypeScript

- Always provide explicit types for function parameters and returns
- Use `unknown` instead of `any`
- Prefer interfaces for object types
- Use type assertions sparingly

### Testing

- Test files in `__tests__` folders or `.test.tsx` suffix
- Use Vitest with React Testing Library
- Test user interactions, not implementation details

### Styling

- Use Tailwind CSS v4 with utility classes
- Single quotes for all strings (className, imports, etc.)
- No trailing commas
- 100 character line width
- Use semantic class names when needed

## Package Management

- Always use `pnpm` for package management
- Never use `npm` or `yarn`
- Keep dependencies up to date

## Git Workflow

- Use conventional commits
- Pre-commit hooks will run linting and formatting
- Fix all TypeScript errors before committing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EternisAI) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
