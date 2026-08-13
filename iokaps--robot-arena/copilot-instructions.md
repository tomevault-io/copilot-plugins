## robot-arena

> Real-time collaborative game application template using Kokimoki SDK.

# AGENTS.md

## Project Overview

Real-time collaborative game application template using Kokimoki SDK.
Has three display modes: `host` (desktop control), `player` (mobile gameplay), `presenter` (spectator screen).

## Build and test commands

- `npm install` - Install all dependencies (Node v22+)
- `npm run dev` - Start development server with HMR
- `npm run build` - TypeScript check + production build
- `npm run lint` - Run ESLint on TypeScript files
- `npm run format` - Format code with Prettier

## Project instructions

**IMPORTANT** Before implementing features, read instruction files in `.github/instructions/`:

- [host-starter-template](.github/instructions/host-starter-template.instructions.md) - Project starter template structure, display modes, state management, layouts, configuration
- [kokimoki-sdk](.github/instructions/kokimoki-sdk.instructions.md) - Kokimoki SDK providing state management, media uploads, server time, leaderboards, AI integration
- [dynamic-stores](.github/instructions/dynamic-stores.instructions.md) - Room-based state management with dynamic stores
- [kokimoki-shared](.github/instructions/kokimoki-shared.instructions.md) - Reusable UI components
- [react](.github/instructions/react.instructions.md) - React and TypeScript best practices

## Code Style Guidelines

- Named exports only (no default exports)
- Use `@/` path alias for all imports
- Use `kebab-case` for filenames except `src/utils/` and `src/hooks/` (use `camelCase` there)

## Common Tasks

- Check `@kokimoki/shared` components before building new UI
- Run `npm run build` after modifying `src/config/schema.ts` to generate corresponding YAML schema
- Always check and update `spec.md` before/after implementing features

## Security Considerations

- **DO NOT** modify `src/kit/` directory

---
> Source: [iokaps/robot-arena](https://github.com/iokaps/robot-arena) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
