## lipsync-engine

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

`@beer-digital/lipsync-engine` is a zero-dependency, renderer-agnostic streaming lip-sync engine for browser-based 2D animation. It performs real-time viseme detection from streaming audio via AudioWorklet + Web Audio API. ~15KB minified.

## Commands

```bash
npm run build          # Full build: vite build + copy worklet + copy types
npm run dev            # Dev server with demo at http://localhost:3000
npm run test           # Run tests once (vitest run)
npm run test:watch     # Run tests in watch mode (vitest)
npm run lint           # ESLint on src/
```

Build outputs to `dist/` as ESM (`.js`) and CJS (`.cjs`) with sourcemaps. The worklet file is copied separately (not bundled).

## Architecture

### Audio Pipeline

```
Audio Input → AudioWorklet (ring buffer, gapless playback) → AnalyserNode (FFT) → FrequencyAnalyzer (viseme classification) → 'viseme' events
```

**Three input modes:**
1. **Streaming PCM** (`feedAudio`) — for TTS APIs (OpenAI Realtime, ElevenLabs). Samples go through the worklet for buffered playback.
2. **MediaStream** (`attachStream`) — microphone/WebRTC. Connected directly to AnalyserNode (no playback, to avoid feedback).
3. **HTMLMediaElement** (`attachElement`) — audio/video elements. Connected through AnalyserNode to destination.

### Key Modules

- **`LipSyncEngine`** (`src/core/LipSyncEngine.js`) — Main orchestrator. Manages AudioContext, wires up the pipeline (WorkletNode → AnalyserNode → GainNode → destination), handles worklet message protocol. Extends `EventEmitter`.

- **`FrequencyAnalyzer`** (`src/analyzers/FrequencyAnalyzer.js`) — Core viseme detection. Reads FFT data from AnalyserNode, extracts 5 frequency band energies (sub/low/mid/high/veryHigh), classifies into one of 15 extended visemes via heuristic rules. Applies temporal smoothing (EMA) and hold-off to prevent flickering. Returns `VisemeFrame` with shape params `{open, width, round}` interpolated during transitions.

- **`streaming-processor.js`** (`src/worklets/streaming-processor.js`) — AudioWorkletProcessor. **Must be self-contained** (no imports). Implements a ring buffer for gapless streaming playback with fade-in/out, auto-start threshold, buffer underrun detection, and position reporting via MessagePort.

- **`visemes.js`** (`src/core/visemes.js`) — Two viseme sets: Extended (15 shapes, Oculus/MPEG-4 compatible) and Simple (6 shapes, Preston Blair A-F). Contains `VISEME_SHAPES` (normalized `{open, width, round}` for procedural rendering), `PHONEME_TO_VISEME` (ARPABET mapping), and `TRANSITION_WEIGHTS` (coarticulation blending speeds).

### Renderers (all interchangeable, receive `VisemeFrame`)

- **`SVGMouthRenderer`** — Procedural SVG paths driven by `{open, width, round}` shape params. No sprite sheet needed.
- **`CanvasRenderer`** — Draws frames from a sprite sheet image using a viseme-to-frame-index map.
- **`CSSClassRenderer`** — Sets data attributes and CSS classes on a DOM element. Also sets CSS custom properties (`--lip-open`, `--lip-width`, `--lip-round`, `--lip-intensity`).

### Utilities

- **`RingBuffer`** — Float32 ring buffer with overflow support (exported but also reimplemented inline in the worklet since the worklet can't import).
- **`EventEmitter`** — Lightweight typed emitter with wildcard (`*`) support. `on()` returns an unsubscribe function.
- **`audio-utils.js`** — PCM format conversions (`int16ToFloat32`, `base64ToInt16`, etc.), `calculateRMS`, `extractBandEnergies`, `resample` (linear interpolation), `smoothValue` (EMA).

## Key Constraints

- The AudioWorklet file (`streaming-processor.js`) must be **fully self-contained** — no ES module imports. It's copied to dist as-is, not bundled by Vite.
- `engine.init()` must be called after a user gesture (browser AudioContext policy).
- The dev server sets `Cross-Origin-Opener-Policy: same-origin` and `Cross-Origin-Embedder-Policy: require-corp` headers for SharedArrayBuffer support.
- TypeScript types are maintained manually in `src/types.d.ts` and copied to `dist/index.d.ts` during build.
- Viseme detection is heuristic/frequency-based (not phoneme-aligned). Classification in `FrequencyAnalyzer._classifyViseme()` uses band energy ratios with hardcoded thresholds.
- `feedAudio` transfers Float32Array buffer ownership to the worklet via `postMessage` with transferable for zero-copy performance.

---
> Source: [Amoner/lipsync-engine](https://github.com/Amoner/lipsync-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
