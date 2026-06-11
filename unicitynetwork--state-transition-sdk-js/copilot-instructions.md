## state-transition-sdk-js

> - Build: `npm run build`

# State Transition SDK Development Guide

## Commands
- Build: `npm run build`
- Type check: `npm run build:check`
- Lint: `npm run lint`
- Lint with auto-fix: `npm run lint:fix`
- Run all tests: `npm run test`
- Run a single test: `npm run test:single tests/path/to/test.ts -t "test description"`

## Code Style Guidelines
- Use TypeScript strict mode with explicit types
- Interfaces must be prefixed with "I" and use PascalCase (e.g., `ISerializable`)
- Static readonly variables must use UPPER_CASE
- Import order: builtin → external → internal with alphabetical sorting
- Always provide explicit function return types
- Explicit member accessibility required (public/private/protected)
- Error handling: Use specific error messages and proper error types
- Async/await is preferred over raw promises
- Sort object keys alphabetically where there are 2+ keys
- Use `.js` extension in imports (ES modules)
- Thorough input validation in constructors and factory methods
- Tests use Jest with descriptive test cases and appropriate timeouts

---
> Source: [unicitynetwork/state-transition-sdk-js](https://github.com/unicitynetwork/state-transition-sdk-js) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-11 -->
