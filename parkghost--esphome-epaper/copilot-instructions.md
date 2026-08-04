## esphome-epaper

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

An **ESPHome external-components** repository (published as `github://parkghost/esphome-epaper`), not a standalone app. It is consumed from an ESPHome device YAML via `external_components:`. Each subdirectory of `components/<name>/` is an independent ESPHome platform.

There are **two independent e-paper component families** that support overlapping panels via different architectures:

- `components/waveshare_epaper/` — the classic ESPHome `waveshare_epaper` driver style. Actively used; the panels in README's table live here.
- `components/epaper_spi/` — a port of ESPHome-dev's newer `epaper_spi` architecture, adding UC8179 + SSD16xx drivers not present upstream.

Pick the family a task refers to; they do not share code.

## Build / validate / flash

There is no build system in the repo. You exercise the components by pointing a throwaway ESPHome config at them and driving the `esphome` CLI. Minimal test config:

```yaml
esphome: { name: test }
esp32: { board: esp32-s3-devkitc-1, framework: { type: esp-idf } }
external_components:
  - source: { type: local, path: /abs/path/to/components }
    components: [epaper_spi]   # or [waveshare_epaper]
spi: { clk_pin: GPIO12, mosi_pin: GPIO11 }
display:
  - platform: epaper_spi
    model: gdey075t7
    cs_pin: GPIO10
    dc_pin: GPIO14
    reset_pin: GPIO13
    busy_pin: GPIO9
    update_interval: never
    lambda: 'it.fill(Color(255,255,255));'
```

- Validate config + schema:  `esphome config test.yaml`
- Compile firmware:          `esphome compile test.yaml`
- OTA flash a live device:   `esphome run <device>.yaml --device <host>.local --no-logs`
- Stream logs:               `esphome logs <device>.yaml --device <ip>`
- **`esphome clean test.yaml` after removing/renaming any component `.cpp`** — PlatformIO caches the source list and errors on a stale `*.cpp.o` target (`Source ... not found`). A clean rebuild fixes it.

### esphome version gotcha

`epaper_spi` needs a recent esphome (dev / ~2026.6+) providing `mipi`, `split_buffer`, `display.add_metadata`. The **CLI** (pipx venv, e.g. 2026.6.4) has these. A bare `python3 -c "import esphome..."` may resolve an **older** `~/.local` install (2025.6.1) that lacks them — do feature/import checks with the CLI's interpreter (`head -1 $(which esphome)`), not bare `python3`. Upstream `epaper_spi` source for reference/patterns: `~/workspaces/github_clone/esphome/esphome/components/epaper_spi/`.

## Architecture: waveshare_epaper (classic)

- One C++ class per panel, extending `WaveshareEPaper` (mono, `get_buffer_length_ = W*H/8`) or `WaveshareEPaperBWR` (`W*H/4`, two planes) in `waveshare_epaper.{h,cpp}`.
- Blocking model: `display()` does the whole update in one call (reset → init → write RAM → 0x12 refresh → `wait_until_idle_()` → deep sleep).
- Registered in `display.py`'s `MODELS` dict: `"model-name": ("<type>", CppClass)`. Type codes gate `full_update_every` (type `b` warns it's unsupported; `a`/`c` support it).
- Buffer bit convention: **bit 1 = white, 0 = black**; `fill(on)` = black. Panels wanting 1=black receive `~buffer` on the wire.
- `is_busy_pin_inverted_()` per class picks busy polarity (UC8179 returns true; SSD16xx false).

## Architecture: epaper_spi (FSM port)

`EPaperBase` (`epaper_spi.{h,cpp}`) is a **non-blocking state machine** driven from `loop()`:

```
IDLE → UPDATE → RESET → RESET_END → INITIALISE → TRANSFER_DATA
     → POWER_ON → REFRESH_SCREEN → POWER_OFF → DEEP_SLEEP → IDLE
```

