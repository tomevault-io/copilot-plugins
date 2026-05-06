## typlit

> **Typlit** is a TypeScript/JSX library that replicates [Pixlet's](https://github.com/tidbyt/pixlet) widget system for 64x32 pixel LED displays (Tidbyt), replacing Starlark with JSX syntax.

# Typlit Development Guide

## Project Overview

**Typlit** is a TypeScript/JSX library that replicates [Pixlet's](https://github.com/tidbyt/pixlet) widget system for 64x32 pixel LED displays (Tidbyt), replacing Starlark with JSX syntax.

### Goals

- 1:1 API compatibility with Pixlet's render widgets
- Pure TypeScript implementation (no Canvas dependency for rendering)
- JSX syntax for defining layouts
- Server-side rendering with Bun

### Architecture

```
src/
├── core/
│   ├── PixelBuffer.ts    # RGBA pixel buffer (no Canvas)
│   ├── render.ts         # JSX tree resolution & widget rendering
│   ├── color.ts          # Color parsing & types
│   └── types.ts          # Shared types (Widget, Bounds, etc.)
├── fonts/
│   ├── parser.ts         # BDF font parser
│   ├── builtin.ts        # Built-in font loading
│   └── data/             # BDF font files
├── widgets/
│   ├── Root.ts           # Root container (64x32, delay)
│   ├── Box.ts            # Centered container with background
│   ├── Row.ts            # Horizontal layout
│   ├── Column.ts         # Vertical layout
│   ├── Stack.ts          # Z-axis stacking
│   ├── Text.ts           # Single-line text
│   ├── WrappedText.ts    # Multi-line wrapped text
│   ├── Image.ts          # PNG image decoder
│   ├── Marquee.ts        # Scrolling animation
│   ├── Padding.ts        # Padding wrapper
│   └── Animation.ts      # Frame-based animation
├── jsx-runtime.ts        # JSX runtime
└── index.ts              # Main exports
```

### Key Concepts

1. **Widget Interface**: All widgets implement `Widget` with:

   - `paintBounds(bounds, frameIdx)` - Returns actual size
   - `paint(buf, bounds, frameIdx)` - Renders to PixelBuffer
   - `frameCount()` - Animation frame count

2. **PixelBuffer**: Pure TypeScript RGBA buffer with:

   - Clipping regions
   - Transform stack (push/pop/translate)
   - Alpha blending

3. **BDF Fonts**: Bitmap fonts parsed from BDF format
   - Baseline positioning: `y = height - descent - offset`

### Running

```bash
bun run dev      # Start dev server at http://localhost:3456
bun test         # Run all tests
```

---

## Session Notes

Take notes during development sessions in the `notes/` folder. Create files named by date (e.g., `notes/2024-12-09.md`). Include:

- What was worked on
- Decisions made and rationale
- Issues encountered and solutions
- Next steps

### Current State (as of 2024-12-09)

- 85 tests passing across 12 files
- 2 example apps: GitHub Stargazer, Beat Time
- All core widgets implemented

### Known Issues

- None currently

### Next Steps

- Port more Tidbyt community apps
- Add more widget tests
- Consider adding Circle, Plot, or other missing widgets

NEVER RUN THE DEV SERVER

---

## ASCII Renderer for Debugging

Use the ASCII renderer to visualize widgets in the terminal without needing a browser. This is useful for verifying widget output and iterating on designs.

### Basic Usage

```typescript
import { render, printBuffer, loadBuiltinFonts } from './src';

await loadBuiltinFonts();

const frames = await render(<MyWidget />);
printBuffer(frames[0], { title: 'My Widget' });
```

### Available Functions

- `toAscii(buffer, options?)` - Convert to ASCII string (uses ` .:-=+*#%@` for brightness)
- `toBlocks(buffer, options?)` - Convert using Unicode half-blocks (`▀▄`) for higher fidelity
- `printBuffer(buffer, options?)` - Print with a nice box border
- `printBlocks(buffer, options?)` - Print blocks with border

### Sampling Animation Frames

To check each page of a multi-page animation:

```typescript
const frames = await render(<MyAnimation />);

// Sample specific frames
const samples = [
  { frame: 15, label: 'Page 1' },
  { frame: 35, label: 'Page 2' },
  { frame: 65, label: 'Page 3' },
];

for (const { frame, label } of samples) {
  printBuffer(frames[frame], { title: label });
}
```

To watch a specific section frame-by-frame:

```typescript
// Watch frames 130-149 (e.g., outro animation)
for (let i = 130; i < 150; i++) {
  printBuffer(frames[i], { title: `Frame ${i}` });
}
```

### Character Brightness Mapping

The default charset maps brightness levels:
- ` ` (space) = black/transparent
- `.:-` = dark grays
- `=+*` = mid grays  
- `#%@` = light to white

---
> Source: [RhysSullivan/typlit](https://github.com/RhysSullivan/typlit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
