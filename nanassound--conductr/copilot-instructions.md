## conductr

> Browser-based MIDI instrument that generates four tracks of real-time MIDI output,

# Conductr

Browser-based MIDI instrument that generates four tracks of real-time MIDI output,
driven by player input and shaped by an Opus 4.6 AI musical director.
Outputs MIDI via Web MIDI API — no audio synthesis. Connect to any DAW or synth.

## Monorepo Layout

Two independent packages, no top-level build orchestrator:

- **`engine/`** — Pure C library (`libgenseq`). Builds natively via `make` or to WASM via Emscripten. See `engine/CLAUDE.md`.
- **`web/`** — Vite browser app. Consumes WASM output from engine. See `web/CLAUDE.md`.

## Three-Timescale Architecture

| Layer | Timescale | Where |
|---|---|---|
| libgenseq (note generation) | Every step (~15 ms at 120 bpm) | WASM, deterministic |
| Performance Analyzer (feature extraction) | Every 1-2 bars (~2-4 s) | JavaScript |
| Opus 4.6 Musical Director (arrangement) | Every 2-4 bars (~4-8 s) or on voice cmd | API call, async |

**Critical invariant:** the engine never waits for Opus. Opus shapes the *next* phrase, not the current note.

## MIDI Channel Map

| Channel | Track |
|---|---|
| Ch 10 | Drums (GM standard) |
| Ch 1 | Bass |
| Ch 2 | Melody |
| Ch 3 | Harmony |
| Ch 5 | User input echo (optional) |

## Development Phases

1. **Core Engine** — C code, native build, unit tests (`make test` passes)
2. **Browser Playback** — WASM build, JS bridge, Web MIDI output, minimal page
3. **Interactive Input** — Pads, MIDI input, performance analyzer, UI
4. **AI Director** — Opus integration, text commands, fallback director
5. **Voice Control & Polish** — Speech API, error handling, docs

## MIDI Clock Sync

Conductr can slave to an external MIDI clock (e.g., from a DAW). Three sync modes:

- **Internal** — Web Worker clock (default)
- **Auto** — Internal clock + auto-detect external; switches on first 0xF8, falls back if clock lost
- **External** — Waits for DAW transport; follows external clock exclusively

Key MIDI messages handled: Clock (0xF8), Start (0xFA), Stop (0xFC), Continue (0xFB), Song Position Pointer (0xF2). 24 PPQN standard, 6 ticks per sixteenth-note step.

## Key References

- `docs/overview.md` — Full design doc (architecture, algorithms, API, schemas)
- `engine/CLAUDE.md` — Build commands, C API, data structures, algorithms
- `web/CLAUDE.md` — Dev setup, JS modules, timing, Opus integration

---
> Source: [nanassound/conductr](https://github.com/nanassound/conductr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
