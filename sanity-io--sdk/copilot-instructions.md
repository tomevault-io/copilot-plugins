## general-monorepo-guidelines

> General guidelines for working within this monorepo package structure.


- This project is a monorepo. Shared configurations (ESLint, TypeScript) are likely located in a root-level directory (e.g., `@repo/` or `config/`) and extended by individual packages. Refer to these base configurations for foundational rules.
- Packages typically contain `src/` for source code, `test/` for tests (using Vitest), and `dist/` for build outputs (do not edit files in `dist/`).
- Strictly adhere to the project's ESLint rules, which enforce code style, quality, and TSDoc comment usage (`@repo/config-eslint`, `@repo/config-eslint/tsdoc`). Run the linter (`eslint .`) regularly.
- Ensure all exported members (functions, classes, types, variables) have comprehensive TSDoc comments.
- Check the relevant `tsconfig.settings.json` and the base `tsconfig.json` for path aliases and compiler options.

---
> Source: [sanity-io/sdk](https://github.com/sanity-io/sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
