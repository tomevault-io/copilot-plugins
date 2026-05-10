## fuzzboxphysics

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Fuzzbox Physics is an educational course website (CalArts CSCM 373-01) that traces sound from air vibration through transduction, amplification, and intentional distortion. The core deliverables are interactive browser-based audio labs.

## Running the Project

```bash
node server.js        # starts at http://localhost:3000, no install needed
```

Any HTTP server works (e.g., `python3 -m http.server`). The Node server adds one API endpoint: `GET /api/wavs` returns a JSON list of files in `wavs/` so labs can build file-loading buttons dynamically. There are no build steps, no package manager, and no dependencies.

## Architecture

### Web Labs (`app/`)

Each lab is a **self-contained single-page HTML file** with embedded CSS and inline JavaScript. No frameworks, no module bundler, no external JS dependencies (except Strudel for the drift piece). All audio processing uses the **Web Audio API**.

- **fuzzbox-lab.html** — Waveform distortion with 7 clipping algorithms, real-time oscilloscopes/spectrum/spectrogram, reverb engine with synthetic impulse responses
- **fuzzbox-sculptor.html** — Spectral editor: STFT analysis → 2D canvas painting → ISTFT resynthesis. Contains its own radix-2 FFT implementation
- **fuzzbox-octavia.html** — Stage-by-stage Roger Mayer Octavia simulation (5 signal chain stages, each with before/after oscilloscope)
- **fuzzbox-reverb.html** — Room acoustics lab: record a clap, synthesize impulse response, match with 7-band Schroeder decay curves
- **fuzzbox-drift.html** — Generative phasing piece (14 sine voices drifting through D major). Uses Strudel library

### Audio Processing Patterns

Labs follow a consistent Web Audio node graph: **Source → Gain → AnalyserNode → Processing → AnalyserNode → Output**. Wet/dry mixing uses parallel gain nodes. `ScriptProcessorNode` is used for per-sample distortion (educational clarity over AudioWorklet). Canvas elements visualize audio data in real time (oscilloscopes, FFT spectra, scrolling spectrograms, VU meters).

### Visual Design

CRT/oscilloscope aesthetic: phosphor green (#39ff14) on dark backgrounds (#0a110a), scan-line patterns, glowing text-shadows. Fonts: Bebas Neue (headings), Share Tech Mono (UI), VT323 (monospace details) — loaded from Google Fonts.

### Server (`server.js`)

112-line zero-dependency Node.js static file server. Routes `/` to `app/index.html` and serves `app/*.html` at root-level URLs. Includes MIME type detection and directory traversal prevention.

## Key Directories

- `app/` — interactive web labs (the main codebase)
- `course/` — syllabus and planning documents
- `lectures/` — weekly lecture notes (W01–W14)
- `listening/` — artist studies and media files
- `hardware/` — breadboard circuit photos and schematics
- `wavs/` — audio files served to labs via `/api/wavs`
- `sketches/` — code experiments (Strudel generative music)
- `texts/` — reference PDFs (git-ignored, large files)

## Conventions

- Labs are standalone HTML — keep all CSS and JS inline within each file
- Audio files dropped into `wavs/` are automatically available in labs
- Large media (video, PDFs, Canvas exports) are git-ignored; see `.gitignore`
- Input modes across labs: oscillator, microphone, and file upload (drag-and-drop)

---
> Source: [douglasgoodwin/FuzzboxPhysics](https://github.com/douglasgoodwin/FuzzboxPhysics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
