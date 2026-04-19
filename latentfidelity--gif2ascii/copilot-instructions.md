## gif2ascii

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Build & Run Commands

### Web App
- **Development server**: `npm run dev` (runs on port 3000)
- **Production build**: `npm run build`
- **Preview production build**: `npm run preview`

### CLI Tool
- **Build CLI**: `npm run build:cli`
- **Link for global usage**: `npm link` (after building)
- **Run directly**: `node cli/dist/index.js <input> [options]`
- **Run after linking**: `gif2ascii <input> [options]`

No linting or test scripts are configured.

## Environment Variables

Create a `.env` file in the project root for optional Tenor GIF search:
```
VITE_TENOR_API_KEY=your_tenor_key_here
```

## Architecture

Gif2Ascii is a React-based web app and Node.js CLI tool that converts GIF animations to ASCII art, with export capabilities.

### Core Flow

1. **FileUpload** accepts GIF/PNG/JPEG/WebP files via drag-drop or file picker
2. **TenorSearch** provides inline GIF search (requires API key in `.env`)
3. **App** manages configuration state (density, colors, font aspect ratio, output dimensions)
4. **AsciiPlayer** handles:
   - GIF parsing via `gifuct-js` (decompresses frames with proper disposal handling)
   - Static image loading for PNG/JPEG/WebP (detected via magic bytes)
   - Frame composition on an offscreen canvas (handles GIF disposal types 2/3)
   - ASCII conversion via `services/asciiUtils.ts`
   - Canvas rendering with JetBrains Mono font
   - Export to GIF/MP4 (animated) or PNG (static)

### Key Files

Source files are in the project root (no `src/` directory):

- `App.tsx` - Main component with all render settings state
- `components/AsciiPlayer.tsx` - Core playback/rendering/export logic (~1250 lines)
- `components/TenorSearch.tsx` - Tenor API integration with debounced search and infinite scroll
- `services/asciiUtils.ts` - Image-to-ASCII conversion algorithms
- `services/postProcessing.ts` - CRT and visual post-processing effects
- `services/animationEffects.ts` - Animated effects (matrix rain, wave, typing)
- `services/videoExport.ts` - MP4/WebM video export with WebCodecs
- `types.ts` - TypeScript interfaces (AsciiConfig, AsciiFrame, AppState)
- `cli/index.ts` - CLI tool source (bundled with esbuild to `cli/dist/index.js`)

### ASCII Conversion Pipeline

`resizeAndGetImageData()` → scales source with high-quality smoothing + optional sharpening → `convertToAscii()` → density-calibrated character mapping + optional dithering

**Quality Features:**
- **Character Density Calibration**: Characters are mapped by their actual visual density (pixel coverage), not linear position in the character string
- **Floyd-Steinberg Dithering**: Optional error-diffusion dithering for smoother gradients in ASCII output
- **Pre-sharpening Filter**: Unsharp mask applied before downsampling to preserve edge details
- **High-Quality Downsampling**: Uses `imageSmoothingQuality: 'high'` for better detail retention

Character presets in `asciiUtils.ts`:
- **Standard**: `@%#*+=-:. ` (10 chars, default)
- **Dense**: 70 chars from `$` to space (high detail)
- **Blocks**: `█▓▒░ ` (Unicode block characters)
- **Simple**: `@#*+:. ` (minimal set)
- **Binary**: `10` (digits only)
- **Braille**: Unicode braille patterns
- **Numeric**: `0123456789`
- **Katakana**: Japanese katakana characters
- **Dots**: `●◉◎○· ` (circle variants)
- **Shades**: `█▓▒░· ` (block shading)
- **Crosses**: `╬╫╪┼+· ` (box drawing)
- **Stars**: `★☆✦✧· ` (star variants)
- **Slashes**: `#/\|-:. ` (diagonal/linear)

Characters map dark to light. The `invert` flag reverses this mapping. The `useSourceColor` option preserves original pixel colors for each ASCII character.

### Image Adjustments

The conversion pipeline supports real-time image adjustments applied before ASCII conversion:
- **Brightness**: -300 to +300 (additive adjustment)
- **Contrast**: -300 to +300 (factor around midpoint 128)
- **Saturation**: -300 to +300 (-100 = grayscale)
- **Sharpness**: 0 to 300 (unsharp mask strength for edge enhancement)
- **Dithering**: Toggle for Floyd-Steinberg error diffusion (smoother gradients)

