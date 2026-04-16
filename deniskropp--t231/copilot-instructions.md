## t231

> **Deep Violet Cyber Tunnel** is an audio-reactive, 140 BPM optimized desktop engine designed as a KDE Plasma Wallpaper. It leverages GLES 3.x for high-performance visual effects and integrates with PipeWire for real-time audio synchronization.

# GEMINI.md

## Project Overview

**Deep Violet Cyber Tunnel** is an audio-reactive, 140 BPM optimized desktop engine designed as a KDE Plasma Wallpaper. It leverages GLES 3.x for high-performance visual effects and integrates with PipeWire for real-time audio synchronization.

The project is currently maintained as a comprehensive manifest in `Deep Violet Desktop Engine.md`, which contains all necessary source files, metadata, and build configurations.

### Main Technologies
- **Frontend:** Qt 5 / QML (Quick)
- **Backend:** C++ (Qt Framework)
- **Visuals:** OpenGL ES 3.0 (GLSL Shaders)
- **Audio:** PipeWire (libpipewire-0.3)
- **Signal Processing:** KissFFT (float version)
- **Build System:** CMake

### Architecture
- **QML Layer:** Handles user configuration (glitch intensity) and serves as the visual container.
- **C++ Renderer:** Inherits from `QQuickFramebufferObject` to bridge QML and OpenGL.
- **Audio Analysis Pipeline:** Taps into PipeWire system audio, processes PCM data via FFT (KissFFT), and passes magnitude data to the GPU via textures.
- **Shader Pipeline:** A "Visual DNA" consisting of vertex and fragment shaders that render a neon-violet cyber tunnel effect reactive to bass and high frequencies.

## Building and Running

The project requires extraction of the source files from the manifest before building.

### Prerequisites
- KDE Plasma Development headers
- Qt 5 Core, Quick, and OpenGL
- PipeWire Development libraries (`libpipewire-0.3-dev`)
- KissFFT library
- CMake 3.16+

### Build Steps (Conceptual)
1. **Extract Files:** Separate the files listed in `Deep Violet Desktop Engine.md` into their respective paths (e.g., `src/`, `contents/ui/`).
2. **Configure:**
   ```bash
   mkdir build && cd build
   cmake ..
   ```
3. **Build:**
   ```bash
   make
   ```
4. **Install:**
   Install as a Plasma Wallpaper package using `kpackagetool5`.

## Development Conventions

- **Modern C++:** Uses idiomatic C++11/14/17 patterns within the Qt framework.
- **Shader Performance:** Optimized for OpenGL ES 3.0 to ensure low CPU/GPU overhead while running as a background wallpaper.
- **Audio Reactivity:** Frequency-specific reactivity (Bass for speed/tunnel movement, Highs for glitch effects).
- **Naming Conventions:** Follows standard Qt/KDE naming patterns (PascalCase for classes, camelCase for properties).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/deniskropp) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
