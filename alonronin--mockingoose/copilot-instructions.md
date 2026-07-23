## mockingoose

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

Mockingoose is a utility for mocking Mongoose models in tests (Jest or Vitest). It intercepts Mongoose operations (find, save, aggregate, etc.) via mock functions so tests run without a database connection.

## Commands

- **Test:** `bun run test` (all tests) or `npx vitest run --testPathPattern=<pattern>` (single test)
- **Build:** `bun run build` (tsdown: `src/` → `dist/` ESM+CJS)
- **Typecheck:** `bun run typecheck` (tsc --noEmit)
- **Lint:** `bun run lint` (Prettier check)
- **Format:** `bunx prettier --write src`

## Architecture

TypeScript library: `src/index.ts` + `src/types.ts` → compiled to `dist/` via tsdown (ESM + CJS dual output with generated `.d.ts`).

### How It Works

1. **Connection mocking** — `mongoose.connect` and `mongoose.createConnection` are replaced with mock function stubs
2. **Operation mocking** — All query operations (`find`, `findOne`, `save`, etc.) on `mongoose.Query.prototype` are replaced with mocks that call `mockedReturn()`
3. **`mockedReturn()`** — Core function that looks up mocked data from `mockingoose.__mocks[modelName][op]`, wraps results in Mongoose Model instances (unless the op is a raw-return type like `deleteOne`/`countDocuments`), and supports promise patterns
4. **Instance methods** — `save`, `$save` are mocked on `mongoose.Model.prototype` with pre/post hook support
5. **Aggregate** — Separately handled via `mongoose.Aggregate.prototype.exec`
6. **Proxy API** — `mockingoose` is a Proxy that allows both `mockingoose(Model).toReturn(data, op)` (preferred) and `mockingoose.ModelName.toReturn(data, op)` (deprecated)
7. **Framework detection** — Runtime detection of `vi` (Vitest) or `jest` globals for mock function creation

### Key Files

- `src/index.ts` — Main library source
- `src/types.ts` — Exported TypeScript types (Op, MockController, Mockingoose, etc.)
- `__tests__/index.test.ts` — Test suite (Vitest)
- `__tests__/User.ts` — Test fixture model

### Style

Prettier with single quotes, trailing commas (es5). See `.prettierrc.json`.

## Skills

Use these Claude Code skills when working in this repo:

- `typescript-docs` — TypeScript documentation generation
- `typescript-advanced-types` — advanced TypeScript type patterns (generics, conditional types, mapped types, etc.)
- `vitest` — testing (writing tests, mocking, coverage, fixtures)
- `mongoose-mongodb` — Mongoose/MongoDB schema design, CRUD, queries
- `github-actions-templates` — GitHub Actions CI/CD workflows, automated testing/building/deploying

## Package Manager

- Always use `bun` as the package manager.
- Never add packages manually to package.json — always use `bun add` / `bun add -d`.
- Always use `bun` to install dependencies.

---
> Source: [alonronin/mockingoose](https://github.com/alonronin/mockingoose) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
