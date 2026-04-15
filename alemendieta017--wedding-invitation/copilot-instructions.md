## wedding-invitation

> - Always use TypeScript strict mode (enabled in [tsconfig.json](mdc:tsconfig.json))

# TypeScript Standards

## General Rules
- Always use TypeScript strict mode (enabled in [tsconfig.json](mdc:tsconfig.json))
- Use explicit types for function parameters and return values
- Prefer interfaces over types for object shapes
- Use `Readonly<T>` for props and immutable data

## React Component Patterns
- Use functional components with hooks
- Define prop interfaces with descriptive names
- Use `React.ReactNode` for children props
- Export components as default exports

## Path Aliases
- Use `@/*` to reference files from the `src/` directory
- Example: `import { Button } from '@/components/Button'`

## Type Definitions
- Create `.d.ts` files for external module declarations
- Use `declare module` for untyped packages
- Define global types in `src/types/` directory

## Best Practices
- Avoid `any` type - use `unknown` or proper typing
- Use union types for multiple possible values
- Leverage TypeScript's built-in utility types
- Enable strict null checks and use optional chaining
description:
globs:
alwaysApply: false
---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alemendieta017)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alemendieta017)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
