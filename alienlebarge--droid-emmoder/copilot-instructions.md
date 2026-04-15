## droid-emmoder

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Droid Emmoder (_pronounced `\ɑ̃.mɔ.de\` meaning "to initiate" in Swiss-French patois_) is a performance sequencer configuration for the [Droid Universal CV Processor](https://shop.dermannmitdermaschine.de/pages/droid-universal-cv-processor). The project contains a single configuration file that defines a 4-track drum sequencer with algorithmic pattern generation capabilities.

**Note**: This is a work-in-progress project inspired by Mathias Kettner's Droid examples.

## Architecture

- **Main Configuration**: `droid.ini` - The primary Droid configuration file containing all circuit definitions and documentation
- **Installation**: Configuration is deployed by copying `droid.ini` to an SD card and loading it into the Droid Master module

## Hardware Requirements

The configuration requires specific Droid hardware modules:
- 1 Master module
- 1 G8 module (gates)
- 1 P10 module (potentiometers)
- 4 P2B8 modules (buttons/LEDs)
- 1 M4 module

### I/O Configuration
- **Inputs**: I1 (Clock), I5 (Reset)
- **Outputs**: O1 (Clock), O2 (CV), O5 (Reset), O6 (Gate)
- **Gates**: G1.1 (Kick), G1.2 (Snare), G1.3 (HiHat), G1.4 (Clap)

## Configuration Structure

The `droid.ini` file is organized into functional sections:

### Core Components
- **Clock Management**: LFO-based internal clock with external clock input support
- **Reset Handling**: Global reset functionality
- **Page System**: Two-page interface system for different control modes

### Drum Sequencer (Page 1)
- **4 Drum Tracks**: Kick, Snare, HiHat, Clap
- **Algoquencer Circuits**: Each track uses the `algoquencer` circuit for pattern generation
- **16-Step Sequencing**: Step buttons distributed across two P2B8 controllers
- **Mute System**: Track selection + mute button combinations
- **Activity/Variation Controls**: Per-track activity and variation potentiometers

### Motoquencer (Page 2)
- **CV Sequencer**: 4-fader CV sequencer with 8 configurable steps
- **Dual Pages**: Pattern and parameter pages
- **Quantized Output**: CV output with 0.2V range and 2-semitone quantization

### Control Mapping
- **P1 (P10)**: Clock speed, track activity/variation controls
- **B2/B3 (P2B8)**: 16-step sequencer buttons (odd/even distribution)
- **B4 (P2B8)**: Track selection and mute functionality
- **B5 (P2B8)**: Page switching and motoquencer controls
- **P5 (M4)**: Motoquencer step count control

## File Editing Guidelines

### Configuration Syntax
- Droid configurations use INI-style syntax with circuit blocks
- Circuit definitions start with `[circuit_name]` headers
- Parameters are defined as `parameter = value` pairs
- Internal variables use `_VARIABLE_NAME` convention
- Hardware references use module notation (e.g., `P1.1`, `B2.1`, `G1.1`)
- **Section Separators**: Use `# -------------------------------------------------` format for section headers to ensure proper recognition by code editors (do NOT use `=` symbols as they break editor section folding)
- **Official Documentation**: Follow the Droid Eurorack module documentation at https://shop.dermannmitdermaschine.de/pages/downloads for proper .ini configuration file syntax and circuit specifications
- **Firmware Version**: This configuration targets Droid firmware blue-6 - refer to the blue-6 user manual for the most current circuit specifications
- **Local Resources**: The `doc/` directory contains comprehensive local documentation (not versioned in git):
  - `manual/droid-manual-blue-6.pdf`: Complete user manual
  - `manual/droid-cheatsheet-blue-6.pdf`: Quick reference guide
  - `manual/droid-patch-snippets-blue-6.txt`: Copy-paste examples for all circuits
  - `patches/`: Example configurations including motoquencer variations and sequencer templates
  - `graphics/circuits/`: Visual icons for all available circuits
  - `utilities/editors/`: Syntax highlighting and snippets for various code editors

### Common Patterns
- **Signal Routing**: Use `[copy]` circuits for signal distribution
- **Logic Operations**: Use `[logic]` circuits for AND/OR operations
- **Button Groups**: Use `[buttongroup]` for exclusive button selection
- **Mute Logic**: Combine track selection with mute button using `[logic]` AND gates

## Development Workflow

Since this is a hardware configuration project, there are no traditional build/test/lint commands. Development involves:

1. **Edit Configuration**: Modify `droid.ini` directly - all configuration and inline documentation is contained in this single file
2. **Update User Manual**: **CRITICAL** - When making changes to `droid.ini`, always update the corresponding user manual in `README.md` to reflect the changes. The README.md contains the French user manual that must be kept in sync with the actual configuration
3. **Reference Examples**: Use `doc/patches/` for inspiration and `doc/manual/droid-patch-snippets-blue-6.txt` for circuit syntax examples
4. **Hardware Testing**: Load configuration onto Droid hardware via SD card
5. **Version Control**: Use git for tracking changes to the configuration

### Documentation Language Requirements

**CRITICAL**: All documentation and comments in this project MUST be written in English:

- **README.md**: The user manual must be written entirely in English
- **droid.ini comments**: All inline comments and documentation within the configuration file must be in English
- **Code comments**: Any additional code comments or documentation must be in English
- **Commit messages**: Git commit messages should be written in English

This ensures consistency and accessibility for the international Droid community.

### Documentation Synchronization

**IMPORTANT**: The `README.md` file contains a comprehensive user manual that mirrors the functionality defined in `droid.ini`. Any changes to the configuration MUST be reflected in the user manual:

- **Hardware connections** (I/O mappings, controller assignments)
- **Control functions** (button mappings, potentiometer functions)
- **Workflow instructions** (page navigation, sequencer operation)
- **Feature descriptions** (mute system, clock management, etc.)

This ensures users always have accurate documentation that matches the actual hardware behavior.

### Useful Resources for Development
- **Circuit Reference**: Check `doc/manual/droid-patch-snippets-blue-6.txt` for exact syntax and parameters of any circuit
- **Example Patches**: Browse `doc/patches/` for similar implementations (especially `motoquencer_*.ini` files)
- **Visual Reference**: Use `doc/graphics/circuits/` images to understand circuit functionality
- **Editor Support**: Install syntax highlighting from `doc/utilities/editors/` for better development experience

## Key Variables and Signals

### Global Signals
- `_CLOCK`: Main clock signal (internal or external)
- `_RESET`: Global reset signal
- `_CLOCK_SPEED`: Internal clock rate control
- `_PAGE_1`, `_PAGE_2`: Page selection system

### Drum Sequencer
- `_ALGO_SELECT`: Current drum track selection (0-3)
- `_KICK_IS_MUTED`, `_SNARE_IS_MUTED`, etc.: Mute state per track

### Motoquencer
- `_MOTO1_PAGE`: Motoquencer page selection
- `_MOTO1_STEPS`: Active step count
- `_MOTO1_FADERMODE`: Fader operation mode

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alienlebarge)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/alienlebarge)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
