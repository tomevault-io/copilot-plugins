## line-steer-reportal-fe

> - Use Prettier for all JS/TS/JSON/MD changes.


# Code Style Guides (Basic)

## Formatting

- **Run formatter**
  - Use Prettier for all JS/TS/JSON/MD changes.
- **Follow lint rules**
  - Fix ESLint errors/warnings instead of suppressing them.
  - Make sure indentation is consistent across files
  - Do not disable lint rules unless necessary
  - Don't disable lint rules or other automated checks

## TypeScript

- **Prefer types over `any`**
  - Use `unknown` + narrowing when you don’t know a value shape.
- **Keep types close to usage**
  - Define component prop types next to the component unless shared.
- **Avoid implicit `any`**
  - Type function parameters and return types when inference isn’t obvious.
- **Avoid using Non-Null Assertion Operator**
  - Do not use `!` operator unless necessary
- **Avoid strings directly as much as possible**
  - Use enums instead, especially if that string is being referenced in multiple places
- **Avoid using `as` keyword**

## React / Next.js (Web)

- **Components**
  - Prefer functional components.
  - Keep components small and focused; extract reusable UI into `src/common/components/`.
- **Routing (Pages Router)**
  - Use the Pages Router under `src/pages/`.
  - Use `next/router` APIs (`useRouter`) for navigation in Pages Router.
  - Avoid `next/navigation` in Pages Router code.
  - Only use `next/navigation` in App Router code (or modules that are explicitly App-only, e.g. `src/app-disabled/`).
  - Do not add `'use client'` directives unless you are working in the App Router.
- **Hooks**
  - Follow Rules of Hooks; keep hook calls at the top level.
- **Conditional rendering**
  - Prefer early returns for loading/error states.
- **Context**
  - Use a separate hook to create context value. Use return type of this hook to define context value type
- **Routing / project structure**
  - Keep pages and API routes in `src/pages/` (API routes in `src/pages/api/`).
  - Put cross-cutting concerns in `src/common/`:
    - `src/common/components/` for shared components
    - `src/common/layouts/` for shared page layouts
    - `src/common/context/` for context providers + hooks
    - `src/common/constants/` for constants/enums
    - `src/common/styles/` for global/theme styles
  - Put feature modules in `src/modules/`.
- **Apollo**
  - When using apollo query or mutation, use loading state returned by the hook instead of defining loading state

## Ant Design (antd)

- **Prefer antd over custom UI**
  - Prefer `antd` components for common UI patterns.
- **Ant Design reference docs**
  - When working on `antd` components, theming, or `@ant-design/cssinjs`, reference `docs/antd/llms.txt`.
  - Only reference `docs/antd/llms-full.txt` if you need deeper component API details.

## Naming

- **Files & folders**
  - Use `PascalCase` for React components (e.g. `LoginPage.tsx`) when applicable.
- **Variables & functions**
  - Use `camelCase` and descriptive names; avoid unclear abbreviations.
  - Use `UPPER_CASE` for constants and graphql queries/mutations.
  - Use descriptive names for variables and functions that explain what the variable is for
  - Avoid unclear abbreviations

## Imports

- **Ordering**
  - Group imports: external libraries, then internal modules, then relative paths.
- **No unused imports**
  - Remove unused imports/exports as part of the same change.

## General hygiene

- **Small PRs**
  - Keep changes scoped; don’t mix refactors with feature/bugfix unless required.
- **No dead code**
  - Delete unused functions/components instead of commenting them out.
- **Consistency**
  - Match surrounding code style in the file you’re editing.
- **Code quality**
  - Follow best practices
  - Write scalable code
  - Prefer descriptive easy to read code over succinct code to make code more readable
  - Do not duplicate code. Extract common logic into functions, components, or, custom hooks
  - Always confirm that code is not being duplicated
  - Avoid writing custom code for complex operations. Use external libraries instead. If library is not available, suggest a library to use
- **JavaScript style**
  - Avoid chaining ternary operator
  - Prefer `if` statements over ternary operator in JS code
  - Always use braces when using `if` conditions
- **Styling**
  - Avoid inline styles
  - Use tailwind as much as possible
  - Use `classNames` library and `classNames/bind` library to make code easier to manage and read
  - Use `antd` components for UI instead of building custom components
  - Prefer `antd` layout/spacing patterns (e.g. `Space`, `Flex`, `Row`/`Col`) over ad-hoc CSS

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/commutatus) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-13 -->
