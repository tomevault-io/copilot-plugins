## switchboard-voice-changer-demo

> We are building a voice changer extension for Switchboard. https://docs.switchboard.audio/extensions/bring-your-own-extension/

We are building a voice changer extension for Switchboard. https://docs.switchboard.audio/extensions/bring-your-own-extension/

Feel free to download anything you need to do so. https://docs.switchboard.audio/downloads/

We'll be focusing on a single platform, Linux.

I don't want to use anything that uses a commerical license.

We should use TDD practices. Red tests->Code to make them green->iterate.

**IMPORTANT: Write tests FIRST, before any implementation code. No exceptions.**

**CRITICAL: Verify paths before giving commands to the user.**
- Before telling the user to run a command, use `ls` or `find` to verify the path exists
- Never assume directory structures; always check first
- If a build produces executables, verify their actual location before providing run instructions

**Baseline Tests Required for All Voice Filter Nodes:**
- Silence in produces silence out (input zeros → output zeros)
- Stereo audio handling (both channels processed correctly)
- setValue/getValue for runtime parameter changes

A simple test app that uses switchboard to collect mic input from the default, modify it, and play it back to the default audio output is adequate.

Work in C++17 or higher (C++20 is acceptable if needed for dependencies).

**Progress Tracking:**
- **IMPORTANT: Add items to `TODO_LIST.md` BEFORE starting any work. No exceptions.**
- Update item status as work progresses (In Progress → Completed)
- Keep completed work documented for reference

## Agent Cues

### Measuring Audio Node Gain Levels

When users report audio is too quiet or loud, use this technique to measure gain:

1. **Add helper functions to ProcessorTests.cpp** (if not already present):
   ```cpp
   float calculateRMS(const float* samples, uint numSamples);  // Root Mean Square
   float calculatePeak(const float* samples, uint numSamples); // Peak amplitude
   float linearToDb(float linear);  // Convert ratio to dB
   ```

2. **Create measurement test cases** tagged `[gaintest]`:
   - Generate known amplitude sine wave input (e.g., 440Hz, amplitude 0.8)
   - Process through the node
   - Calculate input/output RMS and peak
   - Report gain ratio and dB change via `INFO()` macro

3. **Run measurement tests**:

4. **Interpret results**:
   - RMS gain shows perceived loudness change
   - Peak gain shows headroom impact
   - -3dB RMS = half perceived loudness
   - Unity gain = ratio of 1.0 (0 dB)

5. **Common causes of gain reduction**:
   - Ring modulation: -3dB inherent (sine RMS = 1/√2)
   - Dry/wet mixing: reduces when wet signal is quieter
   - Filter resonance: can boost or cut depending on design

6. **Solutions**:
   - Add `outputGain` parameter to the node (preferred)
   - Chain a gain stage after the effect
   - Adjust mix parameter to compensate

### Building Voice Changer Demo

Guide for recreating the real-time voice changer demo with presets.

**Core Library: signalsmith-stretch**
- MIT-licensed polyphonic pitch/time stretching library: https://github.com/Signalsmith-Audio/signalsmith-stretch
- Header-only, designed for real-time from the ground up
- Capabilities:
  - Pitch shifting via frequency ratio or semitones
  - Formant preservation (prevents chipmunk effect)
  - Tonality limiting for timbre preservation at extreme shifts
  - Time stretching (0.75x to 1.5x typical)
- Key API:
  ```cpp
  #include "signalsmith-stretch.h"
  signalsmith::stretch::SignalsmithStretch<float> stretch;

  // Configuration
  stretch.presetDefault(channels, sampleRate);  // or presetCheaper() for lower latency

  // Set pitch (choose one method)
  stretch.setTransposeSemitones(semitones);     // e.g., -12 for octave down
  stretch.setTransposeFactor(factor);           // e.g., 2.0 for octave up

  // Formant shifting (frequency multiplier, NOT a 0-1 blend value!)
  stretch.setFormantFactor(formantFactor);

  // Process audio
  stretch.process(inputBuffers, inputSamples, outputBuffers, outputSamples);
  ```

