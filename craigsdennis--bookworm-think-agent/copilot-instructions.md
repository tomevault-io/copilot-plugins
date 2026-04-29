## bookworm-think-agent

> This file gives repo-specific guidance to coding agents working in this project.

# BookWorm Agent Notes

This file gives repo-specific guidance to coding agents working in this project.

## Purpose

BookWorm is a Cloudflare Think demo. Keep changes aligned with that goal:

- showcase Think session memory
- showcase long-running agent behavior
- showcase extensions
- keep the UX readable for demos and screenshots

## Commands

Use these commands from the repo root:

```bash
npm install
npm run types
npm run build
npm start
npm run deploy
```

## Critical Constraints

### Callable methods

Do not remove `agents/vite` from `vite.config.ts`.

Reason: `@callable()` depends on the TC39 decorator transform provided by `agents/vite`. If this plugin is missing, client RPC calls such as `initializeOwner()` will silently break at runtime.

### Reader memory

The Think `memory` context is backed by a custom provider, not the default auto-backed provider.

Source of truth:

- `/bookworm-reader-memory.json`

If you change reader memory behavior, keep these paths aligned:

- `configureSession(...).withContext("memory", { provider: ... })`
- `getReaderMemory()`
- `replaceReaderMemory()`

### Extension contexts

When installing extensions dynamically, register any declared context blocks into the live session immediately.

Relevant code:

- `install_extension`
- `registerExtensionContexts()`

If you skip this, extension `host.setContext()` calls can fail with readonly/permission errors until restart.

### UI state sync

Be careful with React effects that derive sidebar state from streaming messages.

The client already guards against repeated state writes to avoid React nested update errors. Prefer idempotent updates and equality checks when adding new sync behavior.

## Current Architecture

### Server

`src/server.ts`

- `BookWormAgent extends Think<Env>`
- workspace backed by DO SQLite + R2
- bookshelf state lives in agent state
- reader profile lives in Think `memory`
- Open Library is used for canonical links and cover images
- scheduling uses agent schedule APIs
- extensions use `LOADER` and `HostBridgeLoopback`

### Client

`src/client.tsx`

- home page captures reader name
- `useAgent()` connects to the reader-specific agent
- `useAgentChat()` drives the conversation UI
- Markdown rendering uses `react-markdown` + `remark-gfm`
- completed tool outputs are collapsed with `<details>`

## Preferred Change Style

- Make the smallest correct change
- Preserve the demo-friendly UI tone and visual style
- Keep new tools explicit and well named
- Avoid adding extra abstractions unless a pattern is repeating

## Good Future Improvements

- richer reader profile UI for memory inspection
- better reminder composer UI instead of chat-only setup
- bundled sample extensions for demos
- code splitting in the client bundle if bundle size becomes a problem

## Deployment Notes

- Cloudflare config is in `wrangler.jsonc`
- current deployment target uses account `51e28e9a83197a9f12a2e39f9477ba4e`
- production URL: `https://bookworm.craigsdemos.workers.dev`

---
> Source: [craigsdennis/bookworm-think-agent](https://github.com/craigsdennis/bookworm-think-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
