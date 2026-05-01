## testing

> Test rules for openapi-to-cli


## Test Writing Rules

### General principles

1. Tests live in the `tests/` directory inside `openapi-to-cli`.
2. Each test must be independent; use mocks for HTTP (axios) and the filesystem when needed.
3. `describe` and `it` names should clearly describe the behavior under test.

### Test structure

- Test files: `*.test.ts`.
- Grouping by modules, for example: `describe("profile-store", ...)`, `describe("openapi-loader", ...)` etc.

### Running tests

From the `openapi-to-cli` directory:

```bash
npm test
```

---
> Source: [EvilFreelancer/openapi-to-cli](https://github.com/EvilFreelancer/openapi-to-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-20 -->
