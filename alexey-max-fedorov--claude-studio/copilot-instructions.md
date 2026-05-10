## claude-studio

> Monorepo with packages in `packages/`:

# Claude Studio

Monorepo with packages in `packages/`:
- `shared` — Message protocol types shared between server and extension
- `server` — Node.js bridge server (WebSocket + Claude Agent SDK)
- `extension` — Plasmo browser extension (element picker, prompt widget, side panel)
- `plugin` — Claude Code plugin (visual-edit skill, post-edit formatting hook)

## Conventions
- pnpm workspaces, TypeScript strict mode everywhere
- Tests with vitest, run via `pnpm -r test`
- Extension dev: `pnpm --filter @claude-studio/extension dev`
- Server dev: `pnpm --filter @claude-studio/server dev`

---
> Source: [alexey-max-fedorov/claude-studio](https://github.com/alexey-max-fedorov/claude-studio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
