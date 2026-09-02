## algorhtym

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev      # Start development server (Vite) at localhost:5173
npm run build    # Production build to dist/
npm run preview  # Preview production build
```

## Architecture

AlgoRhythm is a vintage turntable-style music player built with React + Vite. Users drag and drop local audio files (MP3, WAV, OGG, FLAC, M4A, AAC, WebM) and play them through an animated turntable interface.

### State Management

All audio state flows through `src/context/AudioContext.jsx` using React's `useReducer`. The context provides:
- Playlist management (tracks array, current index)
- Playback state (isPlaying, volume, currentTime, duration)
- A shared `audioRef` pointing to the HTMLAudioElement

Components consume state via the `useAudio()` hook.

### Audio Playback

`src/hooks/useAudioPlayer.js` manages the actual HTMLAudioElement:
- Creates/controls the Audio instance
- Attaches event listeners (timeupdate, ended, loadedmetadata)
- Provides `play()`, `pause()`, `stop()`, `seek()`, `seekPercent()` functions
- Auto-advances to next track on song end

The hook must be called once in a component (currently in `App.jsx`'s `Player` component) to initialize the audio element.

### Turntable Animation

The turntable visual consists of two animated components:
- `VinylDisc.jsx` - CSS animation spinning at 33⅓ RPM (1.8s rotation). Controlled by `isPlaying` state via `vinyl-disc--spinning` class.
- `ToneArm.jsx` - Uses Framer Motion spring animation. Rotates from -30° (rest) to 0° (playing) based on `isPlaying && currentTrack`.

### Styling

SCSS with BEM methodology. Key files:
- `src/assets/styles/_variables.scss` - Apple-inspired minimal design tokens: pure white/black monochrome palette, SF Pro font stack, precise spacing scale, compact sizing optimized for viewport fit
- `src/assets/styles/_animations.scss` - Subtle keyframes (spin, pulse, shine) with Apple-like cubic-bezier easing
- `src/assets/styles/global.scss` - Global layout, typography, and responsive breakpoints
- Each component has co-located `.scss` file

The design prioritizes fitting all content in viewport without scrolling at 100% zoom through compact spacing and reduced component sizes.

### File Upload

`src/hooks/useFileUpload.js` handles drag-and-drop and file input:
- Validates audio MIME types and extensions
- Creates blob URLs via `URL.createObjectURL()`
- Adds tracks to playlist via context

## Key Patterns

- Components use index.js barrel exports (e.g., `import Turntable from './components/Turntable'`)
- Control components (Play, Stop, Skip, Progress, Volume) share `Controls.scss` stylesheet
- Animation state is driven by context's `isPlaying` boolean - components read it and apply CSS classes or Framer Motion props

---
> Source: [JuanJvilla09/AlgoRhtym](https://github.com/JuanJvilla09/AlgoRhtym) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-02 -->
