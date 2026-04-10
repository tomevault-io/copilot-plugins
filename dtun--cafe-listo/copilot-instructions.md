## cafe-listo

> Emergency preparedness checklist app. pnpm monorepo with TypeScript (strict mode, ESM).

# Cafe Listo

Emergency preparedness checklist app. pnpm monorepo with TypeScript (strict mode, ESM).

## Packages

- `packages/shared` — Types and seed data, consumed by all packages
- `packages/landing` — Vite + React landing page
- `packages/mobile` — Expo 52 / React Native mobile app
- `packages/mcp-server` — MCP server

## Scripts

| Command             | Description                        |
| ------------------- | ---------------------------------- |
| `pnpm test`         | Run all test suites                |
| `pnpm typecheck`    | `tsc --noEmit` across all packages |
| `pnpm lint`         | ESLint                             |
| `pnpm lint:fix`     | ESLint with auto-fix               |
| `pnpm format`       | Prettier write                     |
| `pnpm format:check` | Prettier check                     |

## Conventions

- Always run `pnpm lint:fix` after making changes
- Always run `pnpm format` after making changes
- Run `pnpm typecheck` before committing
- Tests: Jest for shared/mobile, Vitest for landing. Tests live in `__tests__/` co-located with source.
- Conventional Commits for all commit messages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dtun)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/dtun)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
