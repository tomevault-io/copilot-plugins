## type-imports-separate-line

> Type imports must use a separate import line; do not mix type and value in one statement.


# Type Imports on Separate Line

Type-only imports must use a **separate** `import type { ... }` statement. Do not declare types inline with value imports.

## Do

- Use a dedicated line for type imports from the same module as value imports:

```typescript
import { DataSource } from 'typeorm';
import type { DataSourceOptions } from 'typeorm';
```

- Use `import type { X } from '...'` when the symbol is only used in type positions.

## Don't

- Do not mix type and value in one import: `import { DataSource, type DataSourceOptions } from 'typeorm'`.
- Do not use inline `type` in a value import statement.

## Enforcement

ESLint `@typescript-eslint/consistent-type-imports` with `fixStyle: 'separate-type-imports'` enforces this. Import order (groups and styles last) is also enforced by ESLint and fixed by the same `npm run lint:fix` command.

---
> Source: [podverse/podverse](https://github.com/podverse/podverse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-05 -->
