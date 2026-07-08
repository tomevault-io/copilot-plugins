## smart-voicing-chords

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev          # Start dev server (Vite)
npm run build        # Production build
npm run lint         # ESLint
npm run test         # Run tests once (Vitest)
npm run test:watch   # Run tests in watch mode
```

Run a single test file:
```bash
npx vitest run src/test/music.test.ts
```

## Architecture

This is a guitar/piano chord reference and practice app. The stack is React 18 + TypeScript + Vite + Tailwind + shadcn-ui.

### Routing (`src/App.tsx`)

| Route | Page | Purpose |
|---|---|---|
| `/` | `Tutorial` | Onboarding |
| `/guitar` | `Index` | Chord browser |
| `/engine` | `VoicingEngine` | Smart voicing progression tool |
| `/songs` | `Songs` | Song chord sheets |
| `/piano` | `PianoChords` | Piano chord diagrams |
| `/tab` | `TabEditor` | Guitar tab editor |
| `/progressions` | `Progressions` | Chord progression builder |
| `/quiz` | `Quiz` | Chord quiz |
| `/scales` | `Scales` | Scale fretboard visualizer |

A persistent `<BottomNav />` and `<AudioUnlockOverlay />` wrap all routes.

### Data layer (`src/data/`)

- **`chords.ts`** — Static guitar chord database. `ChordVoicing` has `positions[]` (fret numbers, -1=muted, 0=open), `fingers[]`, optional `barres[]`, and `baseFret`. 10 voicings per chord. `getAllChordsWithCustom()` merges in user-created custom chords from localStorage.
- **`pianoChords.ts`** — Piano chord data (MIDI note sets).
- **`songs.ts`** — Hardcoded song chord sheets.
- **`leadsheet.ts`**, **`tab.ts`** — Leadsheet and tab data types/helpers.

### Engine (`src/engine/voicingEngine.ts`)

Core algorithm for the `/engine` (Smart Voicing) page. Given a chord progression, it scores and selects voicings that minimize hand movement between chords using `fretDistance()`. Voicings are classified by fret region (low ≤4, mid ≤8, high >8) and returned as `VoicingPath[]` with three positional options.

### Music theory library (`src/lib/`)

- **`music.ts`** — Note spelling, chord tone names, MIDI-to-note-name mapping. Used by tests.
- **`synth.ts`** — Web Audio API synth with selectable instruments. Called via `playWithInstrument()`.
- **`chordDetection.ts`** — Detect chord name from a set of notes.
- **`progressions.ts`** — Progression analysis helpers.
- **`scales.ts`** — Scale interval data.
- **`substitutions.ts`** — Chord substitution suggestions.

### Audio (`src/hooks/`)

Audio is gated behind `AudioUnlockOverlay` (requires a user gesture). `useAudioContext.ts` manages a shared `AudioContext` via `withSharedAudioContext()`. `useChordPlayer` and `usePianoPlayer` convert voicing positions to MIDI frequencies and play via `synth.ts`. Settings (volume, tone, instrument) are persisted via `useAudioSettings`.

### Persistence

User data is stored in `localStorage`:
- Custom chords (merged in `getAllChordsWithCustom()`)
- Favorites (`useFavorites`)
- Recent chords (`useRecentChords`)
- Audio settings (`useAudioSettings`)
- Theme (`useTheme`)
- Left-handed mode (`useLeftHanded`)

### Path alias

`@/` maps to `src/`. Used throughout.

### Tests

Tests live in `src/test/` and alongside source files (`*.test.ts`). The test environment is jsdom. Current tests cover music theory spelling (`music.test.ts`) and example utilities (`example.test.ts`).

---
> Source: [MatitaPere/smart-voicing-chords](https://github.com/MatitaPere/smart-voicing-chords) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
