## crawbot

> CrawBot is an Electron desktop app — GUI for OpenClaw AI agents. Connects to OpenClaw Gateway (child process) over WebSocket JSON-RPC.

# CLAUDE.md

## Project Overview

CrawBot is an Electron desktop app — GUI for OpenClaw AI agents. Connects to OpenClaw Gateway (child process) over WebSocket JSON-RPC.

Reference sources: OpenClaw at `~/openclaw`, Claude Code CLI at `~/claude-code`.

## Commands

```bash
pnpm dev               # Dev mode with hot reload (Vite + Electron)
pnpm build             # Full production build (Vite → bundle OpenClaw → electron-builder)
pnpm build:vite        # Vite build only (no packaging)
pnpm lint              # ESLint with auto-fix
pnpm typecheck         # tsc --noEmit
pnpm test              # Vitest unit tests
pnpm test:e2e          # Playwright E2E
```

Single test: `pnpm vitest run tests/unit/stores.test.ts`

## Architecture

```
Main Process (electron/) ──IPC──▶ Renderer (src/, React 19 + Zustand + Tailwind)
       │                                    │
       │ manages lifecycle                  │ WebSocket JSON-RPC
       ▼                                    ▼
  OpenClaw Gateway (port 18789) ◀───────────┘
```

### Key directories

- `electron/main/` — Window, menu, tray, IPC handlers, updater
- `electron/gateway/` — Gateway lifecycle, WebSocket client, protocol types
- `electron/utils/` — Config, paths, storage, provider registry, UV/Python setup
- `src/pages/` — Setup, Dashboard, Chat, Channels, Skills, Cron, Settings
- `src/stores/` — Zustand stores (chat.ts is the largest — streaming, sessions, RPC)
- `src/components/ui/` — shadcn/ui (Radix + CVA + tailwind-merge)
- `src/i18n/locales/` — `en/`, `zh/`, `ja/`, `vi/`

Path aliases: `@/*` → `src/*`, `@electron/*` → `electron/*`

Ports: dev `5173`, gateway `18789`, production `23333`

## Code Style

- **Prettier**: single quotes, semicolons, 2-space indent, 100 char width, ES5 trailing commas
- **ESLint**: TypeScript strict, React hooks. Unused vars OK with `_` prefix
- **TypeScript**: strict mode, `noUnusedLocals`, `noUnusedParameters`

## Key Patterns

- **IPC**: All renderer↔main via preload bridge. Add channels in `electron/preload/index.ts`, handlers in `electron/main/ipc-handlers.ts`
- **Gateway**: JSON-RPC over WebSocket with exponential backoff. Types in `electron/gateway/protocol.ts`
- **i18n**: All user-facing strings through i18next
- **UI**: shadcn/ui + dark mode via Tailwind class strategy

## OpenClaw Patches

`electron/gateway/openclaw-patches-preload.cjs` — runtime in-memory patches via ESM loader hooks. This is the most complex file in the project.

Key patches:
- **H patch**: Rewrites "OpenClaw"→"Claude Code" in system prompt (Anthropic OAuth signature bypass)
- **I6 patch**: Renames `sessions_*` tool names (another Anthropic signature bypass)
- **D1 patch**: OAuth token refresh scope fix
- **Other patches**: Dispatch, relay, screenshot, stream-flush, session-affinity, subagents-rpc

After editing patches: source file is `electron/gateway/openclaw-patches-preload.cjs`. Dev mode loads it directly from source (`manager.ts:701`). Packaged build copies it via `electron-builder.yml` to `resources/openclaw-patches-preload.cjs`. No vite copy step involved — `pnpm build:vite` is NOT required after patch edits in dev mode.

After OpenClaw upgrades: verify patch FIND strings still match — see `docs/openclaw-patch-maintenance.md`.

Full OAuth bypass research: `docs/anthropic-oauth-bypass-research.md`.

## Private Docs (git submodule)

`docs/` is a submodule → private repo `Neurons-AI/crawbot-docs`. Commit separately:

```bash
cd docs/ && git add -A && git commit -m "update" && git push && cd ..
git add docs && git commit -m "chore: update docs submodule" && git push
```

New machine: `git clone --recurse-submodules <url>` or `git submodule update --init`

---
> Source: [Neurons-AI/crawbot](https://github.com/Neurons-AI/crawbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
