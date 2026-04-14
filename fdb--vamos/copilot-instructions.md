## vamos

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Vamos is a polyphonic software synthesizer plugin (VST3/AU/Standalone) written in C++20 using JUCE 8.0.6. Its architecture is modeled after Ableton Live's Drift synthesizer. The project uses CMake as its build system and fetches JUCE and Catch2 via FetchContent (first configure takes ~2 minutes to download JUCE).

## Build & Run Commands

```bash
./scripts/dev.sh              # Debug build + launch Standalone app
./scripts/dev.sh --release    # Release build + launch
./scripts/dev.sh --test       # Build and run all tests (DSP + Plugin)
```

Manual test targets:
```bash
cmake --build build/Debug --target VamosTests -j$(sysctl -n hw.ncpu)
./build/Debug/tests/VamosTests              # DSP unit tests (no JUCE, fast)

cmake --build build/Debug --target VamosPluginTests -j$(sysctl -n hw.ncpu)
./build/Debug/tests/VamosPluginTests        # Plugin integration tests (links JUCE)
```

Run a single Catch2 test by name:
```bash
./build/Debug/tests/VamosTests "[Oscillator]"       # Run tests tagged [Oscillator]
./build/Debug/tests/VamosTests "Saw wave"            # Run test matching name substring
```

Build artifacts: `build/{Debug,Release}/Vamos_artefacts/{Standalone,VST3,AU}/`

## Architecture

### Signal Flow (per voice)
```
Osc1 + Osc2 + Noise → Mixer → Filter → Amp Envelope → Pan → Output
                                  ↑
              ModMatrix (Env1, Env2/CycEnv, LFO, Velocity, Modwheel, etc.)
```

### Two-Layer Separation

**DSP layer** (`src/dsp/`): Pure C++20, zero JUCE dependencies. All synthesis happens here. This is why VamosTests can build and run without linking JUCE.

**Plugin layer** (`src/PluginProcessor.cpp`, `src/PluginEditor.cpp`): JUCE integration — AudioProcessor, APVTS parameter system (30+ parameters), custom LookAndFeel GUI.

### Key DSP Classes (all in `vamos` namespace)

- **Synth** — 8-voice polyphonic engine with oldest-voice-stealing. Handles Poly/Mono/Stereo/Unison modes.
- **Voice** — Single voice container. Owns all DSP blocks and runs the per-sample signal chain.
- **Oscillator** — Phasor-based with PolyBLEP anti-aliasing. 7 waveforms (OscillatorType1) for Osc1, 5 (OscillatorType2) for Osc2.
- **Envelope** — Exponential ADSR. Attack overshoots to 1.2 so the curve naturally reaches 1.0.
- **Filter** — 8 filter types (Sallen-Key, SVF, Vowel, Comb, DJ, etc.) via FilterType enum.
- **ModMatrix** — Per-voice modulation routing. 3 general-purpose slots + dedicated filter/pitch/shape mod slots. ModSource (8 sources) → ModTarget (12 destinations).
- **LFO** / **CyclingEnvelope** — Modulators feeding into ModContext for per-sample modulation.

### Parameter Flow

`PluginProcessor::processBlock()` reads APVTS → packs into `SynthParams` struct → `Synth::setParameters()` → each `Voice::setParameters()`. Four critical parameters use `juce::SmoothedValue` to prevent audio clicks (volume, filterFreq, osc1Gain, osc2Gain).

### GUI

Click-to-reveal panel design: main view shows signal flow blocks, clicking a block reveals its parameter panel. Custom `VamosLookAndFeel` with synthwave-minimal dark theme (background #0a0a0a, accent #00E5FF).

## Testing

Two test executables with different dependency profiles:
- **VamosTests** — DSP-only unit tests. No JUCE. Fast. Tests Oscillator, Envelope, Filter, Voice, Synth modules.
- **VamosPluginTests** — Plugin integration. Links JUCE. Tests AudioProcessor creation, state save/load, MIDI dispatch.

Both use Catch2 v3.5.2 and are registered with CTest via `catch_discover_tests()`.

## Video Series

Educational video series in `video/` built with Remotion 4.0 (React-based video rendering). Each episode covers a development phase of the synth. Target audience: synth enthusiasts with some programming background but not deep DSP/audio coding experience.

### Video Commands

```bash
cd video && npm install              # Install dependencies
npx remotion studio                  # Open Remotion Studio for preview
npx remotion render Ep01-Foundation --output=out/ep01.mp4  # Render episode 1
```

### Video Structure

- `video/src/ep01/` — Episode 1 content (Video.tsx, timing.ts, narration.ts, code-snippets.ts, scenes/)
- `video/src/components/` — Reusable visual components (CodeBlock, WaveformVisualizer, ADSRVisualizer, SignalFlowDiagram, VoiceGrid, SpectrumVisualizer, etc.)
- `video/src/lib/` — Shared utilities: colors, fonts, shared timing constants, syntax highlighting
- `video/src/Root.tsx` — Registers all episode compositions
- `video/src/types.ts` — Shared types (VideoSchema, NarrationSegment, SceneNarration)
- `video/public/voiceover/` — ElevenLabs TTS MP3 files (gitignored, regenerated via `npx tsx generate-voiceover.ts`)

### Key Conventions

- **Audio-driven timing**: Narration audio lengths determine scene/section durations. Each visual section starts when its narration segment starts.
- **No CSS transitions**: All animations use `useCurrentFrame()` + `interpolate()` or `spring()`.
- **All Sequences get `premountFor={30}`** to prevent stuttering.
- **Dark synthwave aesthetic**: Background #0a0a0a, primary accent #00E5FF (cyan), matching the Vamos plugin UI.

See `video/WORKFLOW.md` for the full episode production workflow.

## Phase Documentation

Detailed design documents in `docs/` (phases 1–7) cover architectural decisions, Drift equivalence notes, and implementation rationale for each development phase.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fdb)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/fdb)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
