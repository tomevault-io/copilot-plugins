## auxxai-apps

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

Monorepo of first-party apps for the [Auxx.ai](https://auxx.ai) platform, built with `@auxx/sdk`. Each app lives in `apps/<app-slug>/` as a **self-contained project** with its own dependencies and package.json. There is no root package.json or monorepo orchestrator — work within individual app directories.

## Main Platform Codebase

The main Auxx.ai application (including the `@auxx/sdk` source) lives at `/Users/mklooth/Sites/auxxai`. Reference that repo when you need to understand SDK internals, platform APIs, or how apps integrate with the platform.

## Common Commands

All commands run from within an app directory (`apps/<app-slug>/`):

```bash
pnpm install              # install dependencies
pnpm run dev              # local dev server
pnpm lint                 # lint (Biome)
pnpm build                # production build
```

Scaffold a new app:
```bash
cd apps/
pnpm dlx @auxx/sdk init <app-slug>
```

## Code Style

Enforced by [Biome](https://biomejs.dev/) (config at repo root `biome.json`):
- Spaces, indent width 2, line width 100
- Single quotes for JS/TS
- Recommended linter rules enabled
- Import organizing enabled

## CI/CD

- **PRs to main**: CI detects which apps changed and runs `pnpm lint` + `pnpm build` per app
- **Merge to main**: Changed apps are auto-published via `pnpm exec auxx version create`
- Node 20, pnpm

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Auxx-Ai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Auxx-Ai)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
