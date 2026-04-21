## obliterator

> This is a JUCE audio plugin that implements multi-effect processing with custom UI. Built using JUCE 8.0.6 with CMake.

# Distortion Plugin - JUCE Project

## Project Overview
This is a JUCE audio plugin that implements multi-effect processing with custom UI. Built using JUCE 8.0.6 with CMake.

## Current State
- **Plugin Type:** Effect plugin (not synth)
- **Formats:** AU, VST3, AAX, Standalone
- **Parameters:** 4 controls with custom knobs and progress arcs
- **UI:** 680x450 resizable window with custom SVG background and knobs
- **Algorithm:** Tanh distortion + asymmetric bias + sub-octave doubling + dry/wet mix

## Current Parameters

### 1. Drive (1.0 to 1000.0)
- **Default:** 1.0 (leftmost, no effect)
- **Algorithm:** `tanh(drive * (input + asymmetry_bias)) - dc_offset`
- **Arc:** Unipolar (fills left to right)

### 2. Asymmetry (-1.0 to 1.0)
- **Default:** 0.0 (center, no bias)
- **Algorithm:** Adds bias before distortion, removes DC offset after
- **Arc:** Bipolar (fills from center outward, left for negative, right for positive)

### 3. Sub Octave (0.0 to 1.0)
- **Default:** 0.0 (leftmost, no effect)
- **Algorithm:** Flip-flop octave divider with zero-crossing detection and lowpass smoothing
- **Arc:** Unipolar (fills left to right)

### 4. Dry/Wet (0.0 to 1.0)
- **Default:** 1.0 (rightmost, fully wet)
- **Algorithm:** Linear crossfade between dry and processed signal
- **Arc:** Unipolar (starts full, empties toward left)

## Custom UI Implementation

### SVG Background
- **File:** `resources/background.svg`
- **Implementation:** Loaded via `juce::Drawable::createFromImageData()`
- **Scaling:** `stretchToFit` for full window coverage

### Custom Knobs
- **File:** `resources/knob1.svg`
- **Implementation:** Custom `DistortionLookAndFeel` class overriding `drawRotarySlider()`
- **Features:** SVG rotation based on parameter value, smart bipolar/unipolar arc detection

### Progress Arcs
- **Unipolar:** Grey background arc, red fill from left (-135°) to current position
- **Bipolar:** Grey background arc, red fill from center (0°) outward in appropriate direction
- **Positioning:** Circles around knobs with larger radius than knob diameter

### Text Layout
- **Parameter Labels:** Positioned close to knobs with 14px font
- **Value Displays:** Real-time numeric values positioned below knobs
- **No Text Boxes:** Clean design without value input boxes

## Technical Implementation

### Core Algorithm Flow
```cpp
// 1. Get parameter values
currentDrive = *parameters.getRawParameterValue("drive");
currentAsymmetry = *parameters.getRawParameterValue("asymmetry");
currentSubOctave = *parameters.getRawParameterValue("suboctave");
currentDryWet = *parameters.getRawParameterValue("drywet");

// 2. Store dry signal
float drySample = inputSample;

// 3. Apply asymmetric distortion
if (currentDrive > 1.0f) {
    float biasedInput = inputSample + currentAsymmetry * 0.5f;
    processedSample = tanh(currentDrive * biasedInput) - dcOffset;
}

// 4. Add sub-octave (flip-flop octave divider)
if (currentSubOctave > 0.0f) {
    // Zero-crossing detection + flip-flop state + lowpass smoothing
    subOctaveSample = state.lowpassZ1 * processedSample * 0.5f;
}

// 5. Dry/wet mix
output = drySample * (1.0f - currentDryWet) + wetSample * currentDryWet;
```

### Custom LookAndFeel Features
- **Automatic Detection:** Bipolar vs unipolar parameters via `slider.getMinimum() < 0.0`
- **Arc Drawing:** Uses `juce::Path::addCentredArc()` with dynamic angle calculation
- **SVG Rotation:** `juce::AffineTransform::rotation()` around knob center
- **Fallback Safety:** Reverts to default JUCE slider if SVG fails to load

## File Structure
```
Source/
├── PluginProcessor.h/cpp     # Audio processing & 4 parameters
├── PluginEditor.h/cpp        # UI layout & value label updates
├── DistortionLookAndFeel.h/cpp # Custom knobs & arcs
resources/
├── background.svg            # Custom UI background
└── knob1.svg                # Custom knob graphics
CMakeLists.txt               # Build config + binary data
```

## Build System
- **CMake-based build** with `juce_add_binary_data()` for SVG assets
- **JUCE 8.0.6** fetched automatically from GitHub
- **Binary Resources:** SVG files embedded as `BinaryData::background_svg`, `BinaryData::knob1_svg`
- **Build Script:** `./build-run.sh` for standalone testing

## Development Notes
- **Tested in:** Bitwig Studio via standalone app
- **UI Updates:** Real-time value labels via slider `onValueChange` callbacks
- **Memory Safe:** No allocation in audio thread, cached SVG drawables
- **Responsive:** Custom layout scales with window resizing (fixed aspect ratio)

## Current Status
- ✅ **4 Parameters:** Drive, Asymmetry, Sub Octave, Dry/Wet all working
- ✅ **Custom Graphics:** SVG background and knobs fully implemented
- ✅ **Progress Arcs:** Unipolar and bipolar arc behaviors working
- ✅ **Tight UI Layout:** Labels and values positioned close to knobs
- ✅ **Real-time Feedback:** Live parameter value displays

## Possible Future Enhancements
- Additional knob SVG variations for different parameters
- Spectrum analyzer or oscilloscope visualization
- Preset system for parameter combinations
- MIDI learn functionality
- Additional distortion algorithms (tube modeling, etc.)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alex-fortunato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
