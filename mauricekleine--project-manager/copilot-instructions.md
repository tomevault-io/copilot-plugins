## structure

> - Prefer `type` over `interface` for type definitions

# TypeScript Preferences

- Prefer `type` over `interface` for type definitions
- Use `type` for object shapes, union types, and computed types
- Only use `interface` when you need declaration merging or extending from classes

# Module Organization

- Don't use barrel exports (index.ts files that re-export everything)
- Import directly from the source files instead of through index files
- Keep imports explicit and traceable

# File Naming Convention

- Always use kebab-case for filenames
- Examples: `user-profile.tsx`, `api-client.ts`, `blood-test-results.component.tsx`
- Avoid camelCase, PascalCase, or snake_case for file names

# Additional Best Practices

- Use descriptive and meaningful file names
- Keep file names concise but clear about their purpose
- Maintain consistency across the entire codebase

# Code styles

- Ignore ESLint warning - they will most likely be auto-fixed later
- Prefer `function myFunction() {}` over `const myFunction = () => {}`

# Package Manager

- Use Bun for everything

# Data

- No need to run migrations or push new schemas - we will do it manually

# Design

- Tailwind is used for designs together with ShadcnUI components and style defaults

---
> Source: [mauricekleine/project-manager](https://github.com/mauricekleine/project-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
