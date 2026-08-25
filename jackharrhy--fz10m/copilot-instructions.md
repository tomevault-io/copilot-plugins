## fz10m

> Fz10m is a standalone iPlug2 instrument repository. It contains one plugin and its own Xcode workspace, Visual Studio solution, packaging scripts, and pinned iPlug2 submodule. It is not an iPlug2 template workspace.

# AGENTS.md

## Project

Fz10m is a standalone iPlug2 instrument repository. It contains one plugin and its own Xcode workspace, Visual Studio solution, packaging scripts, and pinned iPlug2 submodule. It is not an iPlug2 template workspace.

## Layout

- `Fz10m.cpp`: parameter definitions, UI construction, and UI/DSP messages
- `Fz10m.h`: stable parameter IDs, control/message tags, and plugin declaration
- `Fz10m_DSP.h`: wavetable oscillator, per-voice processing, filters, envelopes, and lo-fi stage
- `Fz10m_State.cpp`: custom state serialization and migration from older releases
- `config.h`: plugin metadata, formats, channel layout, and UI dimensions
- `projects/`: macOS/iOS Xcode projects and the Windows Visual Studio solution
- `resources/`: plugin metadata and interface assets
- `config/`: platform build configuration
- `scripts/`: build and packaging helpers
- `installer/`: installer assets and release changelog
- `docs/gameplan.md`: current design constraints, technical debt, and roadmap
- `iPlug2/`: pinned framework submodule; do not treat it as project-owned source

## Setup and building

After cloning, run `just setup`. It initializes submodules, downloads the pinned VST3 and CLAP SDKs, builds the VST3 validator, and installs the local CLAP validator. `pluginval` remains a Homebrew cask.

macOS VST3 Debug:

```bash
just build
```

Other macOS schemes include `macOS-APP`, `macOS-AUv2`, `macOS-CLAP`, `macOS-AAX`, and `macOS-AUv3`. Pass the scheme to `just build`, for example `just build macOS-AUv2`.

Use `just test` for the macOS VST3, AUv2, and CLAP build and validator suite. Use `just release-dry` to exercise macOS packaging without tagging or pushing.

`just release` reads the next version from `installer/changelog.txt`, updates version files, commits, tags, and pushes. Run it only when a release is explicitly requested and the intended changes are already committed on a clean `main` branch.

Windows builds use Visual Studio 2022 and `projects/Fz10m.sln`.

VST2 is intentionally unsupported because its SDK is proprietary and deprecated. Ignore the legacy VST2 project files inherited from iPlug2.

## Architecture

- `ProcessBlock` and anything it calls must be realtime safe. Do not allocate, lock, log, or perform file I/O on the audio thread.
- Parameters use non-normalized values and compile-time enum indexes. Access them with `GetParam(kIndex)->Value()`.
- Each voice currently produces one mono sample and adds it to both output channels.
- The rendered 128-sample wavetable is shared by the UI/transport and voice DSP. It is not double-buffered yet, so do not add higher-rate writes without implementing a safe handoff.
- Keep UI synchronization and table preparation off the audio thread.

## State compatibility

Host sessions and presets depend on both parameter IDs and the custom byte layout in `Fz10m_State.cpp`.

- Parameter IDs in `EParams` are public, persisted IDs. Append new parameters immediately before `kNumParams`. Never insert, reorder, remove, or reuse an existing ID.
- `SerializeParams()` writes one raw `double` per parameter without a count or ID. Appending a parameter therefore changes the state payload and requires a state-version migration.
- Changing `kNumParams`, `kWavetableSize`, `kNumHarmonics`, field order, or serialized field types also requires a state-version migration.
- State version 2 writes, in order: the version `int`, 128 rendered wavetable samples as bipolar `double` values, 16 harmonic amplitudes as `double` values, then all 20 parameter values.
- Existing migration code reads unversioned 7- and 11-parameter states, plus version 1 states containing 11, 17, 19, or 20 parameters. Preserve these readers unless compatibility is deliberately dropped.
- Some VST3 state chunks have a trailing 4-byte field. Legacy size matching accepts that trailer and new migration code must not accidentally reject it.
- The serialized rendered wavetable is authoritative on recall. Do not regenerate it from the Wave preset parameter after loading, or custom Draw/Additive state will be lost.
- Harmonics and the rendered wavetable are custom state, not parameters. New persistent non-parameter data belongs in the versioned custom payload.

When the payload changes:

1. Increment `kStateVersion`.
2. Keep `SerializeState()` writing only the newest layout.
3. Add an explicit `UnserializeState()` branch for the previous layout and read its exact field and parameter counts.
4. Initialize fields that did not exist in the old version to current defaults.
5. Restore the rendered table, harmonics, parameters, DSP state, and open UI without allowing parameter callbacks to overwrite custom data.
6. Test a newly saved session and at least one session from the previous release. In Ableton VST3, verify save/reopen, GUI close/reopen, Draw mode, Additive mode, generated presets, Random, and parameter automation.

There are no automated legacy-state fixtures yet. Treat state work as incomplete until host-level recall has been checked.

## Code signing

Local macOS VST3, AUv2, and CLAP builds use ad hoc signing (`CODE_SIGN_IDENTITY = "-"` in `projects/Fz10m-macOS.xcodeproj/project.pbxproj`) so bundle resources are sealed. Without it, Ableton Live 12 can reject the bundle during scanning with "code has no resources but signature indicates they must be present". Distribution and AUv3 delivery require a real identity.

## Code style

- Use 2-space indentation, no tabs, and Unix line endings. Windows project files are exempt.
- Use `mCamelCase` for members, `pCamelCase` for pointer arguments, and `_methodName` for internal methods.
- Use C++17 features such as `override`, `final`, `auto`, `std::optional`, and `std::string_view`.
- Prefer WDL (`iPlug2/WDL`) over STL in hot paths.
- Refer to the [iPlug2 wiki](https://github.com/iPlug2/iPlug2/wiki) and [API docs](https://iplug2.github.io/docs) for framework behavior.

---
> Source: [jackharrhy/Fz10m](https://github.com/jackharrhy/Fz10m) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
