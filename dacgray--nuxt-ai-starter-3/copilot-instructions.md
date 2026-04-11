## nuxt-ai-starter-3

> As this project's AI coding tool, you must follow the additional conventions below, in addition to the built-in functions.

# Additional Conventions Beyond the Built-in Functions

As this project's AI coding tool, you must follow the additional conventions below, in addition to the built-in functions.

# Project Overview

## Tech Stack

- **Runtime**: Node.js >=22.0.0, npm ^11
- **Framework**: Nuxt ^4.3.1
- **Language**: TypeScript (via vue-tsc ^3.2.5)
- **UI**: @nuxt/ui ^4.5.1
- **Content**: @nuxt/content ^3.12.0
- **i18n**: @nuxtjs/i18n ^10.2.3
- **Database**: better-sqlite3 ^12.6.2
- **Testing**: Vitest ^4.0.18, @nuxt/test-utils ^4.0.0, @vue/test-utils ^2.4.6, Playwright ^1.58.2, happy-dom ^20.8.3
- **Linting**: @nuxt/eslint ^1.15.2
- **Formatting**: Prettier ^3.8.1

## General Guidelines

- Use TypeScript for all new code
- Follow consistent naming conventions
- Write self-documenting code with clear variable and function names
- Prefer composition over inheritance
- Use meaningful comments for complex business logic

## Code Style

- Use 2 spaces for indentation
- Use semicolons
- Use double quotes for strings
- Use trailing commas in multi-line objects and arrays

## Architecture Principles

- Organize code by feature, not by file type
- Keep related files close together
- Use dependency injection for better testability
- Implement proper error handling
- Follow single responsibility principle

## Post-Change Verification

After making any code changes, ALWAYS run the following checks in order:

1. **Format check**: `npm run pretty:check`
2. **Type check**: `npm run type:check`
3. **Lint**: `npm run lint`
4. **Tests**:
   - For any change: run tests for all files touched in the change
     - Unit/component tests: `npm run test:unit`
     - Nuxt integration tests: `npm run test:nuxt`
     - E2E tests: `npm run test:e2e`
   - For major changes (new features, refactors, dependency updates, architectural changes): run ALL tests: `npm run test`

All checks must pass before considering a change complete.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dacgray)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dacgray)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
