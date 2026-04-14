## piano-midi-viewer-builds

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Piano MIDI Viewer is a PyQt6-based desktop application that displays a visual piano keyboard responding to MIDI input in real-time. Made for music teachers, students, and content creators.

**Package architecture**: The application is split into the `piano_viewer/` Python package with focused modules.

**Current Version: 9.3.0**

For full version history, see [CHANGELOG.md](CHANGELOG.md).

## Project Structure

```
piano_viewer.py          # Thin launcher (delegates to piano_viewer package)
piano_viewer/            # Application package
  __init__.py            # VERSION, paths, logger, _SOUND_AVAILABLE, re-exports
  __main__.py            # Entry point (main function, font loading, startup)
  constants.py           # Sizing, colors, MIDI ranges, layout helpers, mutable globals
  i18n.py                # Translation system (tr(), load_translations(), LANGUAGES)
  helpers.py             # Config management, MIDI math, colors, fonts, button styles
  icons.py               # SVG/PNG loading from assets/, icon + cursor creation
  synth.py               # Wavetable synthesizer (PianoSynthesizer, _Voice)
  dialogs.py             # ErrorDialog
  settings.py            # SettingsDialog, UpdateChecker
  keyboard.py            # PianoKeyboard widget (rendering, mouse interaction)
  main_window.py         # PianoMIDIViewer (QMainWindow, MIDI, layout, app state)

assets/                  # SVG icons, cursors, and font (loaded at runtime by icons.py)
  icon.png               # App icon (1024x1024 PNG, used by CI for .ico/.icns generation)
  icon.svg               # App icon (SVG, used by website)
  pencil.svg             # Pencil icon (Phosphor Bold)
  pencil-cursor.svg      # Pencil cursor (white-filled Bold, 32x32)
  eraser.svg             # Eraser icon (Phosphor Bold)
  eraser-cursor.svg      # Eraser cursor (white-filled Bold, 32x32)
  camera.svg             # Camera/save icon (Phosphor Bold)
  settings.svg           # Cogwheel/gear icon (Phosphor Bold)
  plus.svg               # Plus icon for octave buttons (Phosphor Bold)
  minus.svg              # Minus icon for octave buttons (Phosphor Bold)
  refresh.svg            # Refresh icon for MIDI device list (Phosphor Bold)
  pedal.svg              # Sustain pedal icon (custom, stroke-based)
  JetBrainsMono-Regular.ttf  # Bundled font for note labels

packaging/               # PyInstaller build specs
  PianoMIDIViewer.spec         # Linux build spec
  PianoMIDIViewer-macos.spec   # macOS build spec

translations/            # UI translations (JSON, one file per language)
tests/                   # Test suite (pytest, 67 tests)
website/                 # Landing page (HTML/CSS/JS)
screenshots/             # README screenshots
.github/workflows/       # GitHub Actions CI (build.yml)
```

## Running the Application

```bash
# Using the Python virtual environment
source venv/bin/activate
python piano_viewer.py      # Thin launcher
# or
python -m piano_viewer      # Package entry point
```

## Dependencies

Core dependencies (installed in venv):
- PyQt6 - GUI framework
- python-rtmidi - MIDI input handling
- certifi - CA certificates for HTTPS in PyInstaller builds
- sounddevice (optional) - Built-in sound output (sine-wave synthesis)

The project uses a Python virtual environment in `venv/`.

## Key Constants and Constraints

**MIDI Range:**
- Full range: A0 to C8 (MIDI notes 21-108)
- Default display: C3 to B5 (MIDI notes 48-83, 3 octaves)
- Minimum span: 1 octave (12 notes), maximum: 7 octaves

**Sizing:**
- Initial key dimensions: 25px width x 150px height (ratio 6:1)
- Absolute minimums: 15px width x 30px height (always enforced)
- Ratio limits (always enforced): width 0.1-0.7x height, height 3-6x width
- Black key size: 80% of white key width, 60% of keyboard height

