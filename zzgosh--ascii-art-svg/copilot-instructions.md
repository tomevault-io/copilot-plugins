## ascii-art-svg

> This file provides guidance to coding agents when working with code in this repository.

# AGENTS.md

This file provides guidance to coding agents when working with code in this repository.

## Project Overview

This is an ASCII Art SVG Generator - a Vue 3 web application that converts text into ASCII art using the figlet.js library and exports the results as high-quality SVG files. The app features real-time generation, dynamic font loading, keyboard shortcuts for font navigation, character width control, and advanced SVG export with pixel-to-shape conversion.

## Development Commands

```bash
# Install dependencies
npm install

# Development server with hot reload
npm run dev

# Build for production
npm run build

# Type checking only
npm run type-check

# Lint code with auto-fix
npm run lint

# Format code with prettier
npm run format

# Preview production build
npm run preview
```

## Architecture

### Core Structure
- **Vue 3** with Composition API and TypeScript
- **Vite** build system with hot module replacement
- **Vue Router** for single-page application routing
- **Pinia** for state management (minimal usage)
- **TailwindCSS** for utility-first styling
- **figlet.js** for ASCII art generation with dynamic font loading

### Key Components
- `AsciiArtGenerator.vue` - Main component containing all ASCII art generation logic, font selection, keyboard navigation, character width control, and advanced SVG export functionality
- `App.vue` - Root component with router outlet
- `HomeView.vue` - Simple wrapper for the main generator component

### Font System
The application now supports **295 ASCII fonts** with dynamic loading through a dedicated font loader utility. Fonts are loaded on-demand to optimize performance. The system includes:
- **Dynamic Font Loading**: Fonts are imported using Vite's dynamic imports when needed
- **Font Status Indicators**: Visual checkmarks show which fonts are loaded
- **Comprehensive Font Library**: From basic fonts like Standard and Big to specialized ones like Star Wars, Rammstein, and various artistic styles
- **Font Navigation**: Dropdown selection, Previous/Next buttons, and arrow key navigation (↑/↓)
- **Character Width Control**: Multiple width options including Full, Fitted, Controlled Smushing, Universal Smushing, and Default

### Font Loading Architecture
Located in `src/utils/fontLoader.ts`:
- **Static Import Mapping**: Pre-defined imports for all 295 figlet fonts for Vite compatibility
- **Lazy Loading**: Fonts are loaded only when requested
- **Load Status Tracking**: Reactive tracking of which fonts are loaded
- **Error Handling**: Graceful handling of font loading failures

### Rendering Paths
- **Shared Generation Step**: All font styles are generated through `figlet.text(...)` in `src/components/AsciiArtGenerator.vue`
- **Generic Text Path**: Most fonts are previewed as raw monospace text and exported through canvas sampling plus rectangle merging
- **Dedicated ANSI Shadow Path**: `src/utils/asciiVectorRenderer.ts` detects ASCII output built from `█` plus `╔═╗║╚╝`, splits face and shadow layers, rasterizes them separately, and emits shape-only SVG
- **Preview/Export Consistency**: ANSI Shadow-style output uses the same dedicated vector pipeline for both preview and export to reduce visual drift

### Advanced SVG Export Logic
Located in `src/components/AsciiArtGenerator.vue` and `src/utils/asciiVectorRenderer.ts`:
- **Generic Export Path**: Uses a high-resolution canvas plus rectangle merging for the standard monospace text pipeline
- **ANSI Shadow Vector Export**: Uses a dedicated layered renderer with face/shadow separation and shape-only SVG output
- **Rectangle Merging Algorithm**: Finds and merges adjacent filled pixels into larger rectangles for smaller SVG output
- **Dimension Calculation**: Sizes the exported SVG from measured content bounds
- **Filename Sanitization**: Builds a clean filename from the input text and character width settings

### Character Width Options
The application provides fine-grained control over ASCII art appearance:
- **Full**: Maximum character spacing
- **Fitted**: Minimal character spacing
- **Controlled Smushing**: Controlled character overlap with specific rules
- **Universal Smushing**: Universal character overlap rules
- **Default**: Standard figlet spacing

### Keyboard Shortcuts
- `↑` (Up Arrow): Switch to previous font
- `↓` (Down Arrow): Switch to next font

## File Structure Notes

- **Main Logic**: All core functionality consolidated in `AsciiArtGenerator.vue`
- **ANSI Shadow Vector Rendering**: Dedicated logic in `src/utils/asciiVectorRenderer.ts`
- **Font Management**: Dedicated `src/utils/fontLoader.ts` utility for font operations
- **Type Definitions**: TypeScript definitions in `src/types/figlet-fonts.d.ts`
- **Routing**: Minimal router configuration with home and about routes
- **Styling**: TailwindCSS utility classes with responsive design
- **Build Configuration**: Multiple TypeScript configs for different build targets

## Development Notes

- **Import Alias**: `@` alias points to `src/` directory
- **Dynamic Font Loading**: Fonts are loaded on-demand using dynamic imports
- **Reactive UI**: Font loading status is tracked reactively for UI updates  
- **Dual SVG Export Paths**: Generic fonts use the legacy text/canvas export path, while ANSI Shadow-style output uses the dedicated vector renderer
- **Figma Compatibility**: The ANSI Shadow vector path emits shape-only SVG, avoiding downstream font substitution
- **Error Handling**: Comprehensive error handling for font loading and ASCII generation
- **Performance Optimization**: Only loads fonts when needed, optimized SVG output
- **Vite Compatibility**: Font imports are statically defined for proper Vite bundling

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zzgosh) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
