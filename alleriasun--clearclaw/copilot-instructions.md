## clearclaw

> Transparent relay daemon: chat channels (Telegram, Slack) ↔ Claude Code CLI. Routes interactions without duplicating CLI functionality.

# ClearClaw

Transparent relay daemon: chat channels (Telegram, Slack) ↔ Claude Code CLI. Routes interactions without duplicating CLI functionality.

## Quick Reference

```bash
npm start          # Run (requires env vars)
npm run dev        # Local dev (tsx --watch, auto-restarts)
npm run dev:relay  # Remote dev (nodemon watches dist/, explicit build)
npm run build      # tsc → dist/ (triggers dev:relay restart)
npm run check      # tsc --noEmit (type check only)
```

**Setup:** `clearclaw setup` saves tokens/default engine and approves DM pairing. Environment overrides remain supported: channel token(s) + `ALLOWED_USER_IDS` (comma-separated, channel-prefixed, e.g. `tg:12345,slack:U67890`).
**Channel:** `TELEGRAM_BOT_TOKEN` for Telegram, or `SLACK_BOT_TOKEN` + `SLACK_APP_TOKEN` for Slack (one channel at a time; Slack takes priority if both are set)
**Optional:** `PERMISSION_MODE` (default|acceptEdits|bypassPermissions|plan|dontAsk), `CLEARCLAW_HOME` (defaults to `~/.clearclaw`)

## Dev Server

**`npm run dev`** — standard local development. `tsx --watch` runs TypeScript directly, restarts on file changes. Use this when developing from desktop with a terminal.

**`npm run dev:relay`** — remote development via Telegram. `nodemon` watches `dist/` and restarts only when `npm run build` produces new output. Builds are explicit, not automatic. This is critical for ClearClaw-through-ClearClaw development where edits are approved one at a time over unpredictable intervals — an auto-rebuilding watcher would restart the server mid-batch, killing the Telegram connection.

## Verification

Before reporting a behavior change as done, prove it with the `verify` skill (`.claude/skills/verify/`). It runs an isolated daemon with real engines behind a recording channel, which you drive like a chat user. Pair it with `npm run check` and `npm test`. Never verify with `npm start`/`npm run dev`: they connect the owner's real bot, which is already served by their running daemon.

## Conventions

- All imports use `.js` extension (required by NodeNext module resolution, even for `.ts` source files)
- Interfaces defined in `types.ts`, implementations in their own files
- Data types use `interface`/`type` + plain objects. Classes only for Channel and Engine implementations
- ClearClaw is a relay that adds orchestration on top of CLI agents. It owns prompt assembly (framework + user instruction files), workspace creation/binding, and permission relay UX. The CLI owns tool execution, settings, and session management. When in doubt about where logic belongs: in the CLI, not here.
- **Design specs** go in `docs/specs/<topic>.md`. Use an ADR-style structure: Status, Context, Decision, Consequences, and relevant alternatives or references. Keep contracts and rationale durable; distinguish accepted decisions from proposed work and supersede decisions explicitly when they change. Update or consolidate an existing topic instead of adding dated phase snapshots. No step-by-step implementation recipes or completed-work logs.
- When developing through ClearClaw (remote via Telegram/Slack), large file writes will fail because the permission prompt content exceeds chat message limits. Break writes into smaller chunks: create/touch the file first, then append sections via Edit.

## Docs

- `docs/ARCHITECTURE.md` — Concepts, file structure, data/permission flows, interfaces, storage, config
- `docs/TASKS.md` — Backlog (phased). Check this first when asked about tasks, the backlog, or what's on the list. Update it when a task is completed or its status changes.
- `docs/OVERVIEW.md` — Strategy, rationale, what ClearClaw is and isn't
- `docs/specs/` — Durable topic specs (contracts, constraints, decisions, rationale)

---
> Source: [alleriasun/clearclaw](https://github.com/alleriasun/clearclaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
