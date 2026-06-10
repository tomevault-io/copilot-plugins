## type-imports

> Use type keyword for type-only imports


# Type Imports

Applies to all TypeScript files. Use the `type` keyword for imports that are only used in type positions.

### Type-Only Imports

- Use `type` keyword for type-only imports to improve bundle size and clarity.

- ✅ Good:
  ```typescript
  import { type ReactNode, type FC, useState } from 'react'
  import type { CreateDealTaskSchema } from '@/schemas/tasks'
  import type { ComponentPropsWithoutRef } from 'react'
  ```

- ❌ Bad:
  ```typescript
  import { ReactNode, FC, useState } from 'react'
  import { CreateDealTaskSchema } from '@/schemas/tasks'
  ```

### When to Use Type Imports

- Use `type` for:
  - Type-only imports (interfaces, types, type parameters)
  - Imports that are only used in type positions
- Do not use `type` for:
  - Runtime values (functions, classes, constants)
  - Mixed imports (combine type and value imports on same line)

---
> Source: [rcrdk/rcrdk.dev](https://github.com/rcrdk/rcrdk.dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-10 -->
