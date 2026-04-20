## vite-three

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

- `npm run dev` - Start Vite development server
- `npm run build` - Build project for production

## Project Architecture

This is a creative coding portfolio website showcasing interactive media projects built with Three.js and p5.js. The project uses Vite as the build tool with specialized plugins for shader development.

### Core Structure

- **Main Entry**: `index.html` serves as the portfolio homepage with navigation to different projects
- **Navigation System**: `js/HomeLinks.js` contains a centralized link management system that dynamically generates navigation from the `linkInfoArrayList` array
- **Project Organization**: All projects are organized under `/pages/` with two main categories:
  - `pages/three/` - Three.js projects including boiler-plates, shaders, and creative projects
  - `pages/p5/` - p5.js creative sketches and interactive art pieces

### Three.js Architecture

The Three.js projects follow a consistent boilerplate pattern:

- **Boiler-plates**: Located in `pages/three/boiler-plates/` with `basic/` and `shader/` templates
- **Standard Setup**: Each Three.js project includes scene, camera, renderer, OrbitControls, and lil-gui for debugging
- **Shader Integration**: Projects use `vite-plugin-glsl` to import GLSL shaders as modules
- **Uniform Structure**: Shader materials follow consistent uniform organization patterns

### p5.js Architecture

- **Sketch Pattern**: Each p5.js project uses `sketch_main.js` as the main entry point
- **Creative Focus**: Projects emphasize generative art, motion graphics, and abstract visualizations
- **Modular Design**: Common functionality extracted into reusable classes and utilities

### Asset Management

- `public/textures/` - Texture assets for Three.js projects
- `public/assets/` - General media assets
- Relative path configuration ensures assets work in both development and production

### Key Dependencies

- `three` - 3D graphics library
- `lil-gui` - Debug GUI for development
- `p5.brush` - Enhanced brush capabilities for p5.js
- `vite-plugin-glsl` - GLSL shader import support
- `vite-plugin-restart` - Auto-restart on static file changes

### Development Patterns

- Each project maintains its own `index.html`, `script.js`/`sketch_main.js`, and `style.css`
- Shader projects include dedicated `shaders/` directories with `.glsl` files
- Projects are designed to be standalone but share common navigation via the HomeLinks system

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomlim2) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
