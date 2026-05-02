## project-nodal

> **Sticky Note ChatBot** is a "spatial thinking canvas" where AI conversations are treated as persistent, movable objects (Sticky Notes) rather than a linear stream. The project prioritizes privacy (local-first storage), simplicity, and a fluid user experience.

# Sticky Note ChatBot

## Project Overview

**Sticky Note ChatBot** is a "spatial thinking canvas" where AI conversations are treated as persistent, movable objects (Sticky Notes) rather than a linear stream. The project prioritizes privacy (local-first storage), simplicity, and a fluid user experience.

**Key Features:**
*   **Workspaces:** Multiple independent canvases for different contexts.
*   **Spatial Canvas:** Infinite panning/zooming canvas to organize notes.
*   **Sticky Notes:** Independent chat sessions that can be dragged, collapsed, and expanded.
*   **Zones:** Organize notes into named, nestable areas. Notes inside zones move with them.
*   **AI Integration:** Supports local LLMs (via Ollama) and cloud providers (OpenAI).
*   **Privacy:** Data is stored locally using IndexedDB (Dexie.js). No server-side storage of chat history.

## Tech Stack

*   **Frontend Framework:** React 19 + Vite
*   **Styling:** TailwindCSS v4
*   **State Management:** Zustand
*   **Persistence:** Dexie.js (IndexedDB wrapper)
*   **Animations:** Framer Motion
*   **Gestures:** @use-gesture/react
*   **Icons:** Lucide React
*   **Markdown:** react-markdown, remark-gfm, marked

## Building and Running

### Prerequisites
*   Node.js (latest stable version recommended)
*   npm

### Commands

*   **Start Development Server:**
    ```bash
    npm run dev
    ```
    Runs the app at `http://localhost:5173`.

*   **Build for Production:**
    ```bash
    npm run build
    ```

*   **Linting:**
    ```bash
    npm run lint
    ```

*   **Preview Build:**
    ```bash
    npm run preview
    ```

## Architecture & Key Components

### File Structure

**Core:**
*   `src/store/useStore.js`: Zustand store. Single source of truth for Workspaces, Canvas, Notes, Zones. Handles IndexedDB persistence via `updateCanvas(workspaceId, patch)`.
*   `src/services/ai_provider.js`: AI API calls (Ollama/OpenAI) with streaming.
*   `src/db.js`: Dexie.js database schema.

**Components:**
*   `src/components/Canvas.jsx`: Main workspace. Coordinates gestures, notes, zones, and background.
*   `src/components/StickyNote.jsx`: Chat UI with drag, expand/collapse, and AI integration.
*   `src/components/canvas/ConnectionsLayer.jsx`: SVG layer for note connections.
*   `src/components/SettingsModal.jsx`: AI provider configuration.
*   `src/components/WorkspaceLayout.jsx`: Main layout wrapper.

**Hooks:**
*   `src/hooks/useCanvasGestures.js`: Canvas gesture handling (pan, zoom, zone drag/resize, background drag/resize).

**Utilities:**
*   `src/utils/zoneUtils.js`: Zone-related constants and helper functions.

### Data Flow

```
┌─────────────────┐     patchCanvas()     ┌──────────────────┐
│   Canvas.jsx    │ ──────────────────────▶│   useStore.js   │
│   (UI Layer)    │                        │  (Zustand Store) │
└─────────────────┘                        └────────┬─────────┘
        ▲                                           │
        │  zones, notes, viewport, etc.             │ updateCanvas()
        │  (read from store)                        ▼
        │                                  ┌──────────────────┐
        └──────────────────────────────────│   IndexedDB      │
                                           │   (Dexie.js)     │
                                           └──────────────────┘
```

### Data Model
*   **Workspace:** Root entity. Contains one Canvas.
*   **Canvas:** `{ notes[], zones[], connections[], viewport, backgroundImage, backgroundTransform }`
*   **Zone:** Named area for organizing notes. Supports nesting.
*   **Note:** Single chat session with position, dimensions, messages, and optional zoneId.

## Development Conventions

*   **Local-First:** All state changes should be reflected in the UI immediately (optimistic updates) and persisted to IndexedDB.
*   **Styling:** Use TailwindCSS utility classes.
*   **Components:** Functional components with Hooks.
*   **AI Handling:** AI logic is abstracted in `ai_provider.js`. Use the `createAIProvider` factory.
*   **State:** Complex global state lives in Zustand (`useStore`). Local component state (like UI toggles) stays local.

---
> Source: [yibie/project-nodal](https://github.com/yibie/project-nodal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