**Performance:**
- MIDI polling interval: 10ms (100Hz)
- Rendering: On-demand via Qt's update() mechanism

## Architecture

### Module Structure

The application is a Python package (`piano_viewer/`) with focused modules:

- **`__init__.py`** — Package root: VERSION, SETTINGS_VERSION, directory paths, logger, `_startup_errors`, `_SOUND_AVAILABLE`. Re-exports helper functions for backwards-compatible `from piano_viewer import X`.
- **`constants.py`** — All sizing, colors, MIDI ranges, layout helpers. Mutable globals: `UI_SCALE_FACTOR`, `LOADED_FONT_FAMILY` (set in `__main__.py`, accessed via `constants.X`).
- **`i18n.py`** — Translation system: `LANGUAGES`, `tr()`, `tr_for()`, `load_translations()`, `get_current_language()`.
- **`helpers.py`** — Pure logic: config management, MIDI math, color blending, font sizing, button styling. No widget creation.
- **`icons.py`** — Loads Phosphor Bold SVGs from `assets/` directory, renders to QPixmap/QIcon via `_render_svg_to_pixmap()`. Color customization via string replacement. Also creates pencil/eraser cursors from cursor SVGs and loads the PNG app icon.
- **`synth.py`** — Optional wavetable synthesizer (`PianoSynthesizer`, `_Voice`). Conditional on `_SOUND_AVAILABLE`.
- **`dialogs.py`** — `ErrorDialog` with copy-to-clipboard and optional settings reset.
- **`settings.py`** — `SettingsDialog` (QDialog) and `UpdateChecker` (QThread).
- **`keyboard.py`** — `PianoKeyboard` (QWidget): rendering, mouse interaction, note tracking.
- **`main_window.py`** — `PianoMIDIViewer` (QMainWindow): MIDI, layout, app state, octave management.
- **`__main__.py`** — Entry point: startup logging, font loading, UI scale, language, window creation.

### Cross-module patterns

- **Mutable globals**: `constants.UI_SCALE_FACTOR` and `constants.LOADED_FONT_FAMILY` are set once during startup. Other modules access them via `import piano_viewer.constants as constants; constants.X`.
- **Circular import avoidance**: `keyboard.py` uses `isinstance(parent, QMainWindow)` instead of importing `PianoMIDIViewer`. `i18n.py` uses lazy import of `get_config_path` inside `load_language_setting()`.
- **SVG assets**: Phosphor Bold icons loaded from `assets/` at runtime. Color customization via `#000000` string replacement. Custom pencil/eraser cursors from cursor SVGs.

### Component Details

1. **`PianoKeyboard`** (`keyboard.py`)
   - Note tracking: `active_notes` (dict: note -> velocity), `drawn_notes` (set), `active_notes_left/right` (sets for out-of-range)
   - Mouse interaction tracking (`mouse_held_note`, `glissando_mode`)
   - Hit detection: `_get_note_at_position()`, `_find_closest_note_to_position()`

2. **`PianoMIDIViewer`** (`main_window.py`)
   - Three-column layout: pencil/save/+/- (left) | piano (center) | settings/S/+/- (right)
   - State: `sustain_pedal_active`, `pencil_active`, `show_velocity`, display settings

3. **`SettingsDialog`** (`settings.py`)
   - Opened with `show()` (not `exec()`) so MIDI keeps working while open

### Key Architectural Concepts

**Sizing System**: Everything derives from white key width. Constants define initial size, window size = key count x key dimensions. Ratio limits and absolute minimums always enforced.

**MIDI Handling**: Polling-based (not callback). QTimer at 10ms. Handles Note On (0x90), Note Off (0x80), Control Change (0xB0 for CC 64 sustain). Out-of-range notes trigger +button glow. Auto-select: if no saved device and exactly one real (non-virtual) device available, connect automatically. Virtual ports (e.g. ALSA "Midi Through") are filtered via `_VIRTUAL_MIDI_PREFIXES` — only affects auto-select, never hides devices from Settings. Device scanning every 3 seconds handles hot-plug/unplug.

