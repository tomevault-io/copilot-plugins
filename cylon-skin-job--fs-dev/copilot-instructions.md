## fs-dev

> > Agent-focused orientation for Fusion Studio. This repository is the development checkout for the Fusion Studio desktop workspace app.

# AGENTS.md - Fusion Studio

> Agent-focused orientation for Fusion Studio. This repository is the development checkout for the Fusion Studio desktop workspace app.

## Current Identity

Fusion Studio is an Electron + React + Node.js workspace application for managing project folders, AI-assisted chat threads, wiki/docs/views, file exploration, and local system resources.

It is not a model provider. AI work is delegated to configured harnesses such as OpenCode or other CLI/service adapters. Fusion Studio owns the workspace shell, persistence, routing, rendering, and orchestration around those harnesses.

## Repository Layout

```
fs-dev/
├── fusion-studio-client/       # Electron shell + React/Vite renderer
│   ├── electron/               # Electron main process, preload, protocol, server spawn
│   └── src/                    # React UI, stores, WebSocket client, view mounting
├── fusion-studio-server/       # Node/Express/WebSocket backend
│   ├── server.js               # HTTP + WebSocket entry point
│   ├── lib/                    # db, workspace, thread, wire, harness, wiki, resources
│   └── data/                   # Runtime server data; contains fusion.db in dev
├── ai/                         # Development workspace content and current project wiki
├── System_Manager/             # Bundled/default system manager workspace files
├── docs/                       # Specs, handoffs, architecture notes
└── README.md                   # Human-facing project overview
```

## Active Code Paths

| Area | Path |
|------|------|
| Renderer UI | `fusion-studio-client/src/` |
| Electron main/preload | `fusion-studio-client/electron/` |
| Server | `fusion-studio-server/` |
| Server libraries | `fusion-studio-server/lib/` |
| Current Chat architecture source of truth | `ai/views/wiki-viewer/Wiki/007-Chat_System/000-Overview_and_References/PAGE.md` |
| Project overview | `docs/FUSION_STUDIO_OVERVIEW.md` |
| Architecture outline | `docs/FUSION_STUDIO_ARCHITECTURE_OUTLINE.md` |

Do not use old client or server directory names. The active directories are `fusion-studio-client/` and `fusion-studio-server/`.

## Development Commands

### Client

```bash
cd fusion-studio-client
npm install
npm run build
npm run electron:dev
```

### Server

```bash
cd fusion-studio-server
npm install
node server.js
npm test
```

The Electron development path builds the renderer and starts the app shell, which manages the server process. When validating server code directly, use `fusion-studio-server/server.js`.

## Architecture Summary

Fusion Studio has three primary runtime layers:

| Layer | Path | Responsibilities |
|------|------|------------------|
| Electron main | `fusion-studio-client/electron/` | Window lifecycle, native APIs, custom protocol, server child process |
| Renderer | `fusion-studio-client/src/` | React UI, Zustand stores, WebSocket client, iframe view mounting |
| Server | `fusion-studio-server/` | Business logic, SQLite, workspace registry, chat/thread routing, harness adapters, file/wiki/resource APIs |

The server owns persistence and orchestration. The renderer presents state and sends user intents. Electron provides local app integration and packages the client/server together.

## Chat And Harness Model

Read `ai/views/wiki-viewer/Wiki/007-Chat_System/000-Overview_and_References/PAGE.md` before changing chat, threads, harness routing, prompt acceptance, live stream rendering, or stop/interrupt behavior.

Current rules:

- `cli.json` is the harness policy. In the current default config, OpenCode is the only listed/enabled harness, so New Thread creates an OpenCode thread directly and no harness picker is shown.
- The persistent identity is the thread, not the workspace tab or current view.
- Passive browsing uses `thread:open`; it hydrates history and live snapshot state without warming or spawning a harness process.
- Assistant activation and new thread creation use `thread:open-assistant`.
- Prompt acceptance is server-owned. The user bubble is committed on `message:sent`, not on optimistic client send.
- Live streams route by `threadId`. Workspace and view context describe where the thread belongs, but `threadId` is the routing key.
- Active in-memory turns can be overlaid on top of SQLite history when a thread is revisited.
- Stop is server-owned. Interrupted turns persist as partial assistant exchanges instead of disappearing.

## Database

The server uses SQLite through Knex + `better-sqlite3`.

- Canonical DB module: `fusion-studio-server/lib/db.js`
- Dev DB path: `fusion-studio-server/data/fusion.db`
- Packaged/user-data DB path: `${FUSION_APP_USER_DATA}/server-data/fusion.db`
- Migrations: `fusion-studio-server/lib/db/migrations/`

Do not describe the current system as using any older database filename. The active database is `fusion.db`.

## Workspace And Views

Each workspace is a folder on disk with an `ai/` subtree. Fusion Studio reads workspace config, views, wiki content, styles, and state from that folder.

Views are moving toward self-contained iframe apps served through the `fusion-studio://` custom protocol. Existing React-backed views still exist during migration. Do not assume all views are React components or all views are iframes; check the current loader and view files.

## Editing Guidance

- Prefer the smallest correct change.
- Preserve existing user/worker changes in the dirty worktree.
- Do not rewrite generated/cache/runtime files unless the task specifically targets them.
- For chat/thread/harness work, start from the Chat wiki tree under `ai/views/wiki-viewer/Wiki/007-Chat_System/`.
- For database work, use migrations and query modules; do not hand-edit `fusion.db`.
- For frontend UI work, preserve the current visual language unless explicitly asked to redesign.

## Verification

Choose the narrowest verification that matches the change:

- Client build: `npm run build` in `fusion-studio-client/`
- Server tests: `npm test` in `fusion-studio-server/`
- Server smoke: `node server.js` in `fusion-studio-server/`
- Electron dev launch: `npm run electron:dev` in `fusion-studio-client/`

If verification is skipped because of time, missing dependencies, or a dirty/concurrent worktree, say so explicitly.

## Historical Names

Older docs, handoffs, archived code, and chat transcripts may mention previous project names or previous directory names. Treat those as historical unless a current file or user instruction says otherwise.

The current product name is Fusion Studio. The current active code directories are `fusion-studio-client/` and `fusion-studio-server/`.

---
> Source: [Cylon-Skin-Job/fs-dev](https://github.com/Cylon-Skin-Job/fs-dev) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
