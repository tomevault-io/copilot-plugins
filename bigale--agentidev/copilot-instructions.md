## agentidev

> <!-- Generated from packages/ai-context/sources/. Do not edit directly. -->

<!-- Generated from packages/ai-context/sources/. Do not edit directly. -->

---
description: Bridge WebSocket server, script management, sessions, scheduling
globs: packages/bridge/**
alwaysApply: false
---


# Bridge Server (`packages/bridge/server.mjs`)

WebSocket server on port 9876. Clients identify via `BRIDGE_IDENTIFY` with a role: `extension`, `script`, `cli`, or `claude`. Roles defined in `packages/bridge/protocol.mjs` (`ROLES` export).

## Message Protocol

All messages use envelope: `{ id, type, source, timestamp, payload }`. Build with `buildMessage()` / `buildReply()` from `protocol.mjs`. All types prefixed `BRIDGE_`.

Categories: connection (`IDENTIFY`, `HEALTH`, `ERROR`, `STATUS`), sessions (`SESSION_CREATE/DESTROY/LIST/CLEAN`), scripts (`SCRIPT_REGISTER/PROGRESS/COMPLETE/PAUSE/RESUME/CANCEL/LIST/LAUNCH`), debugger (`SCRIPT_CHECKPOINT/STEP/SET_BREAKPOINT`, `DBG_*`), scheduling (`SCHEDULE_CREATE/UPDATE/DELETE/LIST/TRIGGER/HISTORY`), files (`SCRIPT_FILE_CHANGED/SAVE`), AI (`SC_GENERATE_UI/SC_CLONE_PAGE`).

## Script Lifecycle

State machine: `registered` -> `running` -> `checkpoint`/`paused` -> `complete`/`cancelled`/`killed`

- `BRIDGE_SCRIPT_LAUNCH`: server spawns child process via `launchScriptInternal()`, stores PID
- `BRIDGE_SCRIPT_REGISTER`: script self-registers with name, totalSteps, pid, checkpoints[]
- Force-kill: `BRIDGE_SCRIPT_CANCEL { force: true }` sends SIGTERM then SIGKILL after 2s

## Pending Maps Pattern

State held by PID between launch and register, transferred at `BRIDGE_SCRIPT_REGISTER`:
- `pendingBreakpoints` — breakpoints set before script connects (eliminates timing race)
- `pendingSessionLinks` — sessionId linked by PID, copied to `script.sessionId`
- `pendingInspectors` — V8 inspector WebSocket URLs by PID

## Playwright Shim (`packages/bridge/playwright-shim.mjs`)

Drop-in replacement for `import { chromium } from 'playwright'`. Auto-connects ScriptClient, wraps Page instances. Intercepts navigate/click/fill/wait/eval/screenshot — declares checkpoints as `p1:navigate`, `p1:click`, etc. Reads `BRIDGE_CDP_ENDPOINT` env var to connect to existing session browser.

## Session-Script 1:1 Linking

One active script per session. Server injects `BRIDGE_CDP_ENDPOINT` env var at launch. `BRIDGE_SESSION_DESTROY` cascades to cancel linked scripts.

## V8 Inspector Debugging

Scripts launched with `--inspect-brk=0`. Server connects via `inspector-client.mjs`. ESM quirk: use `runIfWaitingForDebugger()` not `resume()`. PID-based inspector lookup since scripts haven't registered at line 1.

## File Watcher

Watches `~/.agentidev/scripts/` with `fs.watch`, 300ms debounce, echo suppression via `fileWatcherIgnore` Set.

## Scheduling

Persisted to `~/.agentidev/schedules.json`. Server-side cron with overlap prevention.

## Script Launch CWD

Server creates `node_modules` symlink in scripts dir pointing to nearest `node_modules` from `originalPath`. Sets `cwd: dirname(originalPath)` on spawn.

## Artifacts

Screenshots captured at checkpoints, console buffer always collected, run archive saved on completion via `BRIDGE_SCRIPT_RUN_COMPLETE`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bigale) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