**CRITICAL: setFormantFactor() Semantics**
- `setFormantFactor()` takes a **frequency multiplier**, NOT a 0-1 preservation blend
- `setFormantFactor(1.0)` = no formant shift (formants stay at original frequencies)
- `setFormantFactor(2.0)` = shift formants UP one octave
- `setFormantFactor(0.5)` = shift formants DOWN one octave
- Values near 0 cause near-silence or severe artifacts (collapse to DC)

**Implementing formantPreserve as a User-Friendly 0-1 Parameter:**
To expose a `formantPreserve` parameter (0=chipmunk effect, 1=natural voice):
```cpp
// Calculate formant compensation when formantPreserve=1.0:
// Pitch up = formants need to go down to compensate
float pitchFactor = std::pow(2.0f, pitchSemitones / 12.0f);
float fullCompensation = 1.0f / pitchFactor;

// Interpolate between no compensation and full compensation:
// formantPreserve=0 -> formantFactor=1.0 (chipmunk: formants shift with pitch)
// formantPreserve=1 -> formantFactor=fullCompensation (natural: formants preserved)
float formantFactor = 1.0f + formantPreserve * (fullCompensation - 1.0f);
stretch.setFormantFactor(formantFactor);
```

**For Chorus/Harmonizer Effects: Use SwitchboardAudioEffects**
- Download: https://switchboard-sdk-public.s3.amazonaws.com/builds/release/3.1.0/linux/SwitchboardAudioEffects.zip
- Chain the built-in `SwitchboardAudioEffects.Chorus` node after pitch shifting
- Provides rich, multi-voice effects without needing multi-pitch in the pitch shifter
- Configure via JSON preset (see Chaining Nodes section)

**Node Architecture Pattern**
All audio nodes inherit from `SingleBusAudioProcessorNode` with:
- Constructor accepting `SBAnyMap& config`
- `setBusFormat()`: called when audio format is established
- `process(AudioBus& inBus, AudioBus& outBus)`: per-buffer processing
- `setValue()/getValue()`: runtime parameter access
- Thread-safe parameters using `std::atomic<float>`

**signalsmith-stretch Buffering**
- The library handles internal buffering automatically
- Query latency with `stretch.inputLatency()` and `stretch.outputLatency()`
- For real-time, use `presetCheaper()` to minimize latency
- Process in small blocks matching your audio callback size
- Use `stretch.reset()` when switching presets to clear internal state

**Registering Nodes**
In `VoiceChangerNodeFactory.cpp`:
```cpp
#include "MyNode.hpp"
// In createNode():
if (type == "MyNode") { return new MyNode(config); }
```

**JSON Preset Format**
```json
{
    "type": "RealTimeGraphRenderer",
    "config": {
        "microphoneEnabled": true,
        "graph": {
            "nodes": [
                {
                    "id": "effectNode",
                    "type": "VoiceChanger.NodeTypeName",
                    "config": { "param": value }
                }
            ],
            "connections": [
                { "sourceNode": "inputNode", "destinationNode": "effectNode" },
                { "sourceNode": "effectNode", "destinationNode": "outputNode" }
            ]
        }
    }
}
```

**Chaining Nodes (e.g., PitchShift + Chorus or RingMod)**
Add multiple nodes to the `nodes` array, connect in series:
```json
"connections": [
    { "sourceNode": "inputNode", "destinationNode": "pitchShiftNode" },
    { "sourceNode": "pitchShiftNode", "destinationNode": "chorusNode" },
    { "sourceNode": "chorusNode", "destinationNode": "outputNode" }
]
```
Use `SwitchboardAudioEffects.Chorus` for harmonizer-like multi-voice effects.

**Standard Parameters**
- `pitchShift`: semitones (-24 to +24)
- `formantPreserve`: 0.0 to 1.0 (1.0 = full preservation, chipmunk prevention)
- `mix`: 0.0 to 1.0 (dry/wet)
- `outputGain`: 0.0 to 4.0

**RingMod for Alien/Robotic Effects**
- `carrierFrequency`: 10-20 Hz for wavering, 50-100 Hz for buzzy
- `mix`: 0.3-0.7 typical
- `threshold`: 0.02 (prevent modulating silence)

