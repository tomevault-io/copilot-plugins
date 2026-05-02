## typescript

> TypeScript conventions for this ESM project


# TypeScript Conventions

- **ESM only**: This project uses `"type": "module"`. All imports must use `.js` extensions:
  ```typescript
  import { parseMeta } from './renderer.js';  // correct
  import { parseMeta } from './renderer';     // wrong
  ```
- **Strict mode**: `tsconfig.json` has `"strict": true`. Do not weaken it.
- **Explicit types on public APIs**: Exported functions must have explicit parameter and return types.
- **Avoid `any`**: Use `unknown` + type narrowing. `any` is allowed only with a justifying comment.
- **Interfaces over type aliases** for object shapes (prefer `interface Foo {}` over `type Foo = {}`).
- **No default exports**: Use named exports for better refactoring support.

---
> Source: [collectioneur/readme-aura](https://github.com/collectioneur/readme-aura) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-23 -->
