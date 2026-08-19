## agent-team

> This repository is a DeepSeek Harness Web plugin written in TypeScript and React.

# Repository Guidelines

## Project Structure & Module Organization

This repository is a DeepSeek Harness Web plugin written in TypeScript and React.

- `src/client/` contains the injected React UI, client API, state, and CSS Module styles.
- `src/domain/` defines Zod schemas, domain types, and errors.
- `src/service/`, `src/runtime/`, `src/storage/`, and `src/transport/` implement team operations, Agent lifecycle, persistence, and Web transport.
- `tests/*.spec.ts` contains Vitest unit and integration-style tests.
- `docs/` records requirements, architecture decisions, feasibility constraints, and implementation status.
- `scripts/check-architecture.mjs` enforces plugin boundaries. Build output is generated in `lib/`.

Do not modify DeepSeek Harness source code. Integrate only through its documented plugin APIs, Slots, services, and semantic design tokens.

## Build, Test, and Development Commands

- `npm install` installs dependencies and runs the package preparation build.
- `npm run build` bundles server and client entries with `tsdown`.
- `npm run typecheck` runs TypeScript without emitting files.
- `npm test` runs all Vitest tests once.
- `npm run test:watch` reruns tests during development.
- `npm run guard:architecture` checks prohibited cross-boundary access.
- `npm run check` runs the architecture guard, typecheck, tests, and production build; run it before every commit.
- `npx @deepseek-ai/dsh web` starts Harness locally after this plugin is installed or linked into the active Web profile.

## Coding Style & Naming Conventions

Use two-space indentation, single quotes, and no semicolons, matching existing TypeScript. Prefer explicit return types on exported or asynchronous APIs. Use `PascalCase` for React components and types, `camelCase` for functions and variables, and descriptive CSS Module names such as `memberTileLeader`. Use Harness `--dsw-*` semantic variables instead of custom color systems.

## Testing Guidelines

Vitest is the test framework. Name tests `*.spec.ts` and place them in `tests/`. Add regression coverage for service mutations, persistence migrations, event projection, and client request behavior. No numeric coverage threshold is configured; changed behavior must have focused assertions. Always finish with `npm run check`.

## Commit & Pull Request Guidelines

Follow the repository’s Conventional Commit style: `feat:`, `fix:`, `refactor:`, `style:`, `test:`, or `docs:` followed by an imperative summary. Keep commits scoped and avoid generated or unrelated changes. Pull requests should explain user-visible behavior, architecture impact, and verification performed; link relevant issues and include before/after screenshots for UI changes. Call out schema migrations, Harness API assumptions, and known limitations explicitly.

---
> Source: [limuyang2/agent-team](https://github.com/limuyang2/agent-team) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-19 -->
