## typescript

> - Add TSDoc style documentation when writing functions, methods and components.


## Typescript Development Guidelines:

### Documentation
   - Add TSDoc style documentation when writing functions, methods and components.

### Single Responsibility
    - SRP is a core principle of our codebase and should be followed in all functions, methods and components.
     => if the function, method or component is more than one responsibility, suggest how to break it down into smaller functions, methods or components. Encourage the user to use the "Single Responsibility" principle
     and explain the benefits of doing so in this case.

### Imports
   - Always use named imports for React components and hooks.
   - Never use non-destructured imports, such as `React.useMemo` or `React.useCallback`.

### Type Safety
   - Never use the `any` type.

---
> Source: [langwatch/scenario](https://github.com/langwatch/scenario) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
