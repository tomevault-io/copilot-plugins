## colorguru

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Color Guru is a React + TypeScript web application for generating accessible color palettes using advanced easing curves and HSL color interpolation. The app allows users to create visually consistent color gradients with real-time WCAG accessibility feedback.

## Commands

### Development
```bash
npm run dev          # Start Vite dev server at http://localhost:5173
npm run build        # Type-check with tsc and build for production
npm run preview      # Preview production build locally
npm run lint         # Run ESLint on all .ts and .tsx files
```

## Architecture

### Core Color Generation Flow

The application's color generation is centralized in a single flow:

1. **App.tsx** maintains `ColorState` (steps, hue, saturation, brightness, and optional pinnedColor) and uses `useMemo` to call `generatePalette()`
2. **paletteGenerator.ts** (`generatePalette()`) orchestrates the entire color generation:
   - Retrieves easing functions (either custom cubic-bezier or preset) from `easingCurves.ts`
   - For each step (0 to n-1), calculates progress (0 to 1) and applies easing curves
   - Uses `colorMath.ts` utilities to interpolate HSL values and convert to RGB/hex
   - If a pinned color is specified, finds best position using multi-dimensional similarity and replaces that generated color
   - Calculates WCAG contrast ratios and accessibility levels for each color
   - Returns `PaletteData` with colors array and value arrays for graphing

3. **Consumption**: The resulting `PaletteData` is consumed by:
   - `Graph.tsx` for visualizations (hue, saturation, brightness, luminance, sat×bri)
   - `PalettePreview.tsx` for color swatches with accessibility indicators (📌 icon for pinned colors)
   - `ExportModal.tsx` for CSS, JSON, plain text, and Figma SVG export formats

### Key Modules

#### Color Mathematics (`src/utils/colorMath.ts`)
- HSL ↔ RGB ↔ Hex conversions using standard formulas
- `hexToHSL()` supports both 3-char and 6-char hex notation with/without #
- Hue interpolation with short/long path support around the 360° color wheel
- WCAG 2.0 luminance and contrast ratio calculations
- All color operations are pure functions

#### Easing Curves (`src/utils/easingCurves.ts`)
- Preset easing functions (Linear, Sine, Quad, Cubic, Quart, Quint, Expo, Circ, Back)
- Cubic-bezier implementation using Newton-Raphson + bisection method
- `CURVE_PRESETS` maps curve names to bezier control points (from easings.net)
- Custom curves can override presets via BezierEditor component

#### Palette Generation (`src/utils/paletteGenerator.ts`)
- Exports: `exportAsCSS()`, `exportAsJSON()`, `exportAsPlainText()`, `exportAsSVG()`
- SVG export creates 40×40 rounded rectangles with 8px spacing, timestamped for Figma pasting

### State Management

Single-level React state in App.tsx:
- `colorState` (ColorState interface) - all generation parameters
- `activeGraph` - which visualization is displayed
- `collapsedSections` - UI collapse state for control sections
- `language` - i18n language code for translations

No external state management library; all components receive data via props.

### Components Structure

- **App.tsx**: Root component with three-panel layout (controls, graph, preview)
- **BezierEditor.tsx**: Interactive cubic-bezier curve editor with draggable handles
- **Graph.tsx**: SVG-based visualization switching between hue/saturation/brightness/luminance/sat×bri
- **PalettePreview.tsx**: Color swatches with contrast ratios, WCAG badges, and copy-to-clipboard
- **ExportModal.tsx**: Tabbed export dialog (CSS/JSON/Text/Figma)
- **Tooltip.tsx**: Reusable tooltip component

### Internationalization

Managed in `src/utils/translations.ts`:
- `languageOptions` array defines supported languages
- `translations` object contains all UI strings per language
- `getTranslation(lang, key)` retrieves translated text
- Add new languages by extending both `languageOptions` and `translations`

## Technical Details

### Color Interpolation
- Hue interpolation respects the circular nature of the color wheel (0° = 360°)
- `longPath` boolean toggles between shortest and longest hue transition paths
- Saturation has a `rate` multiplier (0-2) to control intensity
- Y values in bezier curves can exceed [0,1] for overshoot effects (Back easing)

### Pinned Colors
- Users can pin colors in two ways:
  1. **Click in graph/palette**: Pins the clicked color using luminance-based positioning
  2. **Type hex in sidebar input**: Pins using luminance-based positioning
- Unpinning:
  - Click the pinned color in graph/palette to unpin it
  - Clear the hex input field to unpin
- Positioning uses WCAG luminance to find optimal position in the palette
  - Calculates luminance using: `0.2126×R + 0.7152×G + 0.0722×B` (with gamma correction)
  - Inserts pinned color at the position where its luminance fits in the descending sequence
  - This ensures the pinned color appears in the visually correct position based on perceived brightness
- Pinned color **repositions automatically** when HSB parameters or curves change (not fixed to index)
- Pinned color **replaces** (not inserts) the generated color at that position—maintains natural interpolation spacing
- The replaced color is conceptually a "ghost" that preserves curve continuity
- Total step count remains constant whether or not a color is pinned
- Pinned colors are visually indicated with a 2px white border in the palette and a white ring in the graph
- User must manually adjust curves/parameters to make smooth transitions through pinned color

### Accessibility Calculations
- Contrast ratios use WCAG 2.0 relative luminance formula
- Each color shows contrast against both white and black backgrounds
- WCAG levels: AAA (≥7), AA (≥4.5), A (≥3), Fail (<3)

### Build System
- Vite with React plugin for fast HMR
- TypeScript with strict mode and comprehensive linting rules
- Target: ES2020, bundler module resolution
- No special configuration needed; standard Vite React setup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jpfaraco)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jpfaraco)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