States numerically greater than `SHOULD_WAIT` **auto busy-wait before executing** — so a refresh that takes seconds never blocks `loop()`. Concrete drivers implement the hooks: `transfer_data()` (chunk RAM writes to ≤`MAX_TRANSFER_SIZE`, yield every `MAX_TRANSFER_TIME` ms, return `false` to resume next loop), `refresh_screen(partial)`, `power_on/off()`, `deep_sleep()`, and optionally `initialise(partial)` / `reset()`.

Busy polarity is **built into the driver**, not the YAML: base `is_idle_()` respects `busy_invert_` (UC8179 sets it true = active-LOW; SSD16xx leaves it false = active-HIGH). Do **not** add `busy_pin: {inverted: true}` in YAML for these.

Mono color mapping follows **upstream's luminance model** (`EPaperBase::color_to_bit`, split at `r+g+b >= 382`), **not** the `waveshare_epaper` family's `is_on` foreground-key convention. This is WYSIWYG: `COLOR_ON` (white) → white paper, `COLOR_OFF`/black → ink. Draw ink with a **dark** color, not the default `COLOR_ON`. Keep this aligned with upstream — do not "fix" it back to `is_on`. 3-color (BWR) drivers instead use `color_to_bwr` in `epaper_spi_bwr_color.h`.

### Drivers are organized by controller IC, not brand

| C++ class | file | IC / mode |
|---|---|---|
| `EPaperUC8179Base` → `EPaperUC8179` / `EPaperUC8179BWR` | `epaper_spi_uc8179*` | UC8179 mono / 3-color |
| `EPaperSSD1608` | `epaper_spi_ssd1608.*` | SSD1608/IL3820 mono |
| `EPaperSSD1683` | `epaper_spi_ssd1683.*` | SSD1683 3-color |

`EPaperUC8179Base` is shared because UC8179 is one IC with two color modes. SSD1608 and SSD1683 are **different ICs** — do not lump them under an "SSD16xx" base; each is self-contained. UC8179 mono does true differential partial updates (tracks the previous frame in a software `old_data_` buffer since the panel deep-sleeps between updates); the flashing full refresh is forced every `full_update_every`.

### Model registry (epaper_spi)

`display.py` auto-discovers every `models/*.py` (`pkgutil.iter_modules`), so **dropping a `.py` in `models/` registers it** — no central list. Each module instantiates `EpaperModel` (or `RequiredPinsModel`, which forces reset/busy required) subclasses. Files are named by IC (`uc8179.py`, `ssd1608.py`, `ssd1683.py`).

- `class_name` (string) → the C++ class (`epaper_spi_ns.class_(...)`).
- `get_init_sequence(config)` returns tuples `(cmd, data...)`, flattened by `mipi.flatten_sequence`; a bare `(0x04,)` is a no-data command, `(DELAY_FLAG, ms)` a delay. Power-on (0x04) for UC8179 is issued in the `POWER_ON` state, not the init sequence.
- `get_constructor_args(config)` supplies **extra C++ ctor args after `(name, w, h, seq, len)`** — this is how **panel-specific data reaches a generic IC driver**. Waveform LUTs belong to the *panel*, not the IC: e.g. `ssd1608.py` passes each panel's `lut_full`/`lut_partial` via `cg.static_const_array`, so a new SSD1608 panel is one registration + its LUT, no C++ change.

## Conventions

- Code and comments in **English**; commit messages follow Conventional Commits.
- Panel model names use dashes (`p750057-mf1-a`), matched case-insensitively with `space="-"`.
- Provenance matters in comments/docstrings: e.g. `DEPG0420` is a **DKE** panel (SSD1683), not Good Display, though it's a GDEY042Z98 equivalent.
- Reference vendor demo code / GxEPD2 when porting a new panel's init/LUT, and verify busy polarity + data-bit polarity against the controller datasheet before trusting an AI-derived sequence — panels ported this way are often only compile-verified, not hardware-verified.

---
> Source: [parkghost/esphome-epaper](https://github.com/parkghost/esphome-epaper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
