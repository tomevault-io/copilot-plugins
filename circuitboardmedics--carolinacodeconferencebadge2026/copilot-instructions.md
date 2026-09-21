## carolinacodeconferencebadge2026

> Use when you need a custom init sequence. Requires `reverse_bytes_in_word=True`.

# AGENTS.md — CircuitPython Drive Guidance

## What This Directory Is

This is the root of a CircuitPython microcontroller filesystem, mounted as a USB drive. Files here run directly on the hardware — there is no build step, no compiler invocation, and no package manager.

## Execution Model

- **`code.py`** is the main entry point. CircuitPython automatically runs `code.py` whenever it is saved or the board resets. There is no explicit flash/upload step — saving the file IS deploying it.
- All `.py` files are interpreted directly on the microcontroller. Standard CPython libraries are not available; use CircuitPython's bundled libraries and any `.mpy` or `.py` libraries present in the `lib/` directory.
- There is no pip, no requirements.txt, and no virtual environment. Libraries must be manually copied into `lib/` from the CircuitPython library bundle.
- Do not add type annotations that require `from __future__ import annotations` or other CPython-only imports.

## Hardware — ESP32-S3-WROOM-1-N8

| Item | Detail |
|------|--------|
| Module | Espressif ESP32-S3-WROOM-1-N8 |
| USB D+ | GPIO20 |
| USB D- | GPIO19 |
| Boot Button | GPIO0 |
| Reset Button | EN (Enable pin) |

## Display — HS180S10B (ST7735S, 160×128, 1.77 inch)

| LCD Signal | GPIO | Notes |
|-----------|------|-------|
| SPI Clock (SCK) | GPIO12 | |
| MOSI | GPIO11 | Data in to LCD |
| Reset | GPIO7 | Active low |
| Data/Command (DC) | GPIO6 | High = display data, Low = command |
| Chip Select (CS) | GPIO10 | |
| Backlight | GPIO5 | High = on; PWM-capable |
| FS0 (font chip MISO) | GPIO44 | Font chip data output |
| Font Chip Select | GPIO9 | |

Typical CircuitPython import: `adafruit_st7735r` (the ST7735S is compatible with the ST7735R driver).

## Addressable LEDs

| Item | Detail |
|------|--------|
| Count | 5 |
| Data Pin | GPIO4 |
| Type | WS2812 (NeoPixel) |

Typical CircuitPython import: `neopixel`

## Tactile Switches

All switches are pull-down configured — GPIO reads LOW when pressed.

| Switch | GPIO |
|--------|------|
| Switch 1 | GPIO1 |
| Switch 2 | GPIO2 |
| Switch 3 | GPIO43 |

## Power

- **USB Micro** or **CR123A** battery.
- 3.3 V switching regulator.
- USB power takes priority automatically; protection circuitry disconnects the battery when USB is present.
- The manual power switch controls the battery only — USB always supplies power when connected.

## Common CircuitPython Patterns

### Display initialisation (tested working on CP 10.2.1)

**Key CP9+/CP10 rules:**
- `displayio.FourWire` no longer exists — use `fourwire.FourWire`
- `displayio.Display` is a non-functional stub — use `busdisplay.BusDisplay` or `adafruit_st7735r.ST7735R`
- Both approaches below require `fourwire.FourWire` for the bus

#### Option A — adafruit_st7735r (simpler, recommended)

The library handles its own init sequence and MADCTL internally. Use portrait dimensions with `rotation=0` and a matching portrait bitmap.

```python
import fourwire
import adafruit_st7735r

displayio.release_displays()
spi = busio.SPI(clock=board.IO12, MOSI=board.IO11)

display_bus = fourwire.FourWire(
    spi,
    command=board.IO6,
    chip_select=board.IO10,
    reset=board.IO7,
    baudrate=8_000_000,
)

display = adafruit_st7735r.ST7735R(
    display_bus,
    width=128,
    height=160,
    rotation=0,
    bgr=True,
    auto_refresh=False,
)

# Bitmap must match width×height (128×160), not the landscape screen size
bitmap = displayio.Bitmap(128, 160, 1)
```

#### Option B — busdisplay.BusDisplay (more control)

Use when you need a custom init sequence. Requires `reverse_bytes_in_word=True`.

```python
import fourwire
import busdisplay

display_bus = fourwire.FourWire(
    spi, command=board.IO6, chip_select=board.IO10,
    reset=board.IO7, baudrate=8_000_000,
)

_INIT = (
    b"\x01\x80\x96"   # SWRESET, delay 150 ms
    b"\x11\x80\xff"   # SLPOUT,  delay 500 ms
    b"\x3a\x01\x05"   # COLMOD:  16-bit RGB565
    b"\x36\x01\x60"   # MADCTL:  landscape (MX+MV bits)
    b"\x29\x80\x0a"   # DISPON,  delay 10 ms
)

display = busdisplay.BusDisplay(
    display_bus, _INIT,
    width=128, height=160, rotation=90,
    reverse_bytes_in_word=True,
    auto_refresh=False,
)

# Bitmap is 128×160 (portrait); rotation=90 presents it as landscape
bitmap = displayio.Bitmap(128, 160, 1)
```

#### Backlight

```python
bl = digitalio.DigitalInOut(board.IO5)
bl.direction = digitalio.Direction.OUTPUT
bl.value = True
```

### NeoPixels

```python
import neopixel
pixels = neopixel.NeoPixel(board.IO4, 5, brightness=0.2)
```

### Switches (pull-down — reads LOW when pressed)

```python
import digitalio
btn1 = digitalio.DigitalInOut(board.IO1)
btn1.direction = digitalio.Direction.INPUT
btn1.pull = digitalio.Pull.UP
```

## Workflow Notes

- **Saving `code.py` deploys immediately.** Keep changes small and test incrementally.
- If the board crashes into the REPL, fix the error in `code.py` and save again.
- Serial output (`print()`) appears on the USB serial console (e.g. `screen /dev/ttyACM0 115200` or the Mu editor serial pane).
- Avoid long-running blocking operations without a `supervisor.reload()` escape hatch or watchdog timer if the board needs to stay recoverable.

---
> Source: [circuitboardmedics/CarolinaCodeConferenceBadge2026](https://github.com/circuitboardmedics/CarolinaCodeConferenceBadge2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-21 -->
