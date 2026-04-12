## pebble-sky

> Analog watchface with GMT and annual calendar complications for the Pebble Round 2 (platform codename: **gabbro**).

# Sky-Pebble — Pebble Round 2 Watchface

Analog watchface with GMT and annual calendar complications for the Pebble Round 2 (platform codename: **gabbro**).

## Project Overview

A watchface featuring analog hands, 24-hour subdial, month indicator ring, date window, and clean polished dial edge — all rendered procedurally in C on a 260x260 64-color round display.

## Tech Stack

- **Language**: C (Pebble SDK)
- **SDK**: PebbleOS SDK 4.9.148 via `pebble-tool` (installed with `uv tool install pebble-tool --python 3.13`)
- **Target**: Gabbro platform (Pebble Round 2, 260x260, 64-color, round)
- **Build**: `pebble build` (waf-based, produces `build/pebble-sky.pbw`)
- **Test**: `pebble install --emulator gabbro` (QEMU)

## Build & Test Commands

```bash
# Build
pebble build

# Install in emulator (requires qemu-pebble on PATH — see Toolchain note below)
pebble install --emulator gabbro

# Capture screenshot
pebble screenshot --no-open --emulator gabbro screenshot_gabbro.png

# View logs
pebble logs --emulator gabbro

# Install on physical watch
pebble install --cloudpebble
```

### Toolchain PATH Setup

The QEMU emulator and toolchain live inside the SDK directory. You need these on PATH for emulator commands:

```bash
export PATH="$HOME/.local/bin:$HOME/Library/Application Support/Pebble SDK/SDKs/4.9.148/toolchain/bin:$PATH"
export DYLD_LIBRARY_PATH="$HOME/Library/Application Support/Pebble SDK/SDKs/4.9.148/toolchain/lib"
```

### Emulator Controls

```bash
# Set watch time (HH:MM:SS format)
pebble emu-set-time --emulator gabbro 10:10:16

# Bluetooth connection (triggers connection_service callback)
pebble emu-bt-connection --connected no --emulator gabbro   # disconnect
pebble emu-bt-connection --connected yes --emulator gabbro  # reconnect

# Accelerometer (presets: tilt-left/right/forward/back, gravity±x/y/z, none)
pebble emu-accel --emulator gabbro gravity+x   # rapid gravity flip triggers tap detection
pebble emu-accel --emulator gabbro gravity-x

# Battery level
pebble emu-battery --emulator gabbro --percent 75
```

### Generating Screenshots

All screenshots use 10:10:16 AM as the display time. Set time with `emu-set-time` before capturing.

- `screenshot_gabbro.png` — main watchface (date in window)
- `screenshot_battery.png` — battery indicator (trigger via gravity flip for tap detection)
- `screenshot_bluetooth.png` — BT disconnect (red Bluetooth rune in date window)

## Project Structure

```
src/c/main.c          # All watchface code (single-file C)
package.json          # Pebble project manifest (UUID, platform targets)
wscript               # Build configuration (waf)
resources/            # GMT disc bitmap (gmt_disc.png, 154×154 rotatable 24h ring)
build/pebble-sky.pbw  # Built watchface artifact
```

## Key Constraints (Pebble C Development)

- **No floating point** — use `sin_lookup()` / `cos_lookup()` with `TRIG_MAX_ANGLE` (65536 = full circle) and `TRIG_MAX_RATIO` (65536)
- **Tick timer** — dynamically switches between `SECOND_UNIT` (seconds hand visible) and `MINUTE_UNIT` (seconds hand hidden) based on accelerometer motion detection with a 30-second inactivity timeout
- **Pre-allocate GPath** — create paths in `window_load`, destroy in `window_unload`
- **Dynamic bounds** — use `layer_get_bounds()`, never hardcode 260x260
- **Antialiasing** — call `graphics_context_set_antialiased(ctx, true)` for crisp edges; use odd stroke widths (1, 3, 5) for best pixel alignment
- **64-color palette** — use `GColor*` constants (e.g., `GColorOxfordBlue`, `GColorWhite`)
- **Warnings are errors** — the compiler uses `-Werror`. No unused variables, no multi-line comments with backslashes

## Design Elements

The watchface features these classic GMT complications:

1. **Analog hands** — skeletonized hour/minute hands with transparent cutouts (4-piece fill: base, left rail, right rail, outer tip) and luminous strips, plus tapered seconds hand
2. **24-hour GMT ring** — off-center rotating disc showing UTC time (via `gmtime()`), with red/white inverted triangle pointer
3. **Month indicator ring** — 12 small squares around the dial at each hour position; current month shown in red, others white (Jan=1 o'clock through Dec=12 o'clock)
4. **Date window** — white rectangle at 3 o'clock showing day of month with lens; on Bluetooth disconnect, shows red Bluetooth rune (persistent until reconnected) with double-pulse vibration; on wrist tap, temporarily shows battery level (color-coded: green ≥40%, yellow ≥20%, red <20%) for 3 seconds. Priority: battery > BT disconnect > date
5. **Clean dial edge** — polished light/dark concentric rings (no fluted bezel)
6. **Baton indices** — rectangular hour markers with light grey border, uniform size at all positions (skipping 12 and 3 o'clock)
7. **Brand text** — "SKY-PEBBLE" below center, decorative marks at bottom flanking 6 o'clock
8. **Earth/ZULU** — globe icon with "ZULU" label at 12 o'clock

## Git Workflow

The user manages their own commits. Claude should suggest commit messages but never run `git commit`, `git push`, or `gh repo create` directly.

## Skill Reference

The `.claude/skills/pebble-watchface/` directory contains the Pebble watchface agent skill with:
- `reference/pebble-api-reference.md` — full Pebble C API docs
- `reference/drawing-guide.md` — graphics primitives, coordinate system, patterns
- `reference/animation-patterns.md` — animation techniques
- `templates/` — starter templates for static, animated, and weather watchfaces
- `scripts/` — icon generation, GIF preview, project validation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ehjacobs)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ehjacobs)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
