## native-dji-browser

> **Native DJI Browser** is a professional-grade, high-performance web application designed specifically for managing DJI Action series camera media assets directly from local storage (SD cards or local folders).

# Native DJI Browser - Project Context & Instructions

## 1. Project Overview
**Native DJI Browser** is a professional-grade, high-performance web application designed specifically for managing DJI Action series camera media assets directly from local storage (SD cards or local folders).

### Main Technologies:
- **Framework**: Vue 3 (Composition API) + TypeScript + Vite.
- **Styling**: Tailwind CSS v4 + Lucide Icons + Shadcn-style UI components.
- **Core Browser APIs**: 
  - **File System Access API**: For direct read/write access to local DCIM folders.
  - **IndexedDB**: For persistent storage of directory handles and reconstructed metadata registry.
- **Embedded Libraries**:
  - **SQL.js (WebAssembly)**: To parse DJI's native `AC004.db` SQLite database.
  - **FFmpeg WASM**: For on-demand high-quality thumbnail extraction and frame capture.
  - **ExifReader**: For extracting metadata and embedded thumbnails from DNG/RAW files.
  - **VueUse**: For dark mode, intersection observers, and local storage management.

## 2. Architecture & Directory Structure
The project follows a "Local-First" architecture where no media is ever uploaded to a server. Data is reconstructed locally in the browser's IndexedDB.

- `/src/composables`:
  - `useDjiMedia.ts`: Handles folder scanning, file grouping (MP4/LRF/THM correlation), and burst stacking logic.
  - `useDjiDatabase.ts`: Responsible for loading `AC004.db` and building a global metadata registry (ISO, GPS, Markers, etc.).
  - `useDjiStore.ts`: Low-level IndexedDB management using `idb`.
  - `useFfmpeg.ts`: Manages the lifecycle of FFmpeg WASM for on-demand processing.
  - `useTaskQueue.ts`: Controls concurrency for heavy WASM tasks.
- `/src/components`:
  - `MediaCard.vue`: High-performance grid item with lazy-loading, pre-fetching (400px buffer), and stack visuals.
  - `MediaModal.vue`: Premium detail view with 3D perspective transitions, native video/photo rendering, and metadata display.

## 3. Key DJI Logic Conventions
Future modifications must adhere to these DJI-specific data structures:
- **Media Grouping**: Files are grouped by basename (e.g., `DJI_0001`). A group includes `.MP4` (Master), `.LRF` (Low-res Proxy), `.THM` (Thumbnail), `.JPG` (Stills), and `.DNG` (Raw).
- **Preview Strategy**: Always prioritize `.THM` for grid display and `.LRF` for detail video playback to maintain performance with 4K/8K content.
- **Metadata Source**: Trust `AC004.db` over file system metadata. It contains specific DJI tags like `is_favorite`, `markers` (highlights), and camera settings.
- **Burst Stacking**: Group photos taken within 3 seconds of each other into a single UI stack.

## 4. Building and Running
### Development
```bash
npm install
npm run dev
```
### Production Build
```bash
npm run build
```
*(Note: Production server must include COOP/COEP headers for SharedArrayBuffer/WASM support)*

## 5. Development Guidelines
- **Performance First**: Large media folders (1000+ items) are expected. Use `IntersectionObserver` and task queues for any heavy processing.
- **Memory Management**: Always call `URL.revokeObjectURL()` for preview Blobs during cleanup/disconnect to prevent memory leaks.
- **Stability**: WASM operations are sensitive. Limit concurrency to 1 or 2 and use "Safe Mode" (skipping very large files) for FFmpeg tasks.
- **UI/UX Style**: Maintain the "Industrial Apple-like" aesthetic: high-contrast typography (10px black uppercase), subtle glassmorphism (`backdrop-blur`), and Hard-Edge rounded corners (`rounded-lg` or `rounded-xl`).
- **Dark Mode**: Always ensure new components fully support `bg-background` and `text-foreground` variables for seamless theme switching.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/OSpoon)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/OSpoon)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
