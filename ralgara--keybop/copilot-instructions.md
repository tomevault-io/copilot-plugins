## keybop

> KeyBop is a finger dexterity trainer for keyboard/piano players. It's a rhythm game-style web app with a piano roll displaying falling notes. Users play along on their keyboard, earning points for accuracy and timing precision.

# KeyBop — Claude Code Project Instructions

## What is this project?

KeyBop is a finger dexterity trainer for keyboard/piano players. It's a rhythm game-style web app with a piano roll displaying falling notes. Users play along on their keyboard, earning points for accuracy and timing precision.

## Tech Stack

- **Frontend:** React + TypeScript
- **Build:** Vite (react-ts template)
- **Rendering:** HTML Canvas (piano roll, falling notes)
- **State:** React Context with useReducer (no external state libraries)
- **Dependencies:** None beyond Vite defaults — no external packages

## Architecture

The game engine is **pure TypeScript** with zero React imports. It owns the `requestAnimationFrame` loop and communicates with React through state snapshot callbacks. React handles UI chrome (menus, HUD, settings) but never touches the canvas directly.

```
React UI (menus, HUD, settings)
    ↕ state snapshots + control methods
Game Engine (rAF loop, pure TS)
    ├── TimingSystem     — clock, delta-time, speed multiplier
    ├── NoteManager      — spawn/advance/expire falling notes
    ├── InputHandler     — keyboard events → normalized InputEvents
    ├── ScoringEngine    — timing window evaluation, combo, score
    └── Renderer         — all canvas drawing
```

## Key Design Decisions

- Use `event.code` (not `event.key`) for key mapping — physical key position, layout-agnostic
- Beat-based sequences in JSON, converted to ms at load time — allows different BPMs
- State snapshot throttling — engine emits to React at ~15fps to avoid re-render interference with 60fps canvas
- `InputEvent` interface is input-source-agnostic — MIDI integration can be added later without engine changes

## Implementation Status

Phase 1 (Vite scaffold) is complete. See `docs/IMPLEMENTATION_PLAN.md` for the full ordered build plan covering phases 1-7.

## Commands

- `npm run dev` — start dev server
- `npm run build` — production build
- `npm run lint` — run ESLint

## Coding Conventions

- TypeScript throughout, strict mode
- No external dependencies beyond Vite defaults
- Game engine modules go in `src/engine/` — pure TS, no React imports
- React components go in `src/components/`
- Types in `src/types/`, one file per domain (note, sequence, scoring, input, game)
- Constants centralized in `src/engine/constants.ts`
- Custom key mappings persisted to localStorage under key `keybop-keymap`
- Commit messages: no AI attribution in commit messages

## Cloud Environment Setup

Before using Claude Code on the web (cloud sessions), ensure the following are configured:

1. **Add the GitHub token** — Go to the project's environment variables and add your GitHub personal access token so that git operations (push, PR creation) work correctly.
2. **Install the GitHub integration** — Run `/install-github-app` to install the GitHub app integration. This enables PR creation, issue management, and other GitHub API operations from within the cloud session.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ralgara) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
