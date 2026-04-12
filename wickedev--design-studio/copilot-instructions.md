## design-studio

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Design Studio - A high-performance design editor for web and desktop platforms, similar to Figma. Built with C++ core engine compiled to WebAssembly and React/TypeScript frontend.

## Development Commands

### Frontend Development
- `yarn install` - Install dependencies
- `yarn dev` - Start development server (Vite)
- `yarn build` - Build for production
- `yarn preview` - Preview production build
- `yarn lint` - Run ESLint
- `yarn type-check` - TypeScript type checking

### WASM Core Engine
- `./scripts/build-wasm.sh` - Build C++ core to WebAssembly
- `yarn build:wasm` - Alternative WASM build command

### Prerequisites
- Node.js and Yarn
- Emscripten SDK for WASM compilation
- CMake for C++ build system

## Architecture

```
Frontend (TypeScript/React)
    ↕ (WebAssembly bindings)
Core Engine (C++)
    ├── Skia (Graphics & Rendering)
    ├── HarfBuzz + FreeType (Typography)
    ├── ICU4C (Internationalization)
    ├── QuickJS (Plugin System)
    └── WebGL/WebGPU (Hardware Acceleration)
```

### Directory Structure
- `frontend/` - React/TypeScript UI application
- `core/` - C++ engine source code and CMake configuration
- `scripts/` - Build and development scripts
- `dist/` - Production build output

### Key Files
- `core/include/studio_engine.h` - Main C++ engine interface
- `frontend/src/engine/DesignEngine.ts` - TypeScript WASM wrapper
- `core/CMakeLists.txt` - C++ build configuration
- `vite.config.ts` - Frontend build configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wickedev)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/wickedev)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
