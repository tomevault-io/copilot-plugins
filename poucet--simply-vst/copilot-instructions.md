## simply-vst

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Audio Plugin Architecture

This is a cross-format audio plugin that supports both CLAP and VST3 formats. The core plugin is built using the `clack-plugin` crate, with VST3 support added via `clap-wrapper`. The plugin is categorized as an instrument synthesizer with stereo output.

### Core Components

- **SimplyVST**: Main plugin struct implementing the `Plugin` trait
- **SimplyVSTAudioProcessor**: Handles real-time audio processing (currently unimplemented with `todo!()`)
- **SimplyVSTShared**: Shared state accessible across different plugin components
- **SimplyVSTMainThread**: Main thread interface for UI and parameter handling

The plugin uses a three-tier architecture:
1. Shared state for data accessible by all components
2. Main thread handling for UI and host communication
3. Audio processor for real-time audio processing

### Key Implementation Notes

- The plugin is exported as a dynamic library (`cdylib`) for VST host loading
- Audio processing and activation methods are currently placeholders (`todo!()`)
- Plugin features are declared as INSTRUMENT, SYNTHESIZER, and STEREO
- Uses the clack-plugin framework's factory pattern for plugin instantiation

## Development Commands

```bash
# Build the plugin (creates dynamic library)
cargo build

# Build and bundle both CLAP and VST3 for release (xtask pattern)
cargo xtask build

# Build with debug profile
cargo xtask build debug

# Build only CLAP bundle
cargo xtask build release clap

# Build only VST3 bundle  
cargo xtask build release vst3

# Check code without building
cargo check

# Run tests (when implemented)
cargo test
```

### Build Output

- **Debug builds**: Creates `target/debug/libvst.dylib`
- **Release builds**: Creates `target/release/libvst.dylib`
- **Bundle creation**: Creates plugin bundles at `target/bundle/`
  - `SimplyVST.clap/` - CLAP plugin bundle
  - `SimplyVST.vst3/` - VST3 plugin bundle

Both bundles have the proper macOS structure:
```
SimplyVST.clap/          SimplyVST.vst3/
├── Contents/            ├── Contents/
│   ├── Info.plist       │   ├── Info.plist
│   └── MacOS/           │   └── MacOS/
│       └── libvst.dylib │       └── libvst.dylib
```

### Build System

The project uses a Cargo workspace with two members:
- **vst**: Main plugin library with CLAP and VST3 exports
- **xtask**: Build automation tool following the xtask pattern that builds the plugin and creates both CLAP and VST3 bundles

The xtask tool automatically builds the plugin first, then copies the compiled dylib and appropriate Info.plist files into the proper bundle structures:
- CLAP bundle uses identifier `com.simply-music.ai.simplyvst`
- VST3 bundle uses identifier `com.simply-music.ai.simplyvst.vst3`

Both formats are supported through the same codebase, with VST3 compatibility provided by the `clap-wrapper` crate.

---
> Source: [poucet/simply-vst](https://github.com/poucet/simply-vst) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
