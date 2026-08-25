## samwise-2

> You're working inside **samwise-2**: Matt's personal web app for chatting

# AGENTS.md — samwise-2

You're working inside **samwise-2**: Matt's personal web app for chatting
with a coding-agent CLI (Claude Code today, Codex pending) from any device.
Read this before doing anything. Then read `CLAUDE.md` for the deeper
architecture notes — most of the same content lives there with more detail.

## Startup rules for this repo

1. Read this file.
2. Read `CLAUDE.md` second.
3. Apply both before doing other work.

## What this app is

A React/Vite frontend ("the Reading Room" — parchment palette, EB Garamond)
that talks over WebSocket to a Node/Express server. The server holds one
**persistent** `claude -p` child process per `(companion, repo)` pair, fed
JSON over stdin and reading stream-json events from stdout. Sessions persist
across browser closes, devices, and server restarts. Auto-starts on the Mac
Mini at boot via launchd; reachable from any device on the Tailscale tailnet.

Distinct from **Samwise** (the autonomous queue at `~/samwise/Personal-Apps/Samwise`).
This one is **manual chat only**. Don't add kanbans, state machines, or
closeout pipelines here.

## Tree (short version — see CLAUDE.md for the long one)

```
samwise-2/
├── src/                      React + Vite + TypeScript frontend
├── server/                   Node + Express + ws backend
├── scripts/                  start.sh, install/uninstall launchd, plist
└── README.md / CLAUDE.md / AGENTS.md
```

## Run

Dev (two terminals):
```
cd server && npm run dev          # :8090 backend
npm run dev                       # :5173 Vite, proxies /api → :8090
```

Production (autostarts on the Mini):
```
./scripts/install-launchd.sh
```
Then: `http://localhost:8090` or `http://<tailscale-name>:8090`.

## Where things live

- **Persistent CLI process** — `server/src/runner.ts` (`ClaudeSession` class
  + manager Map keyed by `${cli}|${cwd}`)
- **WS protocol** — `server/src/index.ts`
- **Repo discovery** — `server/src/repos.ts` (scans `~/code`,
  `~/samwise/Personal-Apps`, `~/Documents/PERSONAL-PROJECTS` + pinned
  `ASSISTANT-HUB`)
- **Chronicle** — `server/src/chronicle.ts` (reads
  `~/.claude/projects/*.jsonl`)
- **Session id persistence** — `server/src/sessions.ts` →
  `~/.samwise-2/sessions.json`
- **Stream-json reducer** — `src/hooks/useChat.ts` (must stay pure under
  React Strict Mode; correlation via `turnId` + `cbIndex` on the blocks
  themselves, not via an out-of-band Map)
- **Threshold view (landing)** — `src/components/desktop/Threshold.tsx`,
  `src/components/mobile/Mobile.tsx`
- **Conversation view** — `src/components/desktop/Conversation.tsx`,
  `src/components/mobile/Mobile.tsx`

## Key invariants — do not break

1. **One persistent process per `(cli, repo)`.** Per-turn spawn was the v1;
   we deliberately moved off it (2-3s startup tax per message). Keep the
   process warm.
2. **The reducer in `useChat.ts` must be pure.** No out-of-band mutable
   state. If you need per-turn correlation, put it on the blocks.
3. **Silent-resume-failure recovery.** If `--resume <id>` makes claude exit
   before init, the manager retries once without `--resume`. Don't strip
   this — it's the bug class that bit us when the session file was missing.
4. **Tailscale is the only auth boundary.** Don't add tokens or login flows.
5. **No databases, no kanbans, no closeout pipelines.** That's the other
   Samwise. This is just chat.

## Conventions

- No em dashes in user-facing strings (use `,` `.` `()` instead). Em dashes
  in code comments are fine.
- Inline styles with CSS variables from `src/tokens.css`. No Tailwind, no
  CSS-in-JS library.
- Stay terse. Personal app, not a public product.
- Always `npx tsc --noEmit` after meaningful edits.
- For UI changes, run `npm run dev` and verify with the Playwright MCP if
  available.

## Open work

1. **Codex companion** — stubbed. `runner.ts` throws for `cli === 'codex'`.
   Needs its own session class because `codex exec --json` events are shaped
   differently from claude stream-json (`thread.started`, `item.started`,
   `item.completed` with `agent_message` and `command_execution` items).
   Probably spawn-per-turn since codex doesn't have a stdin-streaming mode
   I could find.
2. **Tailscale serve** for an HTTPS hostname instead of `:8090`.
3. **Notifications** when a long turn finishes (Telegram bot via
   `assistant-mcp`).

---
> Source: [mrchevyceleb/samwise-2](https://github.com/mrchevyceleb/samwise-2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
