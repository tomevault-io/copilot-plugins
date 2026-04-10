## grooveseq

> This file briefs autonomous agents working inside `drum-machine`. Follow the guardrails here before touching code.

# AGENTS

This file briefs autonomous agents working inside `drum-machine`. Follow the guardrails here before touching code.

## Mindset
- Treat GrooveSeq as an audio plug-in where UI, DSP, and file I/O must stay responsive; avoid blocking the audio thread.
- Prefer incremental, well-documented changes; mirror JUCE idioms already in the repository.
- When unsure, inspect existing classes in `Source/` and mimic their flow instead of inventing new patterns.

## Repository Tour
- `Source/PluginProcessor.*` – audio engine, sample loading, sequencing, parameter/state handling.
- `Source/PluginEditor.*` – JUCE UI glue, slider wiring, pad selection, file browser integration.
- `Source/SamplePad.*` – reusable pad component with drag-and-drop and selection visuals.
- `Source/Sequencer.*` – grid data model and pattern generation.
- `Source/SequencerGrid.*` – grid renderer and interaction logic.
- `Source/PluginEntry.cpp` – JUCE entry point.
- `scripts/build_vst3.sh` – convenience build+install script.
- `build/` – generated artifacts; never hand-edit.

## External Rules
- No `.cursor/rules/`, `.cursorrules`, or `.github/copilot-instructions.md` files exist, so this document is the authoritative agent policy.

## Dependencies
- CMake ≥3.15 is required.
- JUCE must be available; default expectation is a `JUCE/` sibling containing JUCE’s CMake project.
- macOS builds rely on the user’s VST3 install path `~/Library/Audio/Plug-Ins/VST3`.
- The project uses C++17 plus JUCE modules linked through `juce_add_plugin`.

## Environment Variables
- `BUILD_TYPE` controls Release vs Debug in the helper script (defaults to `Release`).
- `JUCE_DIR` can point to a custom JUCE checkout; default is `<repo>/JUCE`.
- `BUILD_DIR` overrides the CMake build tree (default `<repo>/build`).

## Build Commands
1. Configure only:
   ```bash
   cmake -S . -B build -DJUCE_DIR=$HOME/dev/JUCE -DCMAKE_BUILD_TYPE=Release
   ```
2. Build after configuring:
   ```bash
   cmake --build build --config Release
   ```
3. End-to-end build + install via helper:
   ```bash
   BUILD_TYPE=Release JUCE_DIR=$HOME/dev/JUCE ./scripts/build_vst3.sh
   ```
- Script copies `GrooveSeq.vst3` into the user VST3 directory; keep backups if you are testing experimental code.

## Fast Iteration Tips
- For UI-only tweaks use `cmake --build build --config Debug -j8` to speed iteration.
- Delete `build/` only if the user okays losing cached results.
- If JUCE is unavailable locally, add it as a git submodule or point `JUCE_DIR` to an existing checkout; do not vendor custom patches without alignment.

## Testing Guidance
- There are currently **no automated unit tests** or CTest targets.
- Manual QA involves loading the VST3 in a DAW (e.g., Ableton, Logic via adapter) and ensuring sequencing + drag/drop features behave.
- If you add tests later, integrate them with CTest so single tests can run via `ctest -R <regex> --output-on-failure` inside the build directory.
- Please document any new test binaries or sample assets in this file when you add them.

## Linting / Formatting
- No automated lint step exists; rely on clang-format-style spacing already used in `Source/*.cpp`.
- Maintain 4-space indentation, braces on the next line for classes and on the same line for functions as currently shown.
- Keep includes ordered: self-header first (if any), then JUCE modules, then STL headers, then project headers.
- Use `#pragma once` at the top of headers.

## Type & Memory Practices
- Prefer `std::array` for fixed-size pad/step data; see `Sequencer` and `padSounds` usage.
- Use `std::unique_ptr` + `std::make_unique` for UI components allocated per pad.
- Keep `std::atomic` for cross-thread state such as `currentStep`.
- Use JUCE smart pointers (`juce::AudioProcessorValueTreeState`, `juce::GridItem`, etc.) where already established instead of manual new/delete.

## Naming Conventions
- Classes: `PascalCase` (`GrooveSeqAudioProcessor`).
- Member variables: camelCase with descriptive nouns (`sequencerGrid`, `padNames`).
- Constants: `kPrefix` inside classes (e.g., `Sequencer::kPads`).
- Functions/methods: camelCase verbs (`generatePattern`, `setPadName`).
- Parameters pass-by-const-ref for heavy JUCE objects; plain values for PODs.

## Header Guidelines
- Header files should declare interfaces and minimal inline code.
- Forward declare JUCE types only when it saves compile time and you understand the dependency graph; otherwise include `<juce_audio_utils/juce_audio_utils.h>` as done now.
- Keep member ordering: public interface, protected (if any), private data; stash helper type aliases near use.

