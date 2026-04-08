## image-toolbox

> Always speak to user in Chinese.

# Image Toolbox
## User requirement
Always speak to user in Chinese.
## Project Overview
This is a desktop application built with **Tauri v2**, designed as a toolbox for image processing tasks. The primary feature is a high-performance image batch processor and compressor powered by **FFmpeg**. The application leverages the security and performance of Rust for the backend while providing a modern, reactive user interface using Vue 3.

## Tech Stack

### Core
- **Framework:** [Tauri v2](https://v2.tauri.app/) (Rust + Webview)
- **Language:** TypeScript, Rust

### Frontend
- **Framework:** Vue 3 (Composition API, `<script setup>`)
- **Build Tool:** Vite
- **Styling:** Tailwind CSS (v4)
- **Icons:** Lucide Vue Next
- **State Management:** Vue Reactivity API (ref, reactive)

### Backend
- **Runtime:** Rust
- **Plugins:**
  - `@tauri-apps/plugin-shell`: Used to execute FFmpeg commands.
  - `@tauri-apps/plugin-fs`: Filesystem operations (readDir, mkdir, exists).
  - `@tauri-apps/plugin-dialog`: System file pickers.

### External Dependencies
- **FFmpeg:** The application requires `ffmpeg` to be installed and available in the system's `PATH` to perform image processing operations.

## Project Structure

```text
├── src/                    # Frontend source code
│   ├── components/         # Vue components
│   │   └── ImageCompressor.vue  # Main image processing logic & UI
│   ├── App.vue             # Main application layout & navigation
│   └── main.ts             # Frontend entry point
├── src-tauri/              # Backend source code & config
│   ├── capabilities/       # Security capabilities (permissions)
│   │   └── default.json    # Configures access to FS and Shell
│   ├── src/                # Rust source files
│   ├── Cargo.toml          # Rust dependencies
│   └── tauri.conf.json     # Main Tauri configuration
└── package.json            # Node dependencies and scripts
```

## Development & Usage

### Prerequisites
1.  **Node.js** (Latest LTS recommended).
2.  **Rust** (Stable toolchain).
3.  **FFmpeg**: Must be installed and accessible via command line.

### Key Commands

- **Install Dependencies:**
  ```bash
  npm install
  ```

- **Run in Development Mode:**
  Starts the Tauri application window with hot-reload.
  ```bash
  npm run tauri dev
  ```

- **Run Frontend Only:**
  Starts only the Vite dev server (useful for UI work, but Tauri APIs will not function).
  ```bash
  npm run dev
  ```

- **Build for Production:**
  Compiles the frontend and builds the Rust binary / installer.
  ```bash
  npm run tauri build
  ```

## Configuration Details

### Permissions (`src-tauri/capabilities/default.json`)
- **Filesystem (`fs:scope`):** configured to allow access to all directories (`["**"]`) to ensure the user can select and process images from any location on their disk.
- **Shell (`shell:allow-execute`):** explicitly allows the execution of the `ffmpeg` command with arguments.

### Image Processing Logic
The core logic resides in `src/components/ImageCompressor.vue`. It:
1.  Scans the selected input directory using `fs` plugin.
2.  Filters for valid image extensions.
3.  Constructs FFmpeg arguments based on user settings (resize, crop/pad, format).
4.  Executes FFmpeg for each file via the `shell` plugin.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/EIHRTeam)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/EIHRTeam)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
