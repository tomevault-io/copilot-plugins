## aka

> AKA is a local-first, task-driven desktop application for LLM-powered coding agents.

# AKA — Root CLAUDE.md

## What This Project Is

AKA is a local-first, task-driven desktop application for LLM-powered coding agents.
Built with Tauri v2 (Rust backend) + React + TypeScript (frontend).

AKA is fully agnostic — users bring any LLM backend (Ollama, MLX, oMLX, LM Studio,
any OpenAI-compatible endpoint) and any agent (Aider, OpenCode, Hermes, OpenClaw,
LangChain, custom scripts). AKA orchestrates them; it never prescribes them.

The app is task-first: describe what you want built, pick your model + agent, launch,
review the diff, approve or reject. Not a chat interface — a work dispatch tool.

---

## Tech Stack

| Layer       | Choice                | Reason                                      |
|-------------|-----------------------|---------------------------------------------|
| Shell       | Tauri v2              | Cross-platform, Rust IPC, small binary      |
| Frontend    | React 18 + TypeScript | Ecosystem, component reuse                  |
| Bundler     | Vite                  | Fast HMR, Tauri-compatible                  |
| State       | Zustand               | Lightweight, slices pattern                 |
| Editor/Diff | Monaco Editor         | VS Code-grade, built-in diff API            |
| Terminal    | Xterm.js              | ANSI support, real-feel streaming output    |
| Styling     | Tailwind CSS v4       | Utility-first, consistent tokens            |

---

## Core Principle: Full Agnosticism

AKA never hardcodes a model, provider, or agent. Every integration is:
- Configured by the user via the Provider Manager or Agent Runner
- Stored as a user-defined record (name, command/URL, args)
- Treated identically at runtime regardless of type

AKA's job is orchestration and UX — not opinion about which LLM or agent is best.

---

## Project Conventions

### Structure
- Each feature lives under `src/features/<NN-feature-name>/`
- Every feature has its own `Context.md`
- `Context.md` = current state, decisions made, open questions

### Naming
- Components: `PascalCase.tsx`
- Hooks: `use-kebab-case.ts`
- Stores: `use-feature-store.ts` (Zustand)
- Tauri commands: `snake_case` in Rust, camelCase wrapper in `src/lib/tauri/commands.ts`

### IPC Pattern
All Tauri `invoke()` calls go through `src/lib/tauri/commands.ts`.
Never call `invoke()` directly from components.

### Streaming
Agent output streams via Tauri events, not command return values.
```ts
listen<AgentChunk>('agent:chunk', (event) => {
  appendChunk(event.payload)
})
```

### State
- One Zustand store per feature at `src/features/<feature>/use-<feature>-store.ts`
- Cross-feature state in `src/stores/`

---

## Rust Backend Rules
- Commands in `src-tauri/src/commands/` — one file per feature domain
- Use `tokio::process::Command` for spawning agent/model subprocesses
- Stream stdout/stderr back via `app_handle.emit()`
- Persist config via `tauri-plugin-store`
- Never block the main thread — everything async

---

## What NOT to Do
- Do not embed or bundle any LLM — AKA orchestrates local models, never bundles one
- Do not build a full code editor — file editing is the agent's job; AKA shows diffs
  - **Markdown exception:** `.md` / `.mdx` / `.markdown` files in a pane are
    user-editable (plain textarea via `MarkdownPane`). The agent may also
    rewrite the same file on disk via its own tools; the pane watches the
    file (`file://changed` events from `commands::files::watch_file`) and
    surfaces a conflict banner (Reload / Keep mine / View diff) when both
    sides changed. All other file types stay read-only in panes.
- Do not hardcode any agent or model type — everything is pluggable
- Do not use `localStorage` — use Tauri's store plugin for persistence
- Do not make API calls from the frontend directly — route through Tauri commands

---

## Feature Index

| #  | Feature                  | Folder                       |
|----|--------------------------|------------------------------|
| 1  | LLM Provider Manager     | `features/01-llm-provider`   |
| 2  | Agent Runner             | `features/02-agent-runner`   |
| 3  | Task Workspace           | `features/03-task-workspace` |
| 4  | Diff Viewer              | `features/04-diff-viewer`    |
| 5  | File Explorer            | `features/05-file-explorer`  |
| 6  | Streaming Output Console | `features/06-output-console` |
| 7  | Task History             | `features/07-history`        |
| 8  | Context Engine           | `features/08-context-engine` |
| 9  | Settings                 | `features/09-settings`       |
| 10 | Plugin System            | `features/10-plugin-system`  |

---
> Source: [Kellastico/AKA](https://github.com/Kellastico/AKA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