### Color Palettes

Retro color palette quantization (`services/asciiUtils.ts`):
- **None**: Original colors
- **Gameboy**: 4-color green monochrome
- **Commodore 64**: 16-color C64 palette
- **CGA**: 4-color cyan/magenta/white
- **Neon**: Vibrant neon colors
- **Synthwave**: Purple/pink/cyan retrowave
- **Amber Monitor**: Amber phosphor terminal
- **Green Monitor**: Green phosphor terminal
- **Grayscale**: 5-level grayscale

### Post-Processing Effects

CRT and visual effects applied after ASCII rendering (`services/postProcessing.ts`), all 0-300 intensity:
- **CRT Scanlines**: Horizontal line overlay simulating CRT monitors
- **Phosphor Glow**: Bloom/glow effect using blur and additive blending
- **RGB Split**: Chromatic aberration with horizontal RGB channel offset
- **Static Noise**: Random noise overlay
- **Vignette**: Radial edge darkening
- **Flicker**: Subtle brightness oscillation for CRT effect

### Animation Effects

Animated overlays and effects requiring continuous updates (`services/animationEffects.ts`):
- **Matrix Rain**: Falling Japanese/alphanumeric characters overlay (0-300 intensity)
- **Wave Distortion**: Horizontal sinusoidal pixel displacement (0-300 intensity)
- **Typing Reveal**: Typewriter-style character reveal with blinking cursor (static images)

These effects run in a continuous render loop when enabled, even for static images or paused GIFs.

### GIF Frame Handling

Uses a composition canvas pattern: each frame's patch is drawn onto a persistent canvas, respecting GIF disposal types. The composition canvas represents the current visual state that gets converted to ASCII.

### Video Export

Video export uses frame-by-frame encoding via `services/videoExport.ts`:
- **MP4 (H.264)**: Uses `mediabunny` library with WebCodecs API. Works in Chrome, Edge, Safari.
- **WebM (VP9)**: Fallback for Firefox and older browsers via MediaRecorder API.

Export settings: optional 2x resolution toggle (controlled by `export2x` prop), 8 Mbps bitrate, 30 fps. GIF exports use Floyd-Steinberg dithering for smoother color gradients.

### Tenor Proxy

In development/preview, Vite proxies `/tenor` to `https://tenor.googleapis.com` to avoid CORS issues. In production (GitHub Pages), API calls go directly to Tenor's endpoint. See `vite.config.ts`.

### Styling

Uses Tailwind CSS via CDN (in index.html). Fonts: Inter for UI, JetBrains Mono for ASCII rendering.

### Path Alias

`@` is aliased to the project root in `vite.config.ts`.

### Deployment

Production build uses `/gif2ascii/` base path. GitHub Actions automatically deploys to GitHub Pages on push to `main` (see `.github/workflows/deploy.yml`).

## CLI Tool (Experimental)

The CLI (`cli/index.ts`) is a test/experimental feature for terminal-based ASCII art playback. It provides two rendering modes:

### ASCII Mode (default)
Renders colored ASCII characters using ANSI escape codes. Uses alternate screen buffer to prevent scrollback pollution. Auto-scales to fit terminal dimensions unless `--no-fit` is specified.

### Sixel Mode
Renders actual pixel graphics for terminals that support Sixel (iTerm2, Windows Terminal, mlterm, foot). First renders ASCII art to a canvas, then encodes to Sixel format.

### CLI Key Features
- Accepts local files or URLs (GIF, PNG, JPEG, WebP)
- ANSI true color (24-bit) support with `--source-color`
- Auto-scales to fit terminal dimensions (disable with `--no-fit`)
- Single frame mode with `--frame <n>`
- Export formats via `-o`:
  - `.sh` - Bash script with animation loop
  - `.ps1` - PowerShell script with animation loop
  - `.ans` - Single frame with ANSI escape codes
  - `.txt` - Plain text (no colors)
- Uses `@napi-rs/canvas` for Node.js canvas operations
- Uses `commander` for argument parsing
- Bundled with esbuild (ESM format, external `@napi-rs/canvas`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/latentfidelity) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
