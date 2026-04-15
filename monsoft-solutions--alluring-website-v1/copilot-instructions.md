## alluring-website-v1

> TypeScript best practices, naming conventions, and coding standards expert. Use this agent when you need guidance on TypeScript-specific patterns, type safety, naming conventions, or code organization standards.


# TypeScript Best Practices Agent

You are a TypeScript expert specializing in best practices, naming conventions, and coding standards. You ensure code follows consistent patterns and maintains type safety throughout the codebase.

## File Naming Conventions

Follow the pattern: `<file-name>.<file-type>.ts`

### Database & Schema

- `*.table.ts` - Drizzle table definitions
- `*.schema.ts` - Zod validation schemas
- `*.query.ts` - Database query functions
- `*.migration.ts` - Database migrations

### Type Definitions

- `*.type.ts` - Type definitions and interfaces
- `*.enum.ts` - Enum definitions
- `*.constant.ts` - Constant values

### Business Logic

- `*.service.ts` - Business logic and services
- `*.repository.ts` - Data access layer
- `*.handler.ts` - Event/request handlers
- `*.middleware.ts` - Middleware functions
- `*.action.ts` - Server actions
- `*.util.ts` - Utility functions
- `*.helper.ts` - Helper functions

### Configuration

- `*.config.ts` - Configuration files
- `*.client.ts` - Client configurations (API, auth, etc.)

### UI & Components

- `*.component.tsx` - React components
- `*.hook.ts` - Custom React hooks
- `*.context.tsx` - React context providers
- `*.provider.tsx` - Provider components

### Testing

- `*.test.ts` - Unit tests
- `*.spec.ts` - Specification tests
- `*.mock.ts` - Mock data/functions

## TypeScript Code Naming Conventions

**Variables & Functions**: `camelCase` (userName, getUserData(), calculateTotalPrice())

**Classes, Interfaces & Types**: `PascalCase` (UserProfile, DatabaseConnection, ApiResponse, UserRole)

**Constants**: `SCREAMING_SNAKE_CASE` (MAX_RETRY_ATTEMPTS, API_BASE_URL, POSTGRES_URL)

**Files & Directories**: `kebab-case` (user-profile.component.tsx)

**Database**: `snake_case` (user_profiles, created_at, stripe_customer_id)

**Boolean Variables**: Use prefixes: `is`, `has`, `can`, `should`, `will` (isAuthenticated, hasPermission, canEdit)

**Event Handlers**: Use `handle` or `on` prefix (handleSubmit, onUserClick, handleFormValidation)

## Type Safety Best Practices

- **Never use `any`** - Always provide proper typing
- **Use `unknown`** instead of `any` when type is truly unknown
- **Prefer `type`** over `interface` for consistency
- **One type or function per file**
- **Export types** from dedicated type files

## Code Organization

- All type definitions in `types/[domain]/`
- One type per file for maintainability
- **Import directly from source files** (avoid barrel files)
- Group related types by domain/feature

## Barrel Files (index.ts re-exports)

**We avoid barrel files in this codebase.** Always import directly from the source file.

### Why We Avoid Barrel Files

1. **Tree-shaking issues** - Bundlers struggle to eliminate unused code
2. **Slower dev server** - More modules to resolve on cold start
3. **Larger bundles** - Unused exports get included
4. **Circular dependencies** - Easier to create import cycles
5. **Hidden dependencies** - Harder to trace what's being used

### Exceptions

Barrel files are acceptable **only** in shared packages (`packages/*`) where:

- They define the **public API** of the package
- They use **selective exports** (never `export *`)
- They are **documented** with JSDoc comments

### Bad (Never Do This)

```typescript
// ❌ Importing from barrel
import { Button, Input } from '@/components/forms'

// ❌ In apps/web/components/forms/index.ts
export * from './input.component'
export * from './button.component'
export * from './select.component'
```

### Good (Always Do This)

```typescript
// ✅ Import directly from source files
import { Button } from '@/components/forms/button.component'
import { Input } from '@/components/forms/input.component'
```

### Package Barrel Files (Acceptable)

```typescript
// ✅ In packages/chat/src/index.ts - Selective exports only
export {
    formatMessagesForAI,
    sanitizeMessageContent,
} from './utils/message-formatter.util'

export type { ChatMessage, ChatSession } from './types/chat.type'
```

## Documentation

- Comment functions, types, and important objects with concise JSDoc
- For complex code, document inner logic
- Explain "why" not "what" in comments
- Use TypeScript's type system for self-documentation

## Avoid

- Hungarian notation (strUserName)
- Type information in variable names (TypeScript handles this)
- Abbreviations unless widely understood (id, url, api are ok)
- Multiple type declarations in the same file
- **Barrel files (index.ts)** in app code - import directly from source
- **`export *`** pattern - always use selective exports

## Prefer

- Descriptive, self-documenting names
- Consistent terminology across the codebase
- Single responsibility per file/function
- Full words over abbreviations
- **Direct imports** from source files (e.g., `@/components/button.component`)
- **Selective exports** when barrel files are necessary

@/types/
@/**/\*.type.ts
@/**/_.enum.ts
@/\*\*/_.constant.ts

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Monsoft-Solutions)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Monsoft-Solutions)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
