## stringthing

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server
npm run build    # TypeScript check + production build
npm run lint     # ESLint
npm run preview  # Preview production build
```

## Architecture

This is a guitar/bass restringing assistant web app built with React 19, TypeScript, and Vite.

### Core Features

- **Tuner** (`src/components/Tuner.tsx`) - Chromatic tuner using Web Audio API with autocorrelation pitch detection via `usePitchDetection` hook. Shows detected note, octave, frequency, and cents deviation with visual tuning indicator.

- **Restring Guide** (`src/components/RestringGuide.tsx`) - Step-by-step restringing instructions for guitar and bass with progress tracking. Includes special instructions for Floyd Rose systems.

- **String Calculator** (`src/components/StringCalculator.tsx`) - Calculates string slack and winding turns based on scale length, tuner peg type, and distance from nut to tuners.

### Audio Processing

The pitch detection (`src/hooks/usePitchDetection.ts`) uses autocorrelation algorithm for frequency detection. Key frequency reference: A4 = 440Hz. Standard tuning frequencies for guitar/bass are defined in the Tuner component.

### Styling

Each component has a corresponding CSS file. Dark theme with accent color #00d4ff (cyan) and #00ff88 (green for "in tune" states). Mobile-responsive.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Shadester) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
