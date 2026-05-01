## typescript-style

> This rule defines the TypeScript code style conventions for the project. Follow these rules for all TypeScript and React code.


# TypeScript Code Style Guide

This rule defines the TypeScript code style conventions for the project. Follow these rules for all TypeScript and React code.

## Parameter Passing

- **Always pass parameters as a single object** (named parameters pattern).

  **Example:**
  ```ts
  // Good
  function doSomething({ id, name }: { id: string; name: string }) { /* ... */ }
  // Bad
  function doSomething(id: string, name: string) { /* ... */ }
  ```

## Type Safety

- **Never use `any` as a type.** Use explicit types, interfaces, or `unknown` with type guards if necessary.
- **Reuse interfaces** and place them in a `models/` directory when shared across files.

  **Example:**
  ```ts
  // models/user.ts
  export interface User { id: string; name: string; }
  // Usage
  import { User } from '@/models/user';
  ```

## Imports

- **Use shorter imports** via path aliases (e.g., `@/components/...`), not relative paths like `../../components`.
- **Do not use index exports.** Use named exports and import modules directly.

  **Example:**
  ```ts
  // Good
  import { Button } from '@/components/shared/ui/button';
  // Bad
  import { Button } from '@/components/shared/ui';
  ```

## Functional Programming

- **Do not use classes.** Use functional methods and hooks.
- **Always wrap `if` statements in curly braces**, even for single-line blocks.

  **Example:**
  ```ts
  // Good
  if (isActive) {
    doSomething();
  }
  // Bad
  if (isActive) doSomething();
  ```

## Comments and Documentation

- **Do not comment obvious things.**
- **Do not explain changes in comments.**
- **Only document extraordinary changes or complex logic.**
- **Use JSDoc or a short description for top-level functions.**

## Forms and Schemas

- **Reuse schemas** for forms (e.g., Zod schemas) and validation logic. Place schemas in a `schemas/` directory if shared.

## Utility Functions

- **Place small utility functions under `utils/function-name.ts`**.

  **Example:**
  ```ts
  // utils/format-date.ts
  export function formatDate(date: Date): string { /* ... */ }
  ```

---
> Source: [PageAI-Pro/vibe-coding-starter](https://github.com/PageAI-Pro/vibe-coding-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
