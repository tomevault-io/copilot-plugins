## typescript-standards

> TypeScript Standards and Conventions

- Always use TypeScript for all new code and migrations
- Enforce strict type checking with `strict: true` in tsconfig
- Use explicit return types on functions and methods
- Prefer interfaces over type aliases for object definitions
- Use enums for fixed sets of values
- Leverage TypeScript's utility types when appropriate
- Use named exports rather than default exports for better tree-shaking
- Define reusable types in dedicated type files within the types/ directory
- Use React.FC<Props> for component typing with explicit props interface
- Avoid using 'any' type; use 'unknown' when type is truly uncertain
- Define parameter and return types for all async functions
- Use type guards for narrowing types when appropriate

---
> Source: [sushaantu/boxento](https://github.com/sushaantu/boxento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
