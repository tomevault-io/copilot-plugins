## expressive

> A VST3/AU audio effect plugin built with JUCE that dynamically pans a stereo input signal based on the velocity of incoming MIDI notes. Low velocity pans left, high velocity pans right.

# Expressive — Velocity-Based Dynamic Panner

## What This Is
A VST3/AU audio effect plugin built with JUCE that dynamically pans a stereo input signal based on the velocity of incoming MIDI notes. Low velocity pans left, high velocity pans right.

## Architecture

### Core Files
- **Source/PluginProcessor.h/.cpp** — Audio processing and MIDI parsing. Reads MIDI note-on velocity, maps it to a pan position, applies equal-power stereo panning to the audio buffer.
- **Source/PluginEditor.h/.cpp** — GUI with a visual pan indicator and parameter controls.

### DSP Design
- Pan value is derived from MIDI velocity: 0 = hard left, 64 = center, 127 = hard right
- Use `juce::SmoothedValue<float>` to interpolate pan changes over ~10ms to avoid clicks/zipper noise
- Equal-power panning law: `leftGain = cos(pan * pi/2)`, `rightGain = sin(pan * pi/2)`
- When no MIDI note has been received yet, default to center pan

### Parameters (via AudioProcessorValueTreeState)
| ID              | Name            | Range       | Default | Description                              |
|-----------------|-----------------|-------------|---------|------------------------------------------|
| `smoothing`     | Smoothing Time  | 1–100 ms    | 10 ms   | How quickly pan interpolates to new value |
| `panWidth`      | Pan Width       | 0–100%      | 100%    | Scales the pan range (0% = always center) |
| `panCurve`      | Pan Curve       | 0 (lin)–1 (log) | 0   | Linear vs logarithmic velocity mapping   |
| `bypass`        | Bypass          | on/off      | off     | Pass audio through unchanged             |

### Signal Flow
```
Audio In (stereo) ──┐
                     ├──► Pan Processing ──► Audio Out (stereo)
MIDI In (velocity) ──┘
```

## Build Instructions
```bash
# First time: add JUCE submodule
git submodule add https://github.com/juce-framework/JUCE.git libs/JUCE
git submodule update --init

# Build
cmake -B build -DCMAKE_BUILD_TYPE=Release
cmake --build build
```

## Testing
Build the `Standalone` target and run it — it hosts itself with an audio device selector. Send it MIDI from a virtual keyboard (e.g., VMPK or your DAW's piano roll) while playing audio through it.

## Coding Conventions
- JUCE style: `camelCase` for methods/variables, `PascalCase` for classes
- Never allocate memory on the audio thread (no `new`, no `std::vector::push_back`, no `String` operations inside `processBlock`)
- All user-facing parameters go through `AudioProcessorValueTreeState`
- Keep GUI updates decoupled from audio thread via `AsyncUpdater` or timer-based polling
- Prefer `juce::dsp` module utilities where available

## Plugin Metadata
- **Manufacturer Code:** `3th7`
- **Plugin Code:** `Expr`
- **Formats:** VST3, AU, Standalone
- **Needs MIDI Input:** Yes
- **Is Synth:** No (it's an effect)

---
> Source: [3than777/Expressive](https://github.com/3than777/Expressive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
