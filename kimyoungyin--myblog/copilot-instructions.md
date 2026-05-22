## typescript-standards

> - **ALWAYS** use strict TypeScript mode (already configured in [tsconfig.json](mdc:tsconfig.json))


# TypeScript Standards (MANDATORY)

## 🔒 **Type Safety (MUST ENFORCE)**

- **ALWAYS** use strict TypeScript mode (already configured in [tsconfig.json](mdc:tsconfig.json))
- **NEVER** use `any` type - use proper interfaces or `unknown`
- **MUST** define interfaces for all data structures in [src/types/index.ts](mdc:src/types/index.ts)
- **ALWAYS** use Zod schemas for runtime validation (see [src/lib/schemas.ts](mdc:src/lib/schemas.ts))

## 📝 **Interface Definitions (CRITICAL)**

```typescript
// ✅ CORRECT - Use interfaces for object shapes
export interface User {
    id: string;
    email: string;
    is_admin: boolean;
}

// ❌ WRONG - Don't use type aliases for object shapes
export type User = {
    id: string;
    email: string;
    is_admin: boolean;
};
```

## 🚫 **FORBIDDEN TypeScript Practices**

- **NEVER** use `any` type
- **NEVER** use `@ts-ignore` or `@ts-nocheck`
- **NEVER** cast types without proper validation
- **NEVER** use `Object` or `Function` constructors
- **NEVER** create untyped arrays or objects

## ✅ **REQUIRED Practices**

- **ALWAYS** use proper return types for functions
- **ALWAYS** validate external data with Zod schemas
- **ALWAYS** use generic types when appropriate (e.g., `useDebounce<T>` from 'use-debounce')
- **ALWAYS** handle null/undefined cases explicitly
  description:
  globs:
  alwaysApply: true

---

---
> Source: [kimyoungyin/myblog](https://github.com/kimyoungyin/myblog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
