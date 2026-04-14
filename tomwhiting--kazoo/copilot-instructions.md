## kazoo

> Kazoo transforms vocal input (humming, mouth noises, beatboxing) into synthesized instrument sounds through genuine synthesis techniques. This is NOT effects processing over voice — it is real synthesis driven by vocal analysis.

# Kazoo - Voice-Driven Synthesizer

## Project Overview

Kazoo transforms vocal input (humming, mouth noises, beatboxing) into synthesized instrument sounds through genuine synthesis techniques. This is NOT effects processing over voice — it is real synthesis driven by vocal analysis.

## Architecture

Two-crate Rust workspace:
- **kazoo-core**: Audio library with zero UI dependencies. Any frontend consumes this API.
- **kazoo-tui**: Ratatui terminal frontend.

### Thread Architecture (kazoo-core)
1. **cpal input callback** (OS-managed) — writes mic samples to ring buffer. ZERO processing.
2. **cpal output callback** (OS-managed) — the main audio workhorse. Reads mic from ring buffer, drains commands, runs mixer (synth + effects), applies soft limiter, writes directly to output buffer, pushes display state. All processing state is owned by this callback's closure.
3. **Analysis thread** — pitch detection, FFT spectrum, formant extraction, onset detection. Allowed to allocate.
4. **Disk I/O thread** — writes recorded audio to WAV files.

### Communication
- UI → Engine: `crossbeam-channel` (bounded MPSC) for commands
- Engine → UI: `ringbuf` (lock-free SPSC) for display state snapshots
- Input callback → Output callback: `ringbuf` for mic samples

## Coding Standards

This codebase runs mission-critical infrastructure for financial, legal, and healthcare settings.

**REQUIREMENTS:**
- **NO LAZY CODE:** Every implementation must be complete and robust
- **NO SHORTCUTS:** Handle all edge cases, no partial implementations
- **NO DEVIATING FROM PLAN:** Follow agreed approach; raise concerns before changing direction
- **NO DEFERRED WORK:** Set work is NOT optional, you must not defer tasks "for later"
- **PRODUCTION READY:** All code deployable immediately
- **STABLE:** All error cases handled, inputs validated
- **PERFORMANT:** Consider memory, complexity, efficiency

**The standard:** Would you trust this code with patient records, financial transactions, or legal documents? If not, it's not ready.

## Critical Architectural Rules

1. **Output callback is the processing engine:** No allocations, no locks, no file I/O, no panics. Pre-allocated state is moved into the callback at engine start.
2. **Pre-allocate everything:** All buffers allocated at engine creation, reused throughout lifetime.
3. **`unsafe_code = "forbid"`** across the entire workspace. No exceptions.
4. **Lock-free communication only:** Ring buffers between real-time threads, channels for command dispatch.
5. **NaN/Inf defense:** All audio processing must handle NaN/Inf inputs gracefully (output silence).

## Module Dependency Graph

```
lib.rs (Processor trait, Error, Db, Pan, TimePosition, constants)
  │
  ├── transport/    (TransportClock — no other internal deps)
  ├── analysis/     (PitchDetector, SpectrumAnalyzer, OnsetDetector, FormantExtractor, EnvelopeFollower)
  ├── io/           (device enumeration, file I/O, disk recorder)
  ├── effects/      (BiquadFilter, Delay, Reverb, Chorus, Distortion, FormantShift, EffectChain)
  ├── synthesis/    (PitchTracked, Wavetable, Granular, Vocoder, PhaseVocoder — uses analysis types)
  ├── mixer/        (Track, Mixer — uses effects + synthesis)
  └── engine/       (EngineHandle, threading, ring buffers — uses ALL modules)
```

## Build Commands

```bash
# Check everything compiles
cargo check --workspace

# Run all tests
cargo test --workspace

# Run clippy (pedantic + nursery lints enabled)
cargo clippy --workspace

# Check formatting
cargo fmt --check

# Run the TUI
cargo run -p kazoo-tui

# Run the TUI in release mode (recommended for audio)
cargo run -p kazoo-tui --release
```

## Key Dependencies

| Crate | Version | Purpose |
|-------|---------|---------|
| cpal | 0.17 | Audio I/O (mic input, speaker output) |
| rustfft | 6.4 | FFT/IFFT for spectral processing |
| fundsp | 0.23 | DSP primitives and synthesis nodes |
| pyin | 1.2 | Vocal pitch detection (PYIN algorithm) |
| hound | 3.5 | WAV file read/write |
| symphonia | 0.5 | Multi-format audio file decoding |
| rubato | 1.0 | Sample rate conversion |
| ringbuf | 0.4 | Lock-free SPSC ring buffer |
| crossbeam-channel | 0.5 | Multi-producer channels |
| ratatui | 0.30 | TUI framework |
| crossterm | 0.29 | Terminal event handling |

## Code Review Standards

Reviews must use the Opus model. There is no such thing as a minor issue. Everything needs to be dealt with. Nothing can be skipped, nothing can be deferred, nothing can be of any standard other than the highest.

@.claude/plans/playful-rolling-prism.md
@.claude/plans/output-callback-refactor.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tomWhiting)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/tomWhiting)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