## Implementation Details
- Group related lambdas near their usage (see slider callbacks) and capture `[this]` explicitly.
- For sequences of UI setup calls, consider helper lambdas (see `setupSlider`) to avoid duplication.
- Always repaint components when visual state changes (`SamplePad::setSelected`).
- Use `juce::Grid` or bounds arithmetic for layout; avoid absolute coordinates littered across the file.

## Error Handling
- Early-return on invalid arguments (pad/step bounds) exactly as `loadSample` and `setStepState` do.
- Prefer `std::optional`-style checks using JUCE objects (e.g., `juce::File::existsAsFile()`) before heavy operations.
- When failing, return `false` or no-op silently unless the caller surfaces the error; this matches current UX (no modal dialogs).
- Guard critical sections with `juce::SpinLock::ScopedLockType` to avoid race conditions; do not hold locks across disk I/O.

## Threading and Real-Time Safety
- `processBlock` must stay lock-free except for copies guarded briefly by `SpinLock` (already done for pattern snapshots); do not add allocations or logging inside it.
- Use `juce::ScopedNoDenormals` and buffer clearing patterns shown in the existing code.
- All sample loading, UI actions, and file operations must occur on the message thread or worker threads, never inside `processBlock`.

## Sequencer Logic
- `Sequencer::generate` seeds a deterministic pattern given density/fills/seed; keep randomness centralized there.
- UI reads/writes state through `SequencerGrid::DataProvider`; always go through `GrooveSeqAudioProcessor` methods to maintain locking.
- When adding features (swing per pad, etc.), snapshot pattern data before iterating to avoid locking the audio thread for long stretches.

## Sample Pads and File Handling
- `SamplePad` components expect `.wav/.wave/.aiff/.aif/.flac`; extend `WildcardFileFilter` plus `SamplePad::isInterestedInFileDrag` in tandem.
- Keep user feedback minimal but informative (labels, highlights) and avoid blocking file dialogs.
- When selecting pads, update ADSR sliders without triggering callbacks by passing `juce::dontSendNotification`.

## Parameter & State Management
- All tweakable values live in `AudioProcessorValueTreeState` parameters registered in `createParameterLayout()`.
- Use `SliderAttachment` for parameters that need two-way binding; new sliders must keep their attachments alive as members to avoid dangling references.
- Persist state strictly via `getStateInformation`/`setStateInformation`; if you add arrays or structs, serialize them into the `ValueTree` rather than ad-hoc XML nodes.

## UI Patterns
- Keep `resized()` deterministic: compute rectangles, then assign bounds; avoid using magic numbers that break when resizing.
- If adding controls, update both layout math and color/Justification settings to match the dark theme aesthetic (midnight backgrounds, muted greys, bright accents).
- Respect JUCE’s repaint model – call `repaint()` when data changes due to user input or timer events.

## Adding New Files
- Place new components in `Source/` and add them to `target_sources` in `CMakeLists.txt` so they compile into the plugin.
- Maintain alphabetical or logical grouping within `target_sources` for readability.
- Remember to update this document when adding significant subsystems.

## Documentation Expectations
- Update AGENTS.md whenever build commands, scripts, or style guides change; keep the tone direct and actionable.
- Record new manual test steps so future agents can reproduce validation quickly.

## Pre-Submission Checklist
- Build succeeds in both Debug and Release (or provide rationale if a configuration is temporarily broken).
- Manual VST3 smoke test completed after any change touching DSP, sample loading, or UI event handling.
- Run `cmake --build build --target GrooveSeq_VST3` (or equivalent) before publishing.
- Verify no JUCE warnings/regressions appear in Xcode/Clion logs; treat warnings as blockers.

## Version Control & Review
- Keep feature branches focused; do not mix refactors with functional changes unless necessary for safety.
- Never rewrite history or force-push shared branches without explicit approval.
- Reference DAW/manual test notes in commit messages when behavior changes.
- Review diffs for generated files (`build/`) and exclude them from commits.

## Manual QA Flow
- Launch a supported DAW, scan plugins, and load GrooveSeq.vst3.
- Confirm pads trigger audio, the sequencer advances, and the grid highlights match audio playback.
- Drag-and-drop at least one `.wav` sample plus use the Browse button to ensure both paths work.
- Wiggle swing/density/fills parameters to confirm UI -> processor wiring.
- Save and reopen the session to verify state restoration before sign-off.

## Next Steps for Contributors
- Consider adding automated tests (e.g., Catch2) around `Sequencer` to enable real single-test runs.
- Explore parameter automation smoothing to avoid zipper noise when tweaking knobs live.
- Evaluate packaging for Windows/Linux if cross-platform support becomes a goal; document the toolchains when you add them.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodrwan)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/rodrwan)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
