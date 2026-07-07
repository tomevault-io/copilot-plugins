## flowcal

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

FlowCal is a visual node graph editor and calculator built with React. Users create computational flows by connecting nodes visually. It supports 100+ node types, custom node composition, real-time evaluation, and cloud storage via Supabase.

## Commands

```bash
npm run dev          # Start Vite dev server with HMR
npm run build        # Production build
npm run lint         # ESLint checks
npm run test         # Vitest in watch mode
npm run test:run     # Vitest single run (CI mode)
```

No TypeScript — the project uses JSX only.

## Architecture

### Graph Engine (`src/engine/`)

The computational core. `evaluateGraph()` in `evaluator.js` is a pure function that performs depth-first traversal with cycle detection. `nodeDefinitions.js` contains the NODE_LOGIC registry mapping ~100+ node types to their computation functions. `valueResolution.js` handles type coercion between connected nodes.

Key concepts: Warp nodes (non-linear skip connections), GROUP nodes (subgraph composition), global variables via GET_GLOBAL nodes.

### Editor (`src/components/Editor.jsx`)

The main controller component (~19KB). Manages canvas state, node CRUD, connections, selection, clipboard, undo/redo, and keyboard shortcuts. Most editor state lives in local useState/useRef, not in context.

### Node Components (`src/components/flow/`)

`Node.jsx` renders individual nodes. Extracted sub-components: `NodeHeader`, `NodeBody`, `NodeHandles` (in `node/` subdirectory). `nodeUIMap.js` maps node types to their UI components. Node-specific body renderers live in `node/bodies/`.

### State Management

- **React Context** for auth (`AuthContext`), toasts (`ToastContext`), and confirm dialogs (`ConfirmContext`)
- **useHistory hook** (`src/hooks/useHistory.js`) for undo/redo
- **No Redux** — editor state is local to `Editor.jsx`

### Data Layer

`src/services/flowService.js` — Supabase CRUD for flows. `src/lib/supabase.js` initializes the client. Requires `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY` env vars.

### Routing (`src/App.jsx`)

- `/dashboard`, `/editor`, `/profile` — protected by `AuthGuard`
- `/admin` — protected by `AuthGuard` + `AdminGuard`
- `/guest` — public (no auth)
- Role system: User, Admin, Superuser

### Styling

Tailwind CSS with `class`-based dark mode. Six themes defined in `src/themes.js` (light, dark, cyberpunk, dracula, ocean, forest). Theme CSS variables set in `src/index.css`.

---
> Source: [atelier-project/flowcal](https://github.com/atelier-project/flowcal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
