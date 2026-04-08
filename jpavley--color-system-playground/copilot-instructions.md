## color-system-playground

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Color System Playground is a single-page web app for designing HSB color palettes for Bit Quiz 16 game levels. It's a companion tool to the main iOS game project at `~/Developer/Card-Flip-Animation`.

## Running the App

Open `index.html` directly in a browser:

```bash
open index.html
```

No build tools, dependencies, or server required.

## Architecture

Single-file architecture (`index.html`) containing:

- **CSS** (lines ~7-385): Dark theme styling, 3-column responsive grid layout, slider and swatch components
- **HTML** (lines ~386-630): Level sidebar, 4 color panels (Accent/Primary/Secondary/Background), preview, export section
- **JavaScript** (lines ~631-end): State management, localStorage caching, HSB↔HSL conversion, palette calculation

### Key Data Structures

**Levels array**: 8 game levels with equidistant default hues (45° apart starting from 35°):

- Nibble (4-bit, 35°), Ascii (7-bit, 80°), Byte (8-bit, 125°), Nobble (10-bit, 170°)
- Tribble (12-bit, 215°), Gobble (14-bit, 260°), HighColor (16-bit, 305°), Monster (32-bit, 350°)

**State object**: 12 HSB values (4 colors × 3 components) plus levelName and showCRT flag

**localStorage cache**: Per-level color settings persisted under key `colorSystemPlayground`

### Color Transformation Defaults

When resetting, each color derives from the level's accent hue:

- **Accent**: Full saturation/brightness (hue, 100%, 100%)
- **Primary**: Same hue, low saturation (hue, 20%, 100%)
- **Secondary**: Complementary hue (+180°), muted (bgHue, 20%, 70%)
- **Background**: Complementary hue, full saturation, medium brightness (bgHue, 100%, 50%)

## Export Format

The Export button generates Xcode Asset Catalog JSON (`Contents.json`) for each color, ready to copy into `.colorset` folders in the iOS project.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jpavley)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jpavley)
<!-- tomevault:4.0:copilot_instructions:2026-04-07 -->
