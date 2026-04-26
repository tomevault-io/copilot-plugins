## parquette-lighting

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

	# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repo layout

- `python/parquette-lights/` — main Python server (Poetry project, package `parquette.lights`). This is the main codebase and the backend server. It orchestrates a DMX lighting system in conjuection with a web UI based on open stage control
- `open-stage-control/` — Open Stage Control front-end config. `layout-config.json` is the UI layout; other files are server/UI initial state.
- `launchd/` — launchd plists + `install` script for auto-running the server and Open Stage Control on the mac mini.
- `esp/` — ESP-based hardware sketches (`controller_base`, `address_changer`, `i2c_scanner`). These will be used to create hardware interfaces sending OSC controls to the server. This can be ignored for now
- `python/parquette-lights/params.pickle` — persisted preset state. Auto-written when presets are saved/cleared in the UI; commit it to persist.

## Common commands

All run from `python/parquette-lights/`:

- `poetry sync` — install deps
- `poetry run server` — run the lighting server (entry: `parquette.lights.server:run`). Useful flags: `--local-ip 0.0.0.0`, `--entec-auto /dev/tty.usbserial-...`, `--art-net-ip`, `--boot-art-net`.
- `poetry run poe check` — runs black, pylint, mypy in sequence (continues on failure)
- `poetry run poe black` - runs black code formatter
- `poetry run poe mypy` - runs mypy and checks for typing issues 
- `poetry run poe pylint` - runs pylint and checks for issues
- `poetry run poe pytest` - runs the test suite
- `poetry run poe strip-osc-layout` — runs `scripts/strip_layout_defaults.py` to clean up the OSC layout JSON

## Architecture

The server (`parquette.lights.server`) wires together a small pipeline:

- **AudioCapture** (PortAudio via pyaudio) feeds raw audio into **FFTManager** (`audio_analysis/`), which produces frequency-band features.
- **FFTManager** analyses this raw audio with the help of the librosa framework. This creates various audio reactive signals to be used for orchestrating and in generators
- **Generators** (`generators/`) turn signals (either time series functions or audio reactive) into per-fixture color/intensity values, although the signals can be used also to control other parameters such as position. Types include `FFTGenerator`, `WaveGenerator`, `ImpulseGenerator`, `BPMGenerator`, plus a `Mixer` and `SignalPatchParam` for routing. `generator.py` defines the base class.
- **Fixtures** (`fixtures/`) model physical lights — `RGBLight`, `RGBWLight`, `YRXY200Spot`, `Spot`, `RadianceHazer`, etc. They convert generator outputs into DMX channel values.
- **DMXManager** (`dmx.py`) outputs to either an Enttec DMX USB Pro (via `dmxenttecpro`) or Art-Net (via `stupidartnet`).
- **OSCManager** (`osc.py`) handles bidirectional OSC with Open Stage Control. Parameters are wrapped as `OSCParam` so UI controls and server state stay in sync.
- **PresetManager** (`preset_manager.py`) serializes/loads parameter state to `params.pickle`.
- **ClientTracker** (`util/client_tracker.py`) tracks connected OSC clients so UI state can be re-pushed on reconnect.

The Open Stage Control front-end (in `open-stage-control/layout-config.json`) is the UI; it talks to the server over OSC. Web UI is served at port 8080 by Open Stage Control, server listens for OSC on 5005 and sends to OSC on 5006 by default.

## Conventions

- Python ≥ 3.10, formatted with black, linted with pylint (config: `pylintrc`), typed with mypy.
- When editing fixtures or generators, keep DMX channel layout and OSC parameter names consistent — both the layout JSON and `params.pickle` reference them by name.
- Changes to presets must be committed via `params.pickle` to persist across deploys.
- Avoid using _ before variable and function names (to denote private etc)
- When creating parameters that need to be controlled by the front end, create them as OSCParam and include them in the preset manage so they can be saved and sync'd with the front end
- When creating parameters assume you need a front end UI element (slider or similar) in the associated tab (most misc configration should go in fft dmx) and that UI element should also have a text area under it with a name and a real time value
- Use OSCParam.obj_param_setter with OSCParam where possible to avoid redundant code
- When making changes to python code we should always run `poetry run check` to format code and check for errors. We should also run `poetry run poe pytest` and check for any test errors
- Code written should use mypy typing hints
- If you don't see a bug don't make changes guessing at the solution, describe possible debugging approaches but don't implement anything unless told.
- Don't use _ named variables and function, assume all functions and variable are public

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/parquette-labs) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
