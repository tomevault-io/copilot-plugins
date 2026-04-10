## couchcms-ai-toolkit

> Auto-load refactoring rules for TypeScript files


# Refactoring Rules for TypeScript

## 🚨 CRITICAL: Refactoring Workflow

**When a refactoring request is made (e.g., "refactor @file.ts"), you MUST:**

1. **First**: Load and follow `@ai-toolkit-shared/prompts/refactoring/router.md`
2. **Router will**: Analyze the file, select appropriate specialist(s), request confirmation
3. **Then**: Apply the rules below as part of the selected specialist workflow

**Never skip the router analysis step - it ensures the correct specialist is selected.**

---

When refactoring this file, apply these rules:

## Type Safety

- ❌ Never use `any` type (use `unknown` if truly unknown)
- ✅ Define interfaces for all data structures
- ✅ Use `import type` for type-only imports
- ✅ Enable strict mode in tsconfig

## Imports

- ❌ Never use barrel file imports: `from './components'`
- ❌ Never use index re-exports: `from './index'`
- ✅ Always use direct imports: `from './components/button'`

## Naming Conventions

- Files: `kebab-case.ts` (e.g., `video-player.ts`)
- Variables: `camelCase` (e.g., `videoPlayer`)
- Interfaces/Types: `PascalCase` (e.g., `VideoPlayer`)

## Alpine.js Integration

- Export functions for Alpine.js use on `window`
- Keep Alpine components simple, complex logic in TypeScript
- Define types for Alpine component data

## Error Handling

- Use Result types for expected errors
- Use try/catch for unexpected errors
- Provide meaningful error messages

## Alpine.js Integration

- ✅ Export functions for Alpine.js use on `window`
- ✅ Keep Alpine components simple, complex logic in TypeScript
- ✅ Define types for Alpine component data
- ✅ Use `alpine:init` event for initialization
- ✅ Register functions globally: `window.functionName = functionName`
- ✅ Use Result types for expected errors
- ✅ Provide typed interfaces for component state

## Error Handling

- ✅ Use Result types for expected errors: `Result<T, E>`
- ✅ Use try/catch for unexpected errors
- ✅ Provide meaningful error messages
- ✅ Use type guards for runtime validation
- ✅ Exhaustive switch statements for union types

## Generic Patterns

- ✅ Use generic functions for reusable components
- ✅ Define constraints for generic types: `<T extends { id: string }>`
- ✅ Use `Omit`, `Pick`, `Partial` utility types
- ✅ Create type-safe event emitters with event maps

## Validation

- ✅ Create validation schemas with typed rules
- ✅ Use type guards for runtime type checking
- ✅ Validate external data with Zod or similar
- ✅ Return typed validation results

## API Patterns

- ✅ Define `ApiResponse<T>` interface for API calls
- ✅ Use typed fetch functions
- ✅ Handle errors explicitly (no silent failures)
- ✅ Use async/await with proper error handling

## Checklist

- [ ] No `any` types (use `unknown` if truly unknown)
- [ ] Direct imports only (no barrel files)
- [ ] Proper naming conventions (camelCase, PascalCase, kebab-case)
- [ ] Type-only imports use `import type`
- [ ] Error handling implemented (Result types or try/catch)
- [ ] Alpine integration via window exports
- [ ] Complex Alpine logic extracted to TypeScript
- [ ] Type guards for runtime validation
- [ ] Generic patterns for reusable components
- [ ] Validation schemas for data validation
- [ ] API responses properly typed
- [ ] Exhaustive switch statements for union types

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/martijnbokma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
