## vue3-flask-moviedb-ui

> - **dev** – Start dev server

# AGENTS.md

## Build / Lint / Test

- **dev** – Start dev server
  `npm run dev` (or `yarn dev`)

- **build** – Build production assets
  `npm run build`

- **preview** – Preview local build
  `npm run preview`

- **lint** – Run ESLint (if configured)
  `npx eslint . --ext .js,.vue`
  *(install ESLint first if missing: `npm i -D eslint`)*

- **test** – Run all Vitest tests
  `npx vitest`

- **test:one** – Run a single test file (replace `Path/To/Test.vue`)
  `npx vitest src/__tests__/Path/To/Test.vue`

## Code‑Style Guidelines

| Topic | Guideline |
|-------|-----------|
| **Imports** | Keep external imports at the top, sorted alphabetically. Local relative imports follow. Separate groups by a blank line. |
| **Formatting** | Use Prettier (installed via `npm i -D prettier`) – `npx prettier --write .`.  Two‑space indent, single quotes, trailing commas where possible. |
| **Type‑Safety** | Add JSDoc types or TypeScript (`*.ts`/`*.tsx`).  When using Vue 3, define component props/types in `script setup`. |
| **Naming** | - Components: PascalCase (`MoviePosterCard.vue`).  <br>- Variables/Props: camelCase.  <br>- Constants: UPPER_SNAKE_CASE. |
| **Error Handling** | Prefer `try…catch` around async API calls; log errors to console and surface user‑friendly messages using `alert` or Snackbar. |
| **Testing** | Write tests in `src/__tests__/` with the same relative path as the source file. Use `vitest` – snapshots for component rendering. |
| **Linting** | Enforce no‑undef, no‑unused-vars, and camelcase rules.  Add a `.eslintrc.cjs` if missing. |
| **Commit style** | Conventional commits (`feat:`, `fix:`, `refactor:`) – not mandatory but recommended. |

## Cursor / Copilot Rules

*(None currently found. Add rules in `.cursor/rules/` or `.cursorrules` if needed.)*

---

**Tip:** Run `npm run dev` then hit **Ctrl+C** to exit; repeat for preview/build. Use `npx vitest` to run all tests. For a single file, pass the path to the test as shown above.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/karthikappy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/karthikappy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
