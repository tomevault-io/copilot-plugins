## conductr

> Deterministic MIDI pattern generation library. Zero dependencies, no heap allocation.

# engine/ — libgenseq (Pure C)

Deterministic MIDI pattern generation library. Zero dependencies, no heap allocation.

## Build Commands

```bash
# Native build + tests
make
make test

# WASM build (output → web/public/wasm/)
source ~/emsdk/emsdk_env.sh
emcc src/*.c -I include -O2 \
  -s EXPORTED_FUNCTIONS="['_genseq_init','_genseq_tick','_genseq_start','_genseq_stop','_genseq_feed_note','_genseq_set_bpm','_genseq_set_scale','_genseq_set_swing','_genseq_regenerate','_genseq_set_seed','_genseq_drum_params','_genseq_bass_params','_genseq_melody_params','_genseq_harmony_params','_malloc','_free']" \
  -s EXPORTED_RUNTIME_METHODS="['ccall','cwrap','HEAPU8']" \
  -s MODULARIZE=1 -s EXPORT_NAME="createGenseqModule" -s TOTAL_MEMORY=65536 \
  -o ../web/public/wasm/genseq.js
```

All exported C functions must be marked `EMSCRIPTEN_KEEPALIVE`.

## Memory Budget

~3.6 KB total static memory, no heap allocation:
- Context: ~16 B | Patterns (x8): ~2,640 B | Params: ~80 B
- State: ~16 B | Templates: ~768 B | Output buffer: ~48 B | PRNG: 4 B

## Source Files

| File | Purpose |
|---|---|
| `include/genseq.h` | Public API, all type definitions |
| `src/genseq.c` | Init, tick loop, pattern swap, transport |
| `src/gen_drums.c` | Euclidean rhythm drum generator |
| `src/gen_bass.c` | Template + scale-constrained bass generator |
| `src/gen_melody.c` | Constrained random-walk melody generator |
| `src/gen_harmony.c` | Diatonic interval offset harmony generator |
| `src/music_utils.c` | Scale degree/MIDI conversion, quantize, chord-tone snap |
| `src/rng.c` | xorshift32 PRNG (deterministic: same seed = same pattern) |

## Key Data Structures

- **`Genseq`** — Main engine instance. Holds context, double-buffered patterns, all generator params/state, PRNG seed, output buffer, and control flags.
- **`GenseqContext`** — Shared musical state: root, scale_mask, chord_tones, bpm, step, steps_per_bar, swing, bar_count.
- **`GenseqEvent`** — Single MIDI event: note, velocity, step, duration, flags (accent/tie/slide).
- **`GenseqPattern`** — Array of MAX_STEPS events + length, channel, octave_base, note_range.
- **`Genseq{Drum,Bass,Melody,Harmony}Params`** — Per-track generator parameters (set by JS/Opus).

## Scale Mask Encoding

12-bit bitmask. Check note in scale: `scale_mask & (1 << (note % 12))`

```
Major: 0xAB5  Minor: 0x5AD  Dorian: 0x5B5
Mixolydian: 0x56B  Pentatonic: 0x295  Blues: 0x69D
```

## Algorithms

- **Drums** — Euclidean rhythms (Bresenham-style). Each layer: E(hits, steps) + rotation. Metric accents + velocity jitter.
- **Bass** — Predefined rhythm templates (8 styles). Pitch via weighted scale degree selection; movement param controls pedal vs walking.
- **Melody** — Constrained random walk. Direction reversal probability increases with run length. Gap-fill principle. Chord-tone gravity on strong beats. Phrase-boundary resolution.
- **Harmony** — Diatonic offset from melody. Configurable interval (3rd/5th) above/below. Optional rhythm thinning to strong beats.

## Pattern Swap

Double-buffered: generators write to `next_patterns[]`. At bar boundary (step == 0), `patterns[]` and `next_patterns[]` swap for active tracks with `pattern_ready` flag set. This ensures pattern changes are always musically aligned.

## Public API

**Lifecycle:** `genseq_init`, `genseq_reset`
**Config:** `genseq_set_bpm`, `genseq_set_time_signature`, `genseq_set_swing`, `genseq_set_scale`, `genseq_set_chord`
**Input:** `genseq_feed_note`, `genseq_feed_chord`
**Transport:** `genseq_start`, `genseq_stop`, `genseq_continue` (resume without reset), `genseq_set_position` (set step + bar for Song Position Pointer)
**Tick:** `genseq_tick` — returns event count
**Output:** `genseq_get_events`, `genseq_get_track_event`
**Params:** `genseq_drum_params`, `genseq_bass_params`, `genseq_melody_params`, `genseq_harmony_params` — return mutable pointers
**Pattern:** `genseq_regenerate`, `genseq_lock_pattern`, `genseq_unlock_pattern`, `genseq_set_seed`

## Test Files

| File | Tests |
|---|---|
| `test_euclidean.c` | Euclidean rhythm generation, known patterns (tresillo, cinquillo, 4-on-floor) |
| `test_scale.c` | Scale degree ↔ MIDI conversion, quantize-to-scale, chord-tone snap |
| `test_generators.c` | Per-track generator output correctness, parameter ranges |
| `test_integration.c` | Full tick loop, multi-track output, pattern regeneration, double-buffer swap, continue/position |

## Reference

See `docs/overview.md` sections 4 (C engine), 5 (WASM compilation), 8 (build commands) for full details.

---
> Source: [nanassound/conductr](https://github.com/nanassound/conductr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
