## rubber-duckie

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Rubber Duckie (package name: `rubber-duckie`) — a macOS Electron app with two faces:

1. **Duckie spotlight** — an always-on-top liquid-glass panel (the visible app). Type a task and Claude drives the real mouse/keyboard via the `mac_input` native addon (AX snapshots + CGEvents), narrating progress in the HUD.
2. **Task scheduler** — persistent prompts that run through the `claude` CLI unattended on a schedule, with per-run NDJSON logs, reports, diffs, and a session chat. Autopilot rules can auto-queue tasks from GitHub/Jira/Trello/Asana.

Everything executes through the Claude Code CLI — there is no direct API integration and no API key.

## Commands

```bash
yarn dev              # Electron app with hot reload
yarn build            # Typecheck + electron-vite build
yarn build:mac        # macOS distributable
yarn test             # vitest run (tests/ — single file: yarn vitest run tests/scheduler.test.ts)
yarn lint             # ESLint with cache
yarn typecheck        # typecheck:node + typecheck:web
```

Use **yarn**, not npm.

## Architecture

- `src/main/index.ts` — lean bootstrap: dotenv (`.dev.vars` then `.env`), windows, tray, runtime, IPC, startup gate.
- `src/main/app/` — hidden main window + `broadcast()` (windows.ts), scheduler/automation singletons (runtime.ts), tray + menu (tray.ts).
- `src/main/ipc/` — all `ipcMain` handlers, one module per domain (claude, onboarding, tasks, computer, integrations, logs, chat, misc).
- `src/main/claude/` — CLI plumbing: `bin.ts` (PATH-safe binary resolution — GUI apps miss `~/.local/bin`; every spawn must use `claudeBin()` + `spawnEnv()`), `check.ts`/`status.ts` (install + `claude auth status --json`), `login.ts` (in-app `claude auth login`), `install.ts` (in-app `curl | bash` installer), `runner.ts` (detached scheduled runs, stream-json logs), `enhance.ts`, `terminal.ts`.
- `src/main/onboarding/` — standalone liquid-glass onboarding window + macOS permission checks (Accessibility via the native addon; deep-links to System Settings panes).
- `src/main/computer/` — the perceive→act loop (`loop.ts` orchestrator + split modules), glass HUD windows (`overlay.ts` barrel over `hud/`), native addon wrapper (`native.ts`).
- `src/main/agents/` — `AgentScheduler` (barrel `scheduler.ts` over scheduler-core/run-lifecycle/session-runs/…), persistence to `agents.json`.
- `src/main/integrations/`, `src/main/automations/` — provider OAuth/tasks + Autopilot engine.
- `src/preload/` — `api.ts` (the whole `window.api` surface) + `index.ts` bridge; types in `index.d.ts`.
- `src/renderer/src/` — React 19, plain co-located CSS (no Tailwind). `main.tsx` routes `#onboarding` to the onboarding window; everything else renders `App`. The main timeline window currently stays hidden — the duckie panel and onboarding window are the visible surfaces.
- `src/shared/` — types + parsers shared by main/renderer (stream-log, run-report, schedule, claude, permissions, computer).
- `native/src/` — `mac_input.mm` (AX capture, CGEvent input, accessibility checks) and `glass_sidebar.mm` (`NSGlassEffectView` panels). Build output in `native/build/` is gitignored.

## Startup / onboarding flow

On launch: Accessibility granted + onboarding previously completed → show the duckie immediately, then background-verify the Claude CLI (signed out/uninstalled reopens onboarding). Otherwise the centered glass onboarding window runs: permissions → install (in-app installer) → sign in → test ping → duckie.

Dev simulation without touching real state: `DUCKIE_FAKE_CLAUDE=missing|logged-out`, `DUCKIE_FAKE_PERMS=denied` (see `.dev.vars.example`).

## Config & environment

- Secrets/env: `.dev.vars` (gitignored; `.env` still read as fallback). Template: `.dev.vars.example`.
- Runtime config `userData/config.json`; tasks `userData/agents.json`; run logs `userData/task-runs/`.
- Computer-use knobs: `DUCKIE_COMPUTER_MODEL`, `DUCKIE_COMPUTER_ESCALATE_MODEL`, `DUCKIE_COMPUTER_EFFORT`, `DUCKIE_COMPUTER_DEBUG=1`.

## Code style

- Prettier: single quotes, no semicolons, 100 char width, no trailing commas. 2-space indent, LF.
- Keep files small (~120 lines) and split by responsibility; original paths stay as barrels so imports never break.
- Comments only for non-obvious constraints (macOS gotchas, GC workarounds) — no narration.

---
> Source: [AlexandrosGounis/rubber-duckie](https://github.com/AlexandrosGounis/rubber-duckie) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
