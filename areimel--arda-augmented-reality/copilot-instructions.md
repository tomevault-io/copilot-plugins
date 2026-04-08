## arda-augmented-reality

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

| Command | Purpose |
| --- | --- |
| `npm install` | Install dependencies |
| `npm run dev` | Start local development server at localhost:4321 |
| `npm run build` | Build production site with type checking (uses increased memory allocation) |
| `npm run preview` | Preview production build locally |
| `astro check` | Run TypeScript type checking |

## Project Architecture

This is an Augmented Reality (AR) project built with **Astro** as the main framework, using **A-Frame** for 3D rendering and **Mind-AR** for marker-based AR tracking. The project creates deployable AR experiences that work in web browsers.

### Core Technology Stack
- **Astro**: Static site generator and component framework
- **A-Frame**: WebXR framework for building 3D/AR experiences
- **Mind-AR**: Image tracking library for marker-based AR
- **SCSS**: Styling with CSS variables system
- **TypeScript**: Type safety and development tooling

### Project Structure Pattern
Each AR experience follows a consistent structure:
- **Project Data**: YAML configuration files in `src/data/projectData/` define all experience parameters with inline documentation
- **Pages**: Each project has its own folder in `src/pages/` with `index.astro`, `splash.astro`, and `project-info.astro`
- **Markers**: Compiled marker files (`.mind` format) and images stored in `public/markers/`
- **Assets**: 3D objects, videos, and images in appropriate `public/` subdirectories

### Key Components
- **AR-Layout.astro**: Main layout wrapper that loads AR dependencies
- **HeadAR.astro**: Loads all AR libraries (A-Frame, Mind-AR, chromakey support)
- **CustomUI.astro**: Handles AR UI states (scanning, marker found/lost)
- **Camera.astro**: A-Frame camera configuration with raycaster support

### AR Experience Configuration
Projects are configured via YAML files with these key sections:
- `experienceType`: "video", "image", or "3d-object"
- `markerFile`: Path to compiled `.mind` marker file
- `modelSettings`: Position/rotation configurations for wall/flat/card orientations
- `uiPreset`: UI preset for messaging ("default", "product", "artwork")
- `splashPage`: Background images and branding
- All YAML files include inline comments for field explanations

### Marker System
- Markers must be compiled using Mind-AR's online tool: https://hiukim.github.io/mind-ar-js-doc/tools/compile/
- Compiled markers generate `.mind` files used by the AR tracking system
- Original marker images are also stored for reference and printing

### Position Control System
The `ar-functions.js` file provides a position control system allowing users to switch between:
- **Wall**: Vertical surface mounting (0° rotation)
- **Flat**: Horizontal surface mounting (90° rotation)
- **Card**: Business card or small format mounting

### Asset Pipeline
- **Videos**: MP4 format, supports alpha channel/green screen with chromakey shader
- **3D Objects**: GLTF format stored in `public/3d-objects/`
- **Images**: Standard web formats for textures and UI elements

### Adding New Projects
1. Copy `general-marker-template.yaml` to create new project configuration (includes inline documentation)
2. Edit the YAML file with your project's settings - comments explain each field
3. Add markers and assets to appropriate `public/` directories
4. Test with `npm run dev` - the page will auto-generate at `/{filename}`
5. TypeScript interfaces in `src/types/projectData.ts` provide type safety

## Important Development Notes

- All AR scripts are loaded inline via HeadAR.astro component
- Project data is imported as YAML modules in Astro pages with full TypeScript support
- YAML files include inline comments for developer guidance and field documentation
- The system supports green screen video compositing for advanced effects
- Marker tracking events (targetFound/targetLost) control media playback
- Build process includes TypeScript checking and uses increased memory allocation for large builds
- TypeScript interfaces in `src/types/projectData.ts` ensure type safety across the project

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/areimel)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/areimel)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