**Preset Design Philosophy**
- Vary `formantPreserve` across presets for distinct character
- Use negative semitones for deep/dark voices
- Chain RingMod after pitch shift for alien/monster effects
- Chain SwitchboardAudioEffects.Chorus for rich, multi-voice harmonizer effects

### Testing Voice Filter Nodes

Tests are located in `/tests/` and use Catch2 framework. Run with:
```bash
cd build && ctest --output-on-failure
# Or run specific test tags:
./VoiceChangerTests "[baseline]"         # Run all baseline tests
./VoiceChangerTests "[PitchShiftNode]"   # Run tests for specific node
```

**Test Infrastructure**

Common `TestAudioBus` helper struct (defined in each test file):
```cpp
struct TestAudioBus {
    std::vector<std::vector<float>> channelData;
    std::vector<float*> channelPointers;
    AudioBuffer<float>* buffer;
    AudioBus bus;

    TestAudioBus(uint sampleRate, uint numChannels, uint numFrames);
    float getSample(uint channel, uint frame) const;
    void setSample(uint channel, uint frame, float value);
};
```

Global Switchboard initialization (required once per test file):
```cpp
struct SwitchboardInitializer {
    SwitchboardInitializer() {
        Config sdkConfig({{"appID", "test"}, {"appSecret", "test"}});
        Switchboard::initialize(sdkConfig);
    }
    ~SwitchboardInitializer() { Switchboard::deinitialize(); }
};
static SwitchboardInitializer switchboardInit;
```

**Baseline Test Pattern (required for all nodes)**

Tag baseline tests with `[NodeName][baseline]`:
```cpp
TEST_CASE("NodeName - Silence in produces silence out", "[NodeName][baseline]") {
    // For STFT-based nodes, process multiple buffers (warmup period)
    for (int i = 0; i < 20; i++) {
        TestAudioBus inBus(44100, 2, 512);  // Zeros by default
        TestAudioBus outBus(44100, 2, 512);
        REQUIRE(node.process(inBus.bus, outBus.bus));

        if (i > 10) {  // Check after warmup
            for (uint ch = 0; ch < 2; ch++) {
                for (uint frame = 0; frame < 512; frame++) {
                    REQUIRE(std::abs(outBus.getSample(ch, frame)) < 0.01f);
                }
            }
        }
    }
}

TEST_CASE("NodeName - Stereo processing (both channels)", "[NodeName][baseline]") {
    bool ch0HasOutput = false, ch1HasOutput = false;
    for (int i = 0; i < 30; i++) {
        // Generate 440 Hz sine on both channels
        for (uint frame = 0; frame < 512; frame++) {
            float t = static_cast<float>(i * 512 + frame) / 44100.0f;
            float sample = 0.5f * std::sin(2.0f * M_PI * 440.0f * t);
            inBus.setSample(0, frame, sample);
            inBus.setSample(1, frame, sample);
        }
        node.process(inBus.bus, outBus.bus);
        if (i > 20) {  // Check after warmup
            // Verify both channels have output > 0.1f
        }
    }
    REQUIRE(ch0HasOutput);
    REQUIRE(ch1HasOutput);
}

TEST_CASE("NodeName - setValue/getValue parameters", "[NodeName][baseline]") {
    auto result = node.getValue("paramName");
    REQUIRE(!result.isError());
    REQUIRE(std::any_cast<float>(result.value()) == Approx(expectedValue));

    auto setResult = node.setValue("paramName", std::make_any<float>(newValue));
    REQUIRE(!setResult.isError());
}
```

**STFT-Based Node Warmup**
- STFT nodes (PitchShiftNode, FormantShiftNode, HarmonizerNode) require warmup buffers
- Process 10-20 buffers before asserting on output
- Use `i > 10` or `i > 20` guards in loops to skip warmup period

**Existing Test Files**
- `PitchShiftNodeTests.cpp` - Pitch shifting with formant preservation
- `RingModNodeTests.cpp` - Ring modulation with threshold gating
- `FormantShiftNodeTests.cpp` - Formant-only shifting with breathiness
- `HarmonizerNodeTests.cpp` - Multi-voice harmony generation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/switchboard-sdk)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/switchboard-sdk)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
