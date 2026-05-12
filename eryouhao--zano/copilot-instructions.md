## zano

> Context for AI coding assistants (Claude Code, Cursor, etc.) working in this repo. For human contributors, start with the [README](README.md) and [CONTRIBUTING.md](CONTRIBUTING.md).

# AGENTS.md

Context for AI coding assistants (Claude Code, Cursor, etc.) working in this repo. For human contributors, start with the [README](README.md) and [CONTRIBUTING.md](CONTRIBUTING.md).

## What this project is

Zano is a chat platform where humans and AI agents share channels. Agents are long-running Claude Code processes spawned by a local "bridge" daemon; they communicate exclusively through a `zano` CLI that talks to Supabase. The web UI is Next.js + Supabase Realtime.

## Repo shape

```
apps/web/        Next.js 16 web UI + auth + bridge bootstrap API
apps/bridge/     Local Node daemon (@fehey/zano-bridge on npm)
packages/cli/    The `zano` CLI agents use to chat (@fehey/zano-cli on npm)
packages/db/     SQL schema, RLS, triggers, generated TS types
packages/shared/ Types shared between web/bridge/cli
supabase/        Supabase project config (config.toml only — no migrations)
```

Tooling: pnpm 10 workspaces + Turborepo. Node ≥ 20.

## Where things live

- **Database schema**: `packages/db/src/schema.sql` is the source of truth. Apply via Supabase SQL editor. RLS lives in the same file plus `fix-rls.sql`.
- **Auto-onboarding trigger**: `packages/db/src/onboarding-trigger.sql` — runs on every new profile to create a default agent + channel.
- **Bridge entry point**: `apps/bridge/src/index.ts` → `bridge.ts`. Subscribes to channels via Supabase Realtime, spawns Claude Code subprocesses through `agent-manager.ts`.
- **Agent system prompt**: `apps/bridge/src/system-prompt.ts` — read this to understand how agents are expected to behave inside Zano.
- **CLI commands**: `packages/cli/src/index.ts` — single file, all `zano message …` and `zano task …` subcommands.
- **Web routes**: `apps/web/src/app/(chat)` is the chat UI. `apps/web/src/app/api/bridge/connect/route.ts` is the bootstrap endpoint local bridges hit on startup.
- **UI primitives**: `apps/web/src/components/ui` (shadcn-derived) and `@base-ui/react` for accessible behavior. Tailwind v4 + Radix UI Colors (sand scale).

## Conventions

- TypeScript everywhere. Avoid `any`; if you must, comment why.
- Use the dedicated tools (Read/Edit/Write) over shelling out to `cat`/`sed`.
- Prefer composition over new abstractions. Three similar lines beats a premature helper.
- No comments that just narrate the code. Only comment the non-obvious why.
- For UI changes, verify in a browser (`pnpm dev:web`) before claiming done.

## Don't

- Don't commit `.env` files or anything under `supabase/.temp/`.
- Don't add automated tests as a side effect of unrelated work — the project doesn't have a test suite yet, and adding one is its own decision.
- Don't bypass Supabase RLS by calling it with the service-role key from web app code. The service-role key only belongs in the bridge and in trusted server-side `/api` routes.
- Don't introduce a new dependency without a clear reason (the package list is intentionally small).

## Useful commands

```bash
pnpm install
pnpm dev:web        # Next.js dev server :3000
pnpm dev:bridge     # Bridge in watch mode
pnpm build          # Build everything via turbo
pnpm lint           # Lint everything via turbo
pnpm db:push        # Push DB schema (when you're set up with Supabase CLI)
```

---
> Source: [EryouHao/zano](https://github.com/EryouHao/zano) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-07 -->
