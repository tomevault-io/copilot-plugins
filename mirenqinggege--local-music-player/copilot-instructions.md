## local-music-player

> `local-music-player` is a desktop music player application built with **Electron** and **Vue 3**. It allows users to scan local directories for audio files, manage playlists, and persist playback state. The application features a modern UI powered by **Ant Design Vue** and **Tailwind CSS**.

# Local Music Player Project Context

## Project Overview

`local-music-player` is a desktop music player application built with **Electron** and **Vue 3**. It allows users to scan local directories for audio files, manage playlists, and persist playback state. The application features a modern UI powered by **Ant Design Vue** and **Tailwind CSS**.

### Key Technologies

*   **Runtime:** [Electron](https://www.electronjs.org/) (Node.js + Chromium)
*   **Frontend Framework:** [Vue 3](https://vuejs.org/) (Composition API, `<script setup>`)
*   **Build Tool:** [Vite](https://vitejs.dev/)
*   **Language:** [TypeScript](https://www.typescriptlang.org/)
*   **UI Component Library:** [Ant Design Vue](https://antdv.com/)
*   **CSS Framework:** [Tailwind CSS](https://tailwindcss.com/)
*   **State Management:** [Pinia](https://pinia.vuejs.org/)
*   **Routing:** [Vue Router](https://router.vuejs.org/) (with `unplugin-vue-router` for file-based routing)
*   **Data Persistence:** `electron-store` (saves playlists and progress locally)
*   **Audio Metadata:** `music-metadata` (parses ID3 tags and duration)

## Architecture

The project follows the standard Electron multi-process architecture:

1.  **Main Process** (`electron/main.ts`):
    *   Creates and manages the application window (`BrowserWindow`).
    *   Handles system-level operations like file scanning (`fs`, `music-metadata`).
    *   Manages data persistence via `electron-store`.
    *   Communicates with the renderer process via IPC channels (`audio:scan-folder`, `storage:save-playlist`, etc.).

2.  **Renderer Process** (`src/`):
    *   **UI/UX:** A Single Page Application (SPA) built with Vue 3.
    *   **Store** (`src/store/player.ts`): Centralized state for the music player (playlist, playback status, volume). Interacts with the Main process to save/load state.
    *   **Routing:** Uses file-based routing convention inside `src/pages`.

3.  **Communication:**
    *   The renderer invokes Electron features via `window.electronAPI` (defined in `electron/preload.ts` - *inferred*).

## Key Features

*   **Recursive File Scanning:** Scans selected directories for audio files (.mp3, .wav, .ogg, .m4a, .flac, .aac).
*   **Playback Control:** Play, pause, seek, volume control, next/previous track.
*   **Playlist Management:** Add/remove tracks, clear playlist.
*   **Persistence:** Automatically saves the current playlist and playback progress (track and time) on change.

## Building and Running

### Prerequisites
*   Node.js (LTS recommended)
*   npm or pnpm

### Commands

*   **Development:**
    ```bash
    npm run dev
    # Starts the Vite dev server and launches the Electron app window.
    ```

*   **Production Build:**
    ```bash
    npm run build
    # Runs type checking (vue-tsc), builds the Vue app, and packages the Electron app using electron-builder.
    ```

*   **Preview:**
    ```bash
    npm run preview
    # Previews the Vite app (useful for checking build output, though Electron features may be limited).
    ```

## Directory Structure

*   `electron/`: Source code for the Electron Main process and Preload script.
*   `src/`: Source code for the Vue Renderer process.
    *   `assets/`: Static assets (images, fonts).
    *   `components/`: Reusable Vue components.
    *   `layouts/`: Layout components (via `unplugin-vue-layouts`).
    *   `pages/`: Application views (via `unplugin-vue-router`).
    *   `store/`: Pinia state stores (logic for player state).
    *   `types/`: TypeScript type definitions.
*   `dist-electron/`: Compiled output for the Main process.
*   `dist/`: Compiled output for the Renderer process (web assets).

## Development Conventions

*   **Path Aliases:** Use `@` to refer to the `src/` directory.
*   **Styling:** 
    *   **Tailwind CSS:** All Tailwind utility classes MUST use the `tw:` prefix (e.g., `tw:flex`, `tw:bg-blue-600`).
    *   Prefer utility classes for layout and spacing. Use Ant Design Vue components for complex UI elements.
*   **State:** Use Pinia for global state. Encapsulate business logic (like audio handling) within store actions.
*   **IPC:** All Main-Renderer communication should be typed and safe.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mirenqinggege)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mirenqinggege)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