**Velocity**: `active_notes` is a dict (note -> velocity 1-127). `blend_colors()` interpolates between base and highlight color. Factor range 0.3-1.0 (soft notes always visible at 30%).

**Pencil Tool**: Independent drawing mode. Left click/drag draws to `drawn_notes` set, right click/drag erases. MIDI Note On toggles drawn_notes (including out-of-range notes, with +button glow), Note Off ignored. Exiting clears all marks.

**Sustain Indicator**: Pedal icon button is read-only — lights up when CC 64 >= 64, does not affect note highlighting. Icon color swaps to contrast with highlight background. When built-in sound is enabled, the sustain pedal keeps synth voices sounding until released.

**Built-in Sound**: Optional wavetable synthesizer using `sounddevice` (hidden if not installed). Off by default, toggled via Settings checkbox. Key design:
- `PianoSynthesizer` class with wavetable-based additive synthesis
- `_Voice` class with ASR envelope (instant attack at sustain level, 50ms release)
- 4 pitch-range wavetables (`_HARMONIC_PROFILES`): 1/n harmonic rolloff, lower notes get more harmonics for laptop speaker audibility
- Loudness compensation: `LOUDNESS_MULT_LOW` / `LOUDNESS_MULT_HIGH` constants interpolated across MIDI range
- Velocity-scaled sustain level: uses same `0.3 + 0.7 * (vel/127)` formula as key coloring (when `show_velocity` is on)
- Sustain pedal support: `set_sustain()` keeps voices alive while pedal held, releases on pedal lift
- Smooth mix gain: `1/sqrt(n)` attenuation interpolated across each buffer to avoid clicks when voice count changes
- Thread-safe: `threading.Lock` guards voice dict, audio callback runs in separate thread
- Max 12 simultaneous voices with oldest-voice stealing
- Sound triggers: MIDI note on/off, mouse press/release/drag, sustain pedal CC 64
- Settings persisted in `[audio]` section of settings.ini

**Settings**: Saved to `~/.config/piano-midi-viewer/settings.ini` via configparser. `SETTINGS_VERSION` constant + `migrate_settings()` for format changes. `[meta]` section tracks version.

**Logging**: Python `logging` module. Logger named `piano-midi-viewer`, levels: info (startup, connections), warning (fallbacks), error (failures).

**Icons**: Phosphor Bold SVG set loaded from `assets/` at runtime via `icons.py`. `_render_svg_to_pixmap()` strips existing dimensions, injects target size, and renders to QPixmap. Color customization via `#000000` string replacement — all SVGs must use full `#000000` hex (not shorthand `#000`). Pedal icon is custom (stroke-based, artist-designed). App icon loaded from `icon.png` (PNG, not SVG).

**Cursors**: Pencil and eraser tools use custom SVG cursors (`pencil-cursor.svg`, `eraser-cursor.svg`). These are Phosphor Bold icons with a white fill layer behind the black compound path, rendered at `CURSOR_SIZE = 24px`. Hotspots are at the pencil tip (1, 23) and eraser edge (4, 21). Default mode shows pencil cursor; holding RMB shows eraser cursor. Cursors are cached on `PianoMIDIViewer.__init__`.

