## all-vibes-opencode-test

> - **Build app:** `npx nx build opencode-test`

# AGENTS.md

## Build, Lint, and Test Commands

- **Build app:** `npx nx build opencode-test`
- **Serve app:** `npx nx serve opencode-test`
- **Lint:** `npx nx lint opencode-test`
- **Test (all):** `npx nx test opencode-test`
- **Test (single file):** `npx nx test opencode-test --testFile=path/to/file.spec.ts`
- **E2E tests:** `npx nx e2e opencode-test-e2e`
- **Run all:** `npx nx run-many -t build test lint`

## Code Style Guidelines

- **Formatting:** Prettier (`.prettierrc`) enforces single quotes.
- **EditorConfig:** 2-space indent, UTF-8, trim trailing whitespace, insert final newline.
- **Imports:** Use ES6 imports; group by external, then internal.
- **Types:** Prefer explicit types and interfaces; use TypeScript strictness.
- **Naming:** Classes/Components: `PascalCase`; variables/functions: `camelCase`.
- **Angular:** Component selectors: `app-` prefix, kebab-case; directive selectors: `app`, camelCase.
- **Error Handling:** Use Angular's error listeners; avoid silent failures.
- **Linting:** Follows Nx/Angular ESLint rules, including module boundaries.
- **File Structure:** Keep code modular; use Nx generators for new libs/apps.
- **VSCode Extensions:** Recommended: Angular Console, Prettier, ESLint, Jest Runner, Playwright.

## Libraries

- **ag-grid-angular** and **Angular Material** are installed and ready for use.
- **NgRx:** Not yet compatible with Angular 20 (as of July 2025). Use Angular signals, RxJS, or alternative state management until official support is released. Check https://github.com/ngrx/platform/issues for updates.

---
> Source: [ronlawrence3/all-vibes-opencode-test](https://github.com/ronlawrence3/all-vibes-opencode-test) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-18 -->
