## caleb-video-editor

> This file is for AI coding agents working on this repository (Claude Code reads it through `CLAUDE.md`). People: start with [README.md](README.md).

# Storyboard — notes for coding agents

This file is for AI coding agents working on this repository (Claude Code reads it through `CLAUDE.md`). People: start with [README.md](README.md).

If you are editing a **video project** under `projects/`, follow `projects/CLAUDE.md` instead (the app writes it when it starts); this file is about developing the Storyboard app itself.

## Safety rules (always)

- **Install dependencies with `npm ci --ignore-scripts` only.** Never run `npm install` to set up or repair the project, and never run `npm ci`/`npm install` without `--ignore-scripts`: package install scripts execute arbitrary code on the user's machine, and `npm install` can pull versions that aren't in `package-lock.json`. The repo's `.npmrc` sets `ignore-scripts=true` as a backstop, not as a replacement for the flag.
- Add or upgrade a dependency only when the user asks, with `npm install --ignore-scripts <package>@<version>`, and show them the `package-lock.json` diff.
- Don't use `npx` to fetch packages that aren't already in `node_modules`; run project tools through the `npm run` scripts.
- Never stop or restart processes you didn't start. The user may have their own Storyboard running.

## Set up a fresh clone

```bash
npm ci --ignore-scripts   # dependencies, exactly as pinned in package-lock.json; install scripts never run
npm run setup             # downloads the headless Chromium used for frames and renders (once; on Linux: npm run setup -- --with-deps)
npm run doctor            # checks Node, ffmpeg, Chromium, the Claude Code login and the port; exits 1 while something is missing
npm run dev               # long-running server → http://127.0.0.1:5199 (start it in the background)
```

- `npm run doctor` prints the fix for each problem. System tools (Node.js 22.12+, ffmpeg, Claude Code) are the user's machine: tell them what's missing and install it only if they ask you to.
- Only the user can log in to Claude Code. If doctor reports "not logged in", ask them to run `claude` in a terminal and use `/login`.
- If port 5199 is taken, doctor says whether Storyboard is already running there; run yours with `PORT=5299 npm run dev`.
- It works when `curl -s http://127.0.0.1:5199/api/projects` lists the "Example: Storyboard teaser" project.
- The music engine is optional and large (about 20 GB of models, 9–16 GB of memory while running). Set it up or start it only when the user asks (README → "Music engine").

## Run & check

- `npm run dev` starts one process: Vite (editor + scene frames) in middleware mode, the REST API (`/api`), SSE (`/api/events`) and the MCP server (`/mcp`). Server code isn't hot-reloaded; restart after editing `server/`.
- `npm run typecheck` (TypeScript 7, covers `src/`, `server/` and project scenes), `npm test` (node:test) and `npm run format` (Prettier, width 130; CI runs `npm run format:check`).
- `npm run analyze -- <audio>` runs the music analyzer on a file.

## Map

- `src/runtime/` — the `storyboard` module scenes import (easing, interpolate, spring, color, random, music grid, components). Keep it deterministic: frames are rendered out of order.
- `src/frame/main.tsx` — `frame.html`: loads a project's scenes, renders any time, exposes `window.__sb` (`src/shared/frameApi.ts`). Used by the editor, thumbnails, agent captures and renders alike.
- `src/editor/` — React UI; state in `store.ts` (zustand), server events in `events.ts`.
- `src/shared/types.ts` — shapes shared by server, editor and runtime.
- `server/index.ts` wires everything; `projects.ts` (files on disk, code generations), `vite.ts`, `capture.ts` (Playwright), `render.ts` (ffmpeg), `seams.ts`, `mcp.ts` (tools + scope enforcement), `chat.ts` (turns, undo, streaming to the UI), `agents/` (provider interface, Claude Code headless provider, prompts), `music/` (beat/bar/phrase analysis; `engine.ts` watches the ACE-Step engine and is its REST client, `engine-cli.ts` is `npm run music`, `library.ts` stores takes, `service.ts` runs generation jobs and describes takes for the agent), `templates.ts` (starter scene, art direction, and the scene guide written to `projects/CLAUDE.md`), `doctor.ts` (`npm run doctor`).

## Conventions

- Scene code changes reach frames through `ProjectStore.syncCode` → Vite module invalidation → a bumped `codeGeneration` → frames re-import. Don't rely on Vite HMR for files under `projects/`.
- The in-app agent runs with `--restricted`, which skips CLAUDE.md discovery, so the scene guide is inlined in its system prompt (`agents/prompts.ts`). Keep `SCENE_GUIDE` in `templates.ts` the single source.
- Tool names the agent sees are `mcp__storyboard__<tool>`; scene chats may only use the tools in `SCENE_TOOLS` (`chat.ts`) and the MCP server re-checks scope from the `X-Storyboard-*` headers.
- Music tools are registered in `mcp.ts` only while `MusicEngine.isReady()` and never for scene chats; the per-turn context says whether the engine runs. Storyboard never starts or stops the engine itself.
- `/api` and `/mcp` accept only local Host headers and same-origin browser requests (`server/index.ts`); requests from other websites could otherwise start agent turns. Keep new endpoints behind that guard.
- Only `projects/example/` is tracked by git; every other project folder is the user's own and ignored.

---
> Source: [saeedvaziry/caleb-video-editor](https://github.com/saeedvaziry/caleb-video-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
