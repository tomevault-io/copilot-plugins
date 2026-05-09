## vibeframe

> VibeFrame — AI-native video editing. CLI-first, MCP-ready.

# CLAUDE.md

VibeFrame — AI-native video editing. CLI-first, MCP-ready.

## Commands

```bash
pnpm install && pnpm build    # Setup
pnpm test                     # Run all tests
pnpm lint                     # Lint (0 errors policy)
pnpm -F @vibeframe/cli test   # Single package
```

## Architecture

```
CLI (Commander.js + Agent)  →  Engine (Project state)  →  Core (Zustand + FFmpeg)  →  AI Providers
```

Monorepo: Turborepo + pnpm workspaces. ESM. TypeScript strict mode.

Packages: `packages/cli`, `packages/core`, `packages/ai-providers`, `packages/mcp-server`, `packages/ui`, `apps/web`

## Conventions

- Conventional commits: `feat:`, `fix:`, `docs:`, `refactor:`, `chore:`
- API keys in `.env` (copy `.env.example`). See **[MODELS.md](MODELS.md)** for provider details.
- CHANGELOG auto-generated: `git-cliff --tag vX.Y.Z -o CHANGELOG.md`

## Skills & Agents

- `/test`, `/release`, `/sync-check`
- `code-reviewer`, `version-checker`, `lint-fixer`, `e2e-tester`, `feature-tester`, `pipeline-tester`

---
> Source: [vericontext/vibeframe](https://github.com/vericontext/vibeframe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-26 -->
