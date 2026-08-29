## contract-first-api

> - This repo is `contract-first-api`, a TypeScript library for defining one shared API contract and deriving server and client code from it. The library is primarily designed to be used in a monorepo with shared, server and client packages.

# AGENTS.md

### Overview
- This repo is `contract-first-api`, a TypeScript library for defining one shared API contract and deriving server and client code from it. The library is primarily designed to be used in a monorepo with shared, server and client packages.
- The library consists of 3 main packages:
  - `@contract-first-api/core` - The core library that has the contract definition DSL, api client and openapi generation logic.
  - `@contract-first-api/express` - An express server adapter that currently also bundles generic reusable server code and websocket support via `ws`.
  - `@contract-first-api/react-query` - A react-query client adapter that is a thin wrapper around the core client and tanstack/react-query. It is designed to be used in a React frontend project.
- The project is not stable and does not have meaningful adoption yet. Breaking changes and broad public API refactors are expected before this fact changes and don't need to consider backward compatibility unless explicitly stated.
- Public API changes must keep all user-facing docs in sync: root/package READMEs, `.agents/skills/contract-first-api` skill files, and the `example` workspace.
- The skill files are mainly consumer guidance for people using this library from other projects not a guidance on how to work on the library itself. Do not treat them as only local contributor notes. Do not keep re-reading the skill file(s) once you already know how the library works.

### Commands
- `pnpm run build:packages` - Packages depend on each other through declaration files. After changing exported package APIs, run this to refresh dependents or when something appears stale or fails mysteriously.
- `pnpm run typecheck` - Run declaration builds and workspace typechecking.
- `pnpm run lint` - Run Biome to fix formatting and import ordering automatically. Don't spend time manually fixing mechanical formatting/import issues. Focus on real typecheck/test failures instead that cannot be fixed deterministically.
- `pnpm run test` - Run tests.
- `pnpm run check` - Runs `lint`, `typecheck`, and `test` in sequence. Use this as the broad all-in-one pass.

### Verification and reporting
- Run the smallest useful verification for the change. Prefer package-scoped tests/typechecks when a change is limited to one package.
- Do not run extra commands just to be ceremonially thorough. In particular, if `pnpm run lint` only applies Biome's safe formatting/import fixes after tests or typechecks have already passed, do not rerun tests or typechecks solely because of those safe fixes.
- Run `pnpm run build:packages` after exported package API changes so dependent package declaration builds are refreshed. This is separate from formatting and does not imply that tests need to be rerun afterward.
- Report command results that are relevant to the task or confidence in the change. Do not mention unrelated warnings, suggestions, or pre-existing issues unless they block the requested work, change the interpretation of verification, or require a user decision.
- If a command produces both relevant failures and unrelated warnings, summarize only the relevant failure and say unrelated output was omitted if that avoids confusion.

---
> Source: [simoalanne/contract-first-api](https://github.com/simoalanne/contract-first-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
