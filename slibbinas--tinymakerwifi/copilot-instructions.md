## tinymakerwifi

> Modified firmware for the open-source **TinyMaker** MSLA resin 3D printer (ESP32-WROOM-32E-N4, 4 MB flash, no PSRAM). Adds WiFi, OTA updates, and direct model upload from PrusaSlicer/UVtools to the original TinyMaker3D firmware. Full user-facing docs are in [README.md](README.md).

# TinyMakerWiFi

Modified firmware for the open-source **TinyMaker** MSLA resin 3D printer (ESP32-WROOM-32E-N4, 4 MB flash, no PSRAM). Adds WiFi, OTA updates, and direct model upload from PrusaSlicer/UVtools to the original TinyMaker3D firmware. Full user-facing docs are in [README.md](README.md).

## Build

PlatformIO project (`platformio.ini`), not Arduino IDE. Two environments:

- `tinymaker` — USB flashing (CH340, `upload_port = COM5`). Build this first on a fresh board.
- `tinymaker-ota` — wireless flashing over WiFi (`upload_protocol = espota`, `tinymaker.local`). Only works once `tinymaker` is already installed.

```
pio run                  # builds both envs
pio run -e tinymaker      # USB env only
pio run -e tinymaker-ota  # OTA env only
pio run -t upload -e tinymaker       # flash over USB
pio run -t upload -e tinymaker-ota   # flash over WiFi
```

PlatformIO CLI is not on PATH in this environment; invoke it via
`~/.platformio/penv/Scripts/platformio.exe`.

Build output goes to `C:/PIO-build/TinyMakerWiFi` (set via `build_dir` in `platformio.ini`, deliberately outside the repo/Google Drive to avoid sync conflicts and locked files).

**Do not upgrade** `platform = espressif32@6.5.0` (Arduino core 2.0.14) to 3.x — the vendored `Arduino_GFX` 1.2.0 in `lib/` is incompatible with it.

### firmware.bin vs firmware-full.bin

- `firmware.bin` (app only) → used for OTA/wireless updates (`http://tinymaker.local/update` or `tinymaker-ota` env). PlatformIO produces this automatically.
- `firmware-full.bin` (bootloader + partition table + app merged) → required for the **first** USB flash on a blank board, at address `0x0`. Auto-generated after every `tinymaker` build by [scripts/post_merge_bin.py](scripts/post_merge_bin.py) (an `extra_scripts` post-action in `platformio.ini`, using `esptool merge_bin`). It is **not** produced for the `tinymaker-ota` env — that firmware is never USB-flashed.

Both files land in `C:/PIO-build/TinyMakerWiFi/tinymaker/`.

### Versioning & self-update

- Version scheme is the fork's own **SemVer** (`FIRMWARE_VERSION` in `platformio.ini`, e.g. `0.7.0`) — the upstream `1.0.2` base is no longer carried in the version string (only noted on the About screen). Bump it per release.
- The `System → Update` screen shows installed vs. latest and can **self-update**: it fetches `version.txt` from GitHub Pages (`OTA_VERSION_URL` in `Network.ino`) over HTTPS, compares SemVer, and if newer, pulls `firmware.bin` via `httpUpdate` and reboots. All firmware flashing (web `/update`, espota, self-update) is gated to `screen == 421` for safety.
- Hosting setup (what to publish on the `gh-pages` branch) is documented in [Firmware_Hosting/](Firmware_Hosting/).

## Source layout

Arduino-style: all `.ino` files in `src/` are concatenated into one translation unit, so any edit anywhere triggers a full relink. No `.h`/`.cpp` split except vendored fonts.

| File | Responsibility |
|---|---|
| `TinyMaker.ino` | Main firmware — `setup()`/`loop()`, state machine (`screen` variable drives UI + print flow), original TinyMaker3D logic |
| `Network.ino` | WiFiManager captive portal, mDNS, minimal OctoPrint API (`/api/version`, `/api/files/local`) so PrusaSlicer's "Send to printer" works, `/upload`, ZIP/SL1 unpacking to SD, OTA update endpoint |
| `Interface.ino` | All `screenN()` UI drawing functions for the ST7789/ST7735 display (Arduino_GFX) |
| `Folder.ino` | SD card folder/file navigation for the Print menu |
| `Motor.ino` | Z-axis stepper control (AccelStepper) — manual lift, homing |
| `TimeCalculate.ino` | Lookup-table-based timing calculations for lift/retract cycles |
| `UVLED.ino` | UV LED exposure control during printing |
| `PNG.ino` | PNG layer decoding (PNGdec) from SD, white-pixel counting for resin volume estimate |

`lib/` holds four vendor-verified libraries unpacked from the original TinyMaker3D `Firmware/Libraries/*.zip` — **do not replace with registry versions** (APIs changed): `AccelStepper` 1.64, `Arduino_GFX` 1.2.0, `PNGdec` 1.0.1, `SdFat` 1.1.2.

Build-time switches (top of `TinyMaker.ino`):
```cpp
#define ENABLE_NETWORK       1   // 0 = original network-free firmware
#define ENABLE_SERIAL_DEBUG  1   // 0 = no serial output
```

## Remotes

- `origin` → `slibbinas/TinyMakerWiFi` (this fork, active development)
- `upstream` → `TinyMaker3D/TinyMaker-Open-Source-3D-Printer` (original project)

---
> Source: [slibbinas/TinyMakerWifi](https://github.com/slibbinas/TinyMakerWifi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
