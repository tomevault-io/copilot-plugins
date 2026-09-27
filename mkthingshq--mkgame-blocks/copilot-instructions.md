## mkgame-blocks

> Super Blocks is a standalone block puzzle game built with TanStack Start,

# AGENTS.md

## Project

Super Blocks is a standalone block puzzle game built with TanStack Start,
React 19, and Cloudflare Workers.

## Commands

```bash
pnpm dev
pnpm check
pnpm test
pnpm e2e
pnpm build
pnpm run deploy
```

Deployment credentials are local CLI credentials only. They are not runtime
Worker secrets and must not be uploaded with `wrangler secret put`.

## Architecture

- `src/components/game/`: game UI and browser-facing game modules
- `src/routes/`: TanStack Router file routes
- `src/lib/haptics.ts`: browser haptic compatibility helper
- `public/game/audio/`: documented learning/research audio assets
- `tests/unit/`: pure game-engine tests
- `tests/e2e/`: Playwright user journeys

The route tree is generated at `src/routeTree.gen.ts`; never edit it manually.

## Constraints

- Keep the Worker stateless.
- Avoid Node.js-only runtime APIs.
- Preserve deterministic daily boards.
- Keep local-storage and session-storage schemas versioned.
- Update Playwright tests for user-visible behavior changes.
- Do not expose credentials in logs or committed files.

---
> Source: [MkThingsHQ/mkgame-blocks](https://github.com/MkThingsHQ/mkgame-blocks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-23 -->
