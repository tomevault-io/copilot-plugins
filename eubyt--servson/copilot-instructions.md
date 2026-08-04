## servson

> This document outlines the project architecture, guidance on how AI can contribute, and strict development rules for maintaining code quality, test integrity, and documentation alignment.

# Agent Guidelines for Servson

This document outlines the project architecture, guidance on how AI can contribute, and strict development rules for maintaining code quality, test integrity, and documentation alignment.

---

## Project Overview

**Servson** is a lightweight, zero-config Mock REST API server built with [Bun](https://bun.sh) and [@tinyhttp/app](https://tinyhttp.v1k.eco/).
It allows developers to spin up a functional RESTful mock server in seconds using a single JSON configuration file (e.g., `mock.json`).

### Key Architecture Components

- **`src/index.ts`**: CLI entry point built with `commander`. Manages CLI arguments, `init` command, and server startup.
- **`src/mock-server.ts`**: Core mock engine handling RESTful routes (`GET`, `POST`, `PUT`, `DELETE`), path parameters, in-memory data state, identity generation, and response formatting.
- **`src/schema.ts`**: Schema URLs and default configuration defaults (`DEFAULT_MOCK_CONFIG`).
- **`src/types.ts`**: TypeScript definitions for `Settings`, `CrudResponses`, `Model`, `MockConfig`, and related types.
- **`src/pagination.ts`**: Utilities for handling response pagination logic.

---

## AI Contributions

AI agents can contribute to Servson by:
1. Implementing new REST features, custom HTTP header options, or server configuration flags.
2. Fixing bugs, optimizing endpoint performance, or handling edge cases in request routing and data manipulation.
3. Adding and maintaining tests to ensure high test coverage and system reliability.
4. Keeping project documentation, types, default schema values, and test fixtures in sync.

---

## Mandatory Development Rules

### 1. Test Creation for Every Feature
- Every new feature, endpoint behavior, or configuration setting **must** include corresponding unit or integration tests.
- Place tests in `src/mock-server.test.ts` or relevant `src/*.test.ts` files.

### 2. Prioritize Code Fixes Over Modifying Tests
- When a test fails after code changes, **prioritize debugging the source code** first.
- Do **not** modify existing test assertions or expectations just to make a failing test pass.
- Only update tests when there is an explicit, approved change to the expected behavior or contract.

### 3. Synchronize Schema, Types, and Documentation
- If you add or modify a configuration setting in `src/schema.ts` (`DEFAULT_MOCK_CONFIG`) or the `Settings` interface in `src/types.ts`:
  - You **must** update the example `mock.json` snippet in `README.md` to reflect the change.
  - Ensure type definitions (`src/types.ts`), default schema values (`src/schema.ts`), and documentation (`README.md`) remain completely aligned.

---

## Verification Checklist

Before completing any task or opening a pull request, run all of the following commands:

```bash
# 1. Run unit/integration tests
bun test

# 2. Run linter
bun run lint

# 3. Verify production build
bun run build
```

---
> Source: [eubyt/servson](https://github.com/eubyt/servson) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