**Computer Keyboard**: Optional feature (off by default) allowing the computer keyboard to act as a MIDI input. Home row A–K maps one octave (A=C, W=C#, S=D, ... K=C+1), Z/X shift octave down/up. Caps Lock toggles on/off globally. Also controllable via Settings checkbox. State: `computer_keyboard_enabled`, `computer_keyboard_octave` (default 4), `_computer_keys_held` (dict: Qt key → MIDI note). Key repeat filtered via `event.isAutoRepeat()`. Velocity fixed at 100. Persisted in `[input]` section of settings.ini.

**Keyboard Shortcuts**: Always-active shortcuts (work regardless of computer keyboard toggle): P = toggle pencil, Escape = exit pencil, `[` = add low octave, `{` = remove low octave, `]` = add high octave, `}` = remove high octave, O = toggle octave numbers, V = toggle velocity (shows toast). Computer keyboard mode adds: A–K = play notes, Z/X = shift octave, Caps Lock = toggle mode.

**Text Rendering**: JetBrains Mono (embedded, fallback to system monospace). Font size scales with key width. Minimum 8pt (hidden if smaller). Dynamic contrast: black text on light, white on dark.

**UI Scaling**: `UI_SCALE_FACTOR` global, `scaled(px)` helper. 50-200% range. Applied live via `rebuild_ui()` — tears down and recreates the layout around the existing `PianoKeyboard` widget, preserving all state (MIDI, notes, synth). Language changes also apply live the same way.

## Code Organization

Each module has a single responsibility (see Module Structure above). The `piano_viewer.py` file at the repo root is a thin launcher that delegates to `piano_viewer.__main__.main()`.

## Styling Conventions

- **Arch Blue** default highlight: `#5094d4` (QColor(80, 148, 212))
- **Button size**: Fixed 36px (BUTTON_SIZE constant)
- **Button icons**: All SVG (Phosphor Bold), displayed at 70% of button size via `setIconSize()`
- **Layout margins**: 5px (LAYOUT_MARGIN), don't scale with window
- **Key corner radius**: 8% of key width, 4px minimum
- **Canvas**: Grey background (120, 120, 120), 4px margin, 6px rounded corners
- **White keys**: Off-white (252) with shadow (170), borders normal (85) / highlighted (25)
- **Black keys**: Near-black (16) with black borders
- **Button styling**: `make_button_style(bg_color, text_color, interactive)` function

## CI / Build

- Workflow: `.github/workflows/build.yml`, manual trigger only (`workflow_dispatch`)
- Builds: Linux (ubuntu-22.04), Windows (windows-latest), macOS ARM (macos-14) + Intel (macos-15-intel)
- macOS: `--onedir` (NOT `--onefile`) for proper universal2 lipo. ARM + Intel combined with lipo per Mach-O file
- Linux: `--onedir` + AppImage (appimagetool with static runtime, no FUSE dependency)
- Windows: `--onefile` with icon generated via ImageMagick
- App icon: `assets/icon.png` -> .ico (Windows via ImageMagick) / .icns (macOS via sips + iconutil) / .png (Linux via ImageMagick)
- Linux audio bundling: libportaudio IS bundled (explicit in spec `extra_libs`); libasound, libjack, libstdc++ are excluded — must use host's PipeWire/PulseAudio stack and C++ runtime
- macOS DMG includes `README.txt` with xattr install instructions
- `create-release` job only runs on tag push

## Version Conventions

- **Patch** (x.x.1): Bug fixes, internal improvements, no new user-facing features
- **Minor** (x.1.0): New user-facing features
- **Major** (x.0.0): Breaking changes or major UX reworks

## Future Features & Ideas

Tracked in Claude Code memory files:
- **App todo** (`app-todo.md`): Pre-video polish, app improvements
- **Website todo** (`website-todo.md`): Content strategy, demos, guide, FAQ, contact
- **Website strategy** (`website-strategy.md`): Design philosophy, visual language, page structure

## Development Notes

- Settings saved to `~/.config/piano-midi-viewer/settings.ini`
- UI strings wrapped in `tr()` for i18n, translations in `translations/*.json` (en, de, es, fr, pl, pt, ru, uk)
- MIDI errors logged via `logging` module (replaced print() in 8.5.1)
- Cross-platform: Linux (AppImage), Windows (.exe), macOS (.dmg)
- Git hooks: post-commit auto-pushes to both remotes (Codeberg + GitHub)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skoomabwoy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/skoomabwoy)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
