## klinker

> - **Type:** Audio Plugin (VST3, Standalone)

# Klinker AI Context & Development Notes

## Project Overview
- **Name:** Klinker
- **Type:** Audio Plugin (VST3, Standalone)
- **Framework:** JUCE 8 (via CMake FetchContent)
- **Language:** C++ (C++20 standard)
- **License:** AGPLv3
- **Target OS:** Linux (Fedora KDE / Wayland), cross-platform compatible code.

## Code Style & Conventions
- **Language:** Code, Comments, and Class Names must be STRICTLY English.
- **Naming:** CamelCase for classes, camelCase for variables.
- **Modern C++:** Use `auto`, `nullptr`, smart pointers where possible.
- **Dependencies:** Conservative. Use JUCE modules unless external libs are strictly better/necessary.

## Build System (CMake)
- **Do NOT use Projucer.** We use a native CMake workflow.
- **Build Command:** `cmake -B build && cmake --build build --target Klinker_Standalone`
- **Linux Specifics (Handled in CMakeLists.txt):**
  - Requires explicit linking of `gtk+-x11-3.0`, `webkit2gtk-4.1`, and `libcurl`.
  - Headers must be found via PkgConfig (already configured).
  - `JuceHeader.h` does NOT exist. Include specific modules (e.g., `#include <juce_audio_utils/juce_audio_utils.h>`).

## Current Status
- Project compiles and runs (Standalone).
- Bundle ID: `nl.spacebabies.Klinker`.
- Basic DSP Skeleton (Circular Buffer) is drafted in `TODO.md` but needs verification/implementation.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/spacebabies) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
