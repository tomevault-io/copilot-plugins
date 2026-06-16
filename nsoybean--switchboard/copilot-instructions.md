## switchboard

> Desktop multi-agent coding orchestrator for Claude Code, Codex, and Bash.

# Switchboard

Desktop multi-agent coding orchestrator for Claude Code, Codex, and Bash.

## Source Of Truth
- This file is the canonical agent instruction file for the repo.
- Keep `CLAUDE.md` as a thin pointer to `AGENTS.md` so project rules stay portable across agents.
- Read `docs/ARCHITECTURE.md` for the current feature map, module guide, and runtime flows.

## Stack
- Backend: Rust + Tauri v2 in `src-tauri/`
- Frontend: React + TypeScript in `src/`
- Styling: Tailwind CSS v4 + shadcn/ui
- Terminal: xterm.js v6 + WebGL addon
- PTY: `portable-pty` via custom Tauri commands
- State: React Context + `useReducer`
- Git: git CLI subprocesses only

## Core Commands
```bash
npm install
npm run tauri dev
npm run build
npx tsc --noEmit
cargo test --manifest-path src-tauri/Cargo.toml
```

## Architecture
- PTY pipeline: `portable-pty -> per-session Tauri events (pty-output-{id}) -> usePty -> xterm.js`
- Active Switchboard sessions are persisted in `~/.switchboard/sessions.json`
- Claude history is read from `~/.claude/history.jsonl` and `~/.claude/projects/...`
- Codex history is read from `~/.codex/state_5.sqlite` and `~/.codex/archived_sessions`
- Switchboard keeps local metadata overlays for archived Claude/Codex sessions in `~/.switchboard/*_session_metadata.json`
- Past sessions open in a structured transcript view; resuming returns to the live PTY experience
- Git panel and worktree operations always run in the session's worktree via git CLI

## UI Rules
- Fonts: Inter for UI, JetBrains Mono for code/terminal
- Use shadcn/ui patterns and semantic tokens such as `bg-background`, `bg-muted`, `text-muted-foreground`
- Support both light and dark mode; avoid hardcoded single-theme styling
- Keep the app clean, minimal, and IDE-like
- Transcript/history UI should match app typography, using mono only for code, diffs, commands, and terminal-like content

## Testing
- Rust tests live alongside source in `mod tests`
- Run `cargo test --manifest-path src-tauri/Cargo.toml` for backend changes
- Run `npx tsc --noEmit` for frontend/type changes

---
> Source: [nsoybean/switchboard](https://github.com/nsoybean/switchboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
