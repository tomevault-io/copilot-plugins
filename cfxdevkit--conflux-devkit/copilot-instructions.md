## conflux-devkit

> This is a TypeScript monorepo for Conflux blockchain development using:

# Cursor Rules for Conflux DevKit

## Project Overview
This is a TypeScript monorepo for Conflux blockchain development using:
- pnpm workspaces
- Turbo for build orchestration
- Biome for linting/formatting
- React + Vite for frontend
- Express for API server
- TypeScript throughout

## Code Style
- Use TypeScript strict mode
- Prefer const over let
- Use template literals for string interpolation
- Organize imports automatically
- Use Biome for formatting and linting
- Follow React best practices with hooks

## File Organization
- Keep components in packages/*/src/components/
- Keep utilities in packages/*/src/utils/
- Keep types in packages/*/src/types/
- Use index.ts files for clean exports

## Testing
- Write unit tests with Vitest
- Test files should be co-located with source
- Use descriptive test names
- Mock external dependencies

## Monorepo Structure
- Each package has its own package.json
- Use workspace:* for internal dependencies
- Build packages in dependency order
- Use Turbo for task orchestration

## Development Workflow
- Run `pnpm install` after cloning
- Use `pnpm run dev` for development
- Use `pnpm run build` to build all packages
- Use `pnpm run test` to run all tests
- Use `pnpm run check:fix` for linting/formatting

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cfxdevkit) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
