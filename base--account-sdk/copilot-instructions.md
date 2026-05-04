## account-sdk

> When running tests with npm, yarn, or vitest, ALWAYS include flags to prevent interactive/watch mode:

# Cursor AI Assistant Rules

## Test Execution Guidelines

### Always use non-interactive mode for tests
When running tests with npm, yarn, or vitest, ALWAYS include flags to prevent interactive/watch mode:

- For npm test: use `npm test -- --run`
- For yarn test: use `yarn test --run`
- For vitest directly: use `vitest run`
- For jest: use `jest --no-watch`

---
> Source: [base/account-sdk](https://github.com/base/account-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
