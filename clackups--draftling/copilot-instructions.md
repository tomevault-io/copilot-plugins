## draftling

> Manages WiFi in station mode. Reads credentials from NVS or from

# Copilot Instructions

## Code Style

- Do not use non-ASCII characters in code, comments, or string literals. All source files must be ASCII-only.

## Project Overview

Draftling is a distraction-free Markdown text editor for ESP32-S3-based
development boards with reflective LCD displays. It is built with the
ESP-IDF framework (v5.3+) and uses LVGL v9 for the graphical interface.

The user connects a Bluetooth keyboard and edits Markdown files stored on
a MicroSD card. The reflective LCD needs no backlight and works well in
daylight. On request the device connects to WiFi and synchronizes files
with a remote Git repository via the GitHub REST API.

### Supported Hardware

| Board | Display |
|-------|---------|
| Waveshare ESP32-S3-RLCD-4.2 | 4.2-inch reflective LCD, 400x300 |
| Seeed Studio reTerminal E1001 | 7.5-inch e-paper (UC8179), 800x480 |
| Waveshare E-Paper Driver HAT (on any BLE-capable ESP32 host, default ESP32-S3-DevKitC-1 wiring) | UC8179 e-paper, panel preset (default Waveshare 7.5" V2 / GDEW075T7, 800x480) |
| M5Stack PaperS3 | 4.7-inch e-paper (ED047TC1), 540x960 |

## Repository Layout

```
CMakeLists.txt              Top-level CMake project file
partitions.csv              Custom partition table (16 MB flash)
sdkconfig.defaults          Common Kconfig defaults for all targets
sdkconfig.defaults.esp32s3  ESP32-S3-specific defaults (PSRAM, BLE, WiFi)
main/                       Application entry point and hardware config
  main.cpp                  app_main(): initializes all subsystems
  app_config.h              Pin definitions and display macros per board
  Kconfig.projbuild         Menuconfig: hardware model, display size, rotation
  idf_component.yml         IDF component manifest (depends on lvgl ^9.2)
  CMakeLists.txt            Registers main as an IDF component
components/                 Reusable IDF components
  battery/                  Battery voltage monitor (ADC)
  ble_keyboard/             BLE HID keyboard host (Bluedroid)
  display/                  RLCD SPI display driver and LVGL port
  editor/                   Gap-buffer text editor, Markdown parser, LVGL UI
  fonts/                    Custom LVGL bitmap fonts (Greybeard family)
  git_sync/                 GitHub REST API file synchronization
  kb_layout/                Keyboard layout translation (US/UA/DE/FR)
  sd_card/                  SD card (SDMMC 1-bit) file operations
  standby/                  Deep-sleep / standby inactivity timer
  wifi_manager/             WiFi STA connection manager
```

## Component Details

### main/

The application entry point. `app_main()` in `main.cpp` initializes every
subsystem in order: NVS flash, display hardware, LVGL, battery monitor,
editor UI, SD card, BLE keyboard, WiFi manager, Git sync, and standby
timer. It also registers an auto-save callback that persists the current
document before entering deep sleep.

`app_config.h` maps Kconfig hardware model selections to concrete GPIO pin
numbers and display dimensions. Each supported board has its own `#if`
block defining SPI pins (MOSI, SCK, DC, CS, RST, TE/BUSY), SD card pins
(CLK, CMD, D0 for SDMMC, or MOSI/MISO/SCK/CS for SPI), I2C pins, the
battery ADC pin, and the deep-sleep wakeup GPIO. The Waveshare E-Paper
Driver HAT block resolves all of its pin macros from `CONFIG_DRAFTLING_HAT_*`
Kconfig values so users can adapt the HAT to a different ESP32-S3 host
board without editing source. The M5Stack PaperS3 block omits the panel
data-bus and control-line pins because the `m5stack/M5GFX` library
configures them internally based on the board id.

### components/battery/

Reads battery voltage through a resistive divider on a configurable
ADC pin using the ESP32 ADC. Applies exponential moving average
smoothing over 8 samples. Maps voltage to a percentage: >=4.10V is
100%, >=3.95V is 75%, >=3.80V is 50%, >=3.60V is 25%, below 3.60V
is 0%. The M5Stack PaperS3 reads its cell through ADC1 on GPIO3 with
a 1:2 divider, matching the M5Unified Power_Class configuration for
that board. The bare Waveshare EPD HAT has no on-board battery and
passes `BATT_ADC_PIN = -1`, which makes `battery_init()` a no-op and
causes `battery_read_percent()` to return -1; the editor UI hides the
battery icon in that case.

Public API: `battery_init()`, `battery_read_mv()`, `battery_read_percent()`.

### components/ble_keyboard/

BLE HID keyboard host built on ESP-IDF Bluedroid. Handles device scanning,
pairing with passkey authentication, connection/disconnection callbacks,
and keyboard event dispatching. Each key event carries the HID keycode,
ASCII character, modifier flags, and pressed/released state.

Public API: `ble_keyboard_init()`, `ble_keyboard_start_scan()`,
`ble_keyboard_is_connected()`, `ble_keyboard_set_callback()`, and
several other callback registration functions.

### components/display/

Per-board display backends behind a single C API:

- **display_rlcd.cpp** -- Waveshare ESP32-S3-RLCD-4.2 reflective LCD
  over SPI.
- **display_uc8179.cpp** -- UC8179 e-paper over SPI. Shared by the
  Seeed reTerminal E1001 and the Waveshare E-Paper Driver HAT;
  resolution and pinout (including BUSY) are passed in at init.
  When `DRAFTLING_EPD_FAST_PARTIAL` is set (auto-enabled by the
  GDEW075T7 panel preset on the HAT) the driver loads custom
  single-stage partial-refresh LUTs into UC8179 registers
  `0x20`-`0x25` and switches PSR to register-LUT mode (`0x3F`) on
  the first partial after init or after a periodic full refresh.
  Unchanged-pixel LUT entries are zero-drive so the screen border
  stops flashing on every keystroke. Under `DRAFTLING_EPD_INVERT`
  (also auto-enabled for the GDEW075T7 preset) the LUT level bytes
  for the K->W and W->K transitions are pair-swapped at compile time
  (0x5A <-> 0xA5, 0x84 <-> 0x48) so the GxEPD2 "kick-then-pull"
  overdrive pattern produces the same visual transition on a panel
  whose framebuffer convention has visual-white = bit 0. Mode is
  switched lazily so a burst of partials shares one upload; the
  full-refresh path restores PSR `0x1F` for the OTP ghost-clearing
  waveform. The LUT timings are calibrated against the GDEW075T7
  panel revision; other 7.5" panels added later should declare their
  own `DRAFTLING_HAT_PANEL_*` preset and either inherit the same
  `DRAFTLING_EPD_FAST_PARTIAL = y` if they share the waveform, or
  leave it off and fall back to the OTP partial waveform (always
  works on UC8179 panels but flashes the border once per refresh).
- **display_eds3.cpp** -- M5Stack PaperS3 ED047TC1 panel via the
  `m5stack/M5GFX` managed component. Unlike the other backends this
  one does not maintain its own 1-bpp framebuffer; M5GFX already
  keeps a 4-bpp grayscale framebuffer internally, and the LVGL port
  pushes RGB565 pixels straight into it through the optional
  `display_push_rgb565()` fast path. The backend accumulates a
  dirty bounding box across pushes and, on `display_flush()`, calls
  `M5GFX::display(x,y,w,h)` over that rect using the single-pulse
  `epd_fast` waveform (one visible flash, ~80-150 ms);
  every `CONFIG_DRAFTLING_EPD_FULL_REFRESH_INTERVAL` partials (or
  whenever the dirty area covers most of the screen, or after
  `display_clear()` / `display_full_refresh()`) the next refresh is
  promoted to a full-screen `epd_quality` pass to clear ghosting.
  Two extras keep typing snappy: a 120 ms flush debounce coalesces
  bursts of keystrokes into a single panel refresh (deferred via
  `esp_timer`, taking `lvgl_port_lock()` for thread safety), and the
  optional one-shot `display_set_partial_clip(x,y,w,h)` API lets the
  editor narrow the next refresh to the area around the typed
  character (cursor + edited columns) when it knows the rest of the
  LVGL-pushed line pixels are unchanged. The backend also honours
  `CONFIG_DRAFTLING_DISPLAY_SCALE` (default 2 on PaperS3): every
  logical LVGL pixel is rendered as SCALE x SCALE physical panel
  pixels via nearest-neighbor expansion in `display_push_rgb565()`,
  and the dirty-bbox / partial-clip math is converted to panel
  coordinates internally. Grayscale UI is still TODO.
- **lvgl_port.cpp** -- creates the LVGL display object, sets up a
  flush callback that first tries `display_push_rgb565()` (used by
  the PaperS3 backend) and otherwise converts LVGL's RGB565 output
  to the backend's 1-bpp pixel format via `display_set_pixel()`, and
  runs the LVGL tick/task timer. Thread safety is provided by a
  mutex exposed as `lvgl_port_lock()` / `lvgl_port_unlock()`.

The component's `idf_component.yml` declares the `m5stack/m5gfx`
dependency required by the PaperS3 backend. (It is downloaded for
every build; the source itself is gated by `#if defined(CONFIG_DRAFTLING_MODEL_M5STACK_PAPERS3)`
so non-PaperS3 builds do not link it into the final image.)

Public API: `display_init()`, `display_clear()`, `display_set_pixel()`,
`display_flush()`, `display_full_refresh()`, `display_push_rgb565()`,
`display_set_partial_clip()`, `display_get_buffer()`,
`display_get_buffer_size()`, `lvgl_port_init()`,
`lvgl_port_lock()`, `lvgl_port_unlock()`.

### components/editor/

The largest component. Contains:

- **editor.cpp** -- gap-buffer text engine (default 256 KB document
  limit, configurable via `CONFIG_DRAFTLING_EDITOR_BUFFER_SIZE_KB` in
  menuconfig; the gap buffer and flat cache both live in PSRAM) with
  cursor movement, selection, clipboard, insert/delete, and file I/O.
- **editor_ui.cpp** -- LVGL-based user interface: title bar with battery
  and layout indicators, scrollable text area with Markdown rendering,
  status bar, file browser dialog, and settings menu (F1).
- **md_parser.cpp** -- single-pass Markdown line parser. Recognizes
  headings H1-H4, bullet and numbered lists, blockquotes, code fences,
  horizontal rules, and inline bold/italic/code/strikethrough spans.
- **draftling_logo.c** -- embedded LVGL image for the splash screen.

Public API: `editor_init()`, `editor_open_file()`, `editor_save_file()`,
`editor_ui_init()`, `editor_ui_handle_key()`, `editor_find()`,
`editor_replace_range()`, `md_parse_line()`, and many
cursor/selection/clipboard helpers.

Editor shortcuts include `Ctrl+F` (Find) and `Ctrl+H` (Find +
Replace). Both open a modal overlay; in Find+Replace mode, `Tab`
switches between the Find and Replace fields, `Enter` jumps to the
next match (wrapping at end-of-document), and `Ctrl+Enter` replaces
the current match and advances to the next.

The title bar shows `L %d/%d` (current line / total lines) on every
build; on non-EPD targets the column counter is appended as well.

The bottom status bar of both the editor and the file browser
displays a small Wi-Fi icon (`components/editor/wifi_icon.c`) in the
right corner whenever `wifi_manager_is_connected()` is true. The
Greybeard fonts cover U+0020-U+04FF plus a few currency / numero
glyphs, which excludes the U+1F6DC "wireless" pictograph, so the
icon is rendered from a small embedded LVGL `LV_COLOR_FORMAT_I1`
image instead of as a font glyph. Two pre-baked descriptors are
exposed (black-on-transparent for the default theme and
white-on-transparent for `CONFIG_DRAFTLING_EPD_BLACK_BACKGROUND`).

### components/fonts/

Custom LVGL bitmap fonts generated from the Greybeard typeface. See the
dedicated section below for the full creation process.

### components/git_sync/

Synchronizes Markdown files between the SD card and a remote GitHub
repository using the GitHub REST API over HTTPS. Reads configuration
(repo URL, branch, token, path prefix) from `/sdcard/git.cfg`. Supports
pull, push, or bidirectional sync with state callbacks and error tracking.

Public API: `git_sync_init()`, `git_sync_start()`,
`git_sync_get_state()`, `git_sync_is_configured()`.

### components/kb_layout/

Translates HID keycodes and modifier flags into UTF-8 character strings
for the active keyboard layout. Supports US-English (QWERTY), Ukrainian
(Cyrillic), German (QWERTZ), and French (AZERTY). Each layout can be
independently enabled or disabled at build time via Kconfig (see
`components/kb_layout/Kconfig.projbuild`). The active layout is cycled
at runtime with `kb_layout_next()`.

Public API: `kb_layout_translate()`, `kb_layout_set()`, `kb_layout_get()`,
`kb_layout_name()`, `kb_layout_next()`.

### components/sd_card/

Initializes the MicroSD card over SDMMC 1-bit interface and mounts it as
a FAT filesystem at `/sdcard`. Provides standard file operations (read,
write, append, delete, rename, existence check, size query) and directory
operations (mkdir, list).

Public API: `sd_card_init()`, `sd_card_read_file()`,
`sd_card_write_file()`, `sd_card_list_dir()`, `sd_card_file_exists()`,
and others.

### components/standby/

Monitors user inactivity and enters ESP32 deep sleep after a configurable
timeout (default 600 seconds / 10 minutes). The timeout is persisted in
NVS so it survives reboots. A pre-sleep callback allows the editor to
auto-save before power-down. On the Waveshare RLCD board, wakeup is
triggered by pressing the GPIO18 button (EXT0, active-low). On the
M5Stack PaperS3 the wake source is the BOOT button on GPIO0 (EXT0,
active-low) -- the only RTC-capable user-input GPIO on the board.
Earlier revisions tried GPIO21 (wrong -- that's the buzzer) and
GPIO48 (the GT911 touch INT) with a light-sleep + `esp_restart()`
workaround; both woke the device immediately, the latter because
M5GFX initializes only the e-paper panel (not the touch controller),
so the GT911 is left uninitialized and holds INT low.

Before arming EXT0, `standby_enter_sleep()` enables the chip's
internal RTC pull-up on the wake GPIO and disables any pull-down.
The supported branded boards (RLCD-4.2 button, reTerminal KEY0,
PaperS3 BOOT strapping pin) all have external pull-ups, so they
worked without it; but the Waveshare E-Paper Driver HAT runs on a
generic ESP32 host where the user-selected wake pin
(`CONFIG_DRAFTLING_HAT_WAKEUP_GPIO`) may be a bare GPIO with no
external pull-up. Without an internal pull-up that line floats LOW
and EXT0 fires immediately, making the device boot back up the
instant it tries to deep-sleep.

In addition to the inactivity timer, `standby_init()` also arms a
"no keyboard connected" countdown of `CONFIG_DRAFTLING_NO_KEYBOARD_SLEEP_SEC`
seconds (default 180, 0 = disabled). When the timer fires it polls
`ble_keyboard_is_connected()` and only enters deep sleep if no
Bluetooth keyboard has paired by then. This conserves battery when
the device is powered on accidentally or no paired keyboard is in
range.

Public API: `standby_init()`, `standby_reset_timer()`,
`standby_set_timeout()`, `standby_set_pre_sleep_cb()`,
`standby_enter_sleep()`.

### components/wifi_manager/

Manages WiFi in station mode. Reads credentials from NVS or from
`/sdcard/wifi.cfg`. Provides connect, disconnect, and state query
functions with an event callback for connection state changes (idle,
connecting, connected, disconnected, error). Required by `git_sync` for
network access.

Public API: `wifi_manager_init()`, `wifi_manager_connect()`,
`wifi_manager_disconnect()`, `wifi_manager_is_connected()`,
`wifi_manager_get_ip()`, `wifi_manager_get_ssid()`.

## Font Creation Process

The `components/fonts/` directory contains custom LVGL bitmap fonts
generated from the **Greybeard** typeface, a monospaced bitmap font that
is a vector port of UW ttyp0 (MIT-licensed, source:
https://github.com/flowchartsman/greybeard).

### Source Files

Greybeard ships as a set of TTF files, each designed for a single native
pixel size. Because the outlines trace exact pixel boundaries, rendering
at the native size with 1-bit-per-pixel (no antialiasing) produces
pixel-perfect glyphs with no scaling artifacts.

### Generation Tool

The fonts were converted to LVGL C source files using **lv_font_conv**,
the official LVGL font conversion utility. The exact command line is
recorded in the header comment of each generated `.c` file. For example,
the 14 px font was generated with:

```
lv_font_conv \
  --font Greybeard-14px.ttf \
  -r 0x20-0x7F,0xA0-0xFF,0x400-0x4FF,0x20AC,0x20B4,0x2116 \
  --size 14 \
  --bpp 1 \
  --format lvgl \
  --no-compress \
  --lv-font-name greybeard_14 \
  -o greybeard_14.c
```

### Unicode Ranges

Every font file includes the same set of Unicode ranges so that all
keyboard layouts (US, UA, DE, FR) can share a single font set:

| Range | Description |
|-------|-------------|
| U+0020 - U+007F | Basic Latin (ASCII printable characters) |
| U+00A0 - U+00FF | Latin-1 Supplement (accented Latin characters, symbols) |
| U+0400 - U+04FF | Cyrillic (Ukrainian and other Slavic alphabets) |
| U+20AC | Euro sign |
| U+20B4 | Hryvnia sign (Ukrainian currency) |
| U+2116 | Numero sign |

### Sizes and Metrics

Six sizes are provided. All except the 26 px variant are rendered at
their native TTF pixel size. The 26 px font is scaled from the 22 px
TTF source.

| File | Pixel Size | Char Width | Line Height | Notes |
|------|-----------|------------|-------------|-------|
| greybeard_11.c | 11 | 6 | 11 | Smallest, used for compact UI elements |
| greybeard_14.c | 14 | 7 | 13 | Default body text |
| greybeard_16.c | 16 | 8 | 15 | |
| greybeard_18.c | 18 | 9 | 17 | |
| greybeard_22.c | 22 | 11 | 21 | Headings |
| greybeard_26.c | 26 | 13 | 25 | Largest heading (scaled from 22 px TTF) |

All fonts are declared in `components/fonts/include/greybeard.h` as
`extern const lv_font_t greybeard_NN` and compiled as an IDF component
that depends on `lvgl__lvgl`.

### Rendering Settings

- **Bits per pixel (bpp):** 1 (pure black and white, no antialiasing).
  This matches the reflective LCD which has no grayscale capability.
- **Compression:** disabled (`--no-compress`) for faster glyph lookup on
  the microcontroller.
- **Format:** `lvgl` (native LVGL font structure).

## Hardware Definitions in Kconfig.projbuild

There are two Kconfig.projbuild files that expose project-specific
menuconfig options.

### main/Kconfig.projbuild -- Hardware Configuration

This file defines the **DRAFTLING Configuration** menu with the following
options:

#### Hardware Model (DRAFTLING_HARDWARE_MODEL)

A `choice` that selects the target board. The first three options are
ESP32-S3-only (`depends on IDF_TARGET_ESP32S3`); the Waveshare HAT is
selectable on any ESP-IDF target with a BLE radio (`depends on
SOC_BLE_SUPPORTED`):

- **DRAFTLING_MODEL_WAVESHARE_RLCD42** -- Waveshare ESP32-S3-RLCD-4.2
  with a 400x300 reflective LCD and GPIO18 deep-sleep wakeup button.
  *Requires ESP32-S3.*
- **DRAFTLING_MODEL_SEEED_RETERMINAL_E1001** -- Seeed reTerminal E1001
  with a 7.5" 800x480 UC8179 e-paper, SD card on the same SPI bus,
  GPIO3 (KEY0) deep-sleep wakeup. *Requires ESP32-S3.*
- **DRAFTLING_MODEL_WAVESHARE_EPD_HAT** -- Waveshare E-Paper Driver
  HAT (UC8179) on any BLE-capable ESP32 host (ESP32, ESP32-S3,
  ESP32-C2/C3/C6, ESP32-H2 - i.e. anything except the BLE-less
  ESP32-S2). The connected panel is picked via the `DRAFTLING_HAT_PANEL`
  choice (see below); only the GPIO pinout (sub-menu "Waveshare E-Paper
  Driver HAT pinout") is user-editable. Pin defaults match the
  ESP32-S3-DevKitC-1 wiring used by Waveshare's example projects.
- **DRAFTLING_MODEL_M5STACK_PAPERS3** -- M5Stack PaperS3 with a
  4.7" 540x960 ED047TC1 e-paper driven by the `m5stack/M5GFX`
  library, on-board MicroSD on SPI3, BOOT button on GPIO0 used as
  the EXT0 deep-sleep wake source (the only RTC-capable user-input
  GPIO on the board). *Requires ESP32-S3.*

The hardware-model selection (and, for the HAT, the
`DRAFTLING_HAT_PANEL` panel preset) drives two non-prompted `int`
symbols consumed in `main/app_config.h` as `DISPLAY_WIDTH` /
`DISPLAY_HEIGHT`:

- **DRAFTLING_DISPLAY_WIDTH** -- 400 (RLCD), 800 (reTerminal),
  800 (HAT / GDEW075T7 preset), 960 (PaperS3).
- **DRAFTLING_DISPLAY_HEIGHT** -- 300 (RLCD), 480 (reTerminal),
  480 (HAT / GDEW075T7 preset), 540 (PaperS3).

Both symbols are non-prompted (no menuconfig entry); to support a
new HAT panel with a different resolution, add a
`DRAFTLING_HAT_PANEL_*` entry and extend the per-panel `default`
lines on these symbols.

#### E-paper panel preset (DRAFTLING_HAT_PANEL)

A `choice` visible only when `DRAFTLING_MODEL_WAVESHARE_EPD_HAT` is
selected. Each entry is a single panel preset that locks in the
panel's resolution (`DRAFTLING_DISPLAY_WIDTH` / `_HEIGHT`),
framebuffer polarity (`DRAFTLING_EPD_INVERT`), and partial-refresh
waveform (`DRAFTLING_EPD_FAST_PARTIAL`).

- **DRAFTLING_HAT_PANEL_GDEW075T7** -- Waveshare 7.5" V2 BW
  (GoodDisplay GDEW075T7, UC8179): 800x480, EPD_INVERT = y,
  EPD_FAST_PARTIAL = y. Confirmed flash-free typing on this
  revision.

To add another panel: append a `config DRAFTLING_HAT_PANEL_<NAME>`
block inside this choice, then extend the per-panel `default` lines
on `DRAFTLING_DISPLAY_WIDTH`, `_HEIGHT`, `DRAFTLING_EPD_INVERT`,
and `DRAFTLING_EPD_FAST_PARTIAL` with the panel's required values.

#### Waveshare E-Paper Driver HAT pinout (DRAFTLING_HAT_*)

A sub-menu that is visible only when `DRAFTLING_MODEL_WAVESHARE_EPD_HAT`
is selected. Each option is an `int` GPIO number with a sensible default
for the ESP32-S3-DevKitC-1:

| Symbol | Default | Use |
|--------|---------|-----|
| `DRAFTLING_HAT_EPD_MOSI_PIN` | 11 | SPI MOSI / DIN |
| `DRAFTLING_HAT_EPD_SCK_PIN`  | 12 | SPI clock |
| `DRAFTLING_HAT_EPD_DC_PIN`   | 13 | Data/command |
| `DRAFTLING_HAT_EPD_CS_PIN`   | 10 | Chip select |
| `DRAFTLING_HAT_EPD_RST_PIN`  | 14 | Reset |
| `DRAFTLING_HAT_EPD_BUSY_PIN` | 9  | BUSY input |
| `DRAFTLING_HAT_EPD_PWR_PIN`  | 7  | Panel power-enable (HIGH = on; driven LOW before deep sleep). -1 if PWR is wired permanently high. |
| `DRAFTLING_HAT_WAKEUP_GPIO`  | 0  | EXT0 deep-sleep wakeup pin |
| `DRAFTLING_HAT_SD_INTERFACE` | None | SD card interface: None / SPI / SDMMC 1-bit. SDMMC is gated to chips with an SDMMC host (ESP32, ESP32-S3). Auto-selects the internal `DRAFTLING_HAT_HAS_SD` symbol when not None. |
| `DRAFTLING_HAT_SD_MOSI/MISO/SCK/CS_PIN` | 35 / 37 / 36 / 34 | SPI pinout, visible only when SPI is selected |
| `DRAFTLING_HAT_SD_SDMMC_CLK/CMD/D0_PIN` | 39 / 38 / 40 | SDMMC 1-bit pinout, visible only when SDMMC is selected |

#### E-paper full-refresh interval (DRAFTLING_EPD_FULL_REFRESH_INTERVAL)

`int` shared by the reTerminal E1001 and HAT models (UC8179 backend)
and the M5Stack PaperS3 (M5GFX backend). Number of partial refreshes
between full refreshes; default 50.

#### Invert e-paper pixel polarity (DRAFTLING_EPD_INVERT)

Non-prompted internal helper `bool` (no menuconfig entry). When set,
the UC8179 driver flips the framebuffer pixel polarity: visual-white
is bit 0 instead of bit 1. Derived automatically:

- `y` for `DRAFTLING_HAT_PANEL_GDEW075T7`
- `n` everywhere else

To add a panel that needs the same flip, set `default y if
DRAFTLING_HAT_PANEL_<NAME>` on the symbol.

#### Fast partial-refresh LUTs (DRAFTLING_EPD_FAST_PARTIAL)

Non-prompted internal helper `bool` (no menuconfig entry). When set,
the UC8179 driver loads custom partial-refresh LUTs into registers
`0x20`-`0x25` and switches the panel to register-LUT mode
(PSR = `0x3F`) the first time a partial refresh runs. The LUTs are
the single-stage waveform from the GxEPD2 community library for the
GDEW075T7 panel: `KW=0x5A, WK=0x84` (pair-swapped to `0xA5` / `0x48`
under `EPD_INVERT`), timing `T1=30, T2=5, T3=30, T4=5, RP=1`. The
W->W and K->K entries use level byte `0x00`, i.e. unchanged pixels
receive **no voltage**, so the screen border stops flashing on every
keystroke and each partial refresh completes in a single ~600 ms
pulse instead of the OTP waveform's three full-frame sweeps.

The driver tracks current mode and switches lazily: a burst of
partial refreshes pays the LUT-upload cost only once. The periodic
full refresh (every `DRAFTLING_EPD_FULL_REFRESH_INTERVAL` partials)
restores PSR = `0x1F` and the original CDI/VDCS so it still does a
proper ghost-clearing OTP pass.

Derived automatically:

- `y` for `DRAFTLING_HAT_PANEL_GDEW075T7`
- `n` everywhere else

The LUT level/timing bytes are calibrated for the Waveshare 7.5"
V2 BW panel revision; new HAT panel presets that share the
GDEW075T7 waveform can opt in by adding `default y if
DRAFTLING_HAT_PANEL_<NAME>` to the symbol. Otherwise the driver
falls back to the panel's OTP partial waveform (always works on
UC8179 panels but flashes the border once per refresh).

#### Display Rotation (DRAFTLING_DISPLAY_ROTATE)

A `choice` that sets the display rotation angle. Options are 0, 90, 180,
and 270 degrees. The default is 0 (no rotation). The selected angle is
exposed as the hidden `int` symbol **DRAFTLING_DISPLAY_ROTATE_ANGLE**,
consumed in `app_config.h` as `DISPLAY_ROTATE`.

#### Display scale factor (DRAFTLING_DISPLAY_SCALE)

User-editable `int` (range 1-8). Logical pixel size: every LVGL pixel
is rendered as SCALE x SCALE physical panel pixels via nearest-neighbor
expansion in the display backend. The editor and LVGL canvas operate
in *logical* coordinates (panel size divided by SCALE); only the
display backend deals in physical panel pixels. Defaults: 2 for the
M5Stack PaperS3 (so the high-density 540x960 panel renders Greybeard
text at a comfortably readable size), 1 for every other board.
Currently only the `display_eds3` backend implements the up-scaling;
on the RLCD and UC8179 backends a value > 1 has no visible effect
because their LVGL framebuffers already match their panel sizes.

Because the symbol is prompted, its value is sticky in `sdkconfig`:
switching `DRAFTLING_HARDWARE_MODEL` does NOT re-apply the per-model
default automatically. Delete `sdkconfig` (or set the field by hand
in menuconfig) when changing boards.

### components/kb_layout/Kconfig.projbuild -- Keyboard Layouts

This file defines the **DRAFTLING Keyboard Layouts** menu. Each layout
is an independent `bool` option that can be enabled or disabled:

| Symbol | Layout | Default |
|--------|--------|---------|
| KB_LAYOUT_ENABLE_US | US-English (QWERTY) | y (enabled) |
| KB_LAYOUT_ENABLE_UA | Ukrainian (Cyrillic) | y (enabled) |
| KB_LAYOUT_ENABLE_DE | German (QWERTZ) | n (disabled) |
| KB_LAYOUT_ENABLE_FR | French (AZERTY) | n (disabled) |

Disabling unused layouts saves flash space because the translation
tables for disabled layouts are excluded from the build. The `kb_layout`
component reads these symbols at compile time to conditionally compile
only the enabled layout tables.

## Building

Requires ESP-IDF v5.3 or later. ESP-IDF 6.0 and newer are not supported
yet because the `m5stack/M5GFX` managed component is not compatible
with ESP-IDF 6.x; the top-level `CMakeLists.txt` enforces this with a
`FATAL_ERROR` on IDF major version >= 6.

PSRAM is required on every supported board. The editor gap buffer
(`CONFIG_DRAFTLING_EDITOR_BUFFER_SIZE_KB`, default 256 KB), the display
framebuffers, the LVGL widget heap (`CONFIG_LV_USE_CUSTOM_MALLOC` routes
through PSRAM), the Git-sync HTTPS response buffers + task stack, and
the Bluedroid host environment (`CONFIG_BT_BLE_DYNAMIC_ENV_MEMORY` plus
`CONFIG_BT_ALLOCATION_FROM_SPIRAM_FIRST`) all assume `MALLOC_CAP_SPIRAM`
is available. The top-level `CMakeLists.txt` aborts the configure step
with a `FATAL_ERROR` if `CONFIG_SPIRAM` is not set. Targets without
on-chip PSRAM support (e.g. ESP32-S2, bare ESP32-C3 modules without
PSRAM) are not supported.

```bash
idf.py set-target esp32s3
idf.py build
idf.py -p /dev/ttyACM0 flash monitor
```

If you update `sdkconfig.defaults` or pull new changes, delete the
generated `sdkconfig` so the defaults are re-applied:

```bash
rm -f sdkconfig
idf.py set-target esp32s3
idf.py build
```

---
> Source: [clackups/draftling](https://github.com/clackups/draftling) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-09 -->
