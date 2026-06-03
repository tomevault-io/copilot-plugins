## react-package-conventions

> Conventions and guidelines specific to the `packages/react` package.


- Follow all `general-monorepo-guidelines`.
- This package contains React components and hooks. Adhere to React best practices and the specific React linting rules (`@repo/config-eslint/react`).
- Prefer functional components and Hooks over class components.
- Tests utilize Vitest with `@testing-library/react`. Use `@testing-library/jest-dom` matchers for DOM assertions. Ensure components, hooks, and interactions are well-tested.

---
> Source: [sanity-io/sdk](https://github.com/sanity-io/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
