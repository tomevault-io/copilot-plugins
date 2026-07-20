## swc-digital

> Read this file and `USB_CLOCK.md` before changing or flashing the clock.

# Smart Weather Clock agent guide

Read this file and `USB_CLOCK.md` before changing or flashing the clock.

## Project goal

Turn the inexpensive Smart Weather Clock into a USB-configurable custom display.
The working baseline is a USB-only ESP8266 firmware: no Wi-Fi portal, no cloud
dependency, and no stored Wi-Fi password. The user wants changes tested on the
connected physical clock until the screen is visually confirmed working.

## Verified reference state (2026-07-17)

- Custom firmware is flashed and running on the physical clock.
- The flashed custom firmware provides six USB-selectable modes: an animated
  cyan LED Face, the usage dashboard, BTC/USD with 24 one-hour candles, a large
  Clock view, a Noto Sans Thai test view showing `สวัสดี`, and a local digital
  Gallery/slideshow.
- Current persisted display settings are title `WWW.PHUD.ME`, city `BANGKOK`,
  brightness `70`, rotation `0`, active-low backlight, and a teal accent.
- The user confirmed that the display works and the blinking is gone.
- The user visually confirmed the Noto Sans Thai test is readable and
  flicker-free.
- Daily reminder alerts are flashed and physically verified: they remain steady
  for about 60 seconds, return to Usage afterward, and do not flicker or show a
  blank frame.
- The user visually confirmed the BTC/USD price and 1H candlestick layout is
  readable, complete, and flicker-free. BTC is RAM-only data supplied by the Mac,
  while choosing the BTC screen persists across collector-triggered resets.
- The flicker fix is important: never redraw the whole panel every second.
  `usb_clock.cpp` redraws the time rectangle only when the minute changes and
  performs a full redraw only after configuration changes.
- Time and usage are RAM-only and must be synchronized again after power
  loss/reset. Opening the CH340 serial port also resets the ESP8266.
- The Gallery stores up to seven 240 × 240 RGB565 photos in the clock's local
  LittleFS partition. Gallery playback settings persist across USB resets so a
  collector update does not stop a running slideshow.
- The clock stores up to eight daily ASCII reminder labels in EEPROM. Reminder
  list/set/delete sessions restore cached time and usage after the USB reset.
- Mac-side cached usage includes the previous complete snapshot so dashboard
  trend arrows survive CH340 resets. An incomplete first-run Claude baseline is
  valid collector state until all three rotating slots have been populated.
- Gallery replacement uploads are staged in a temporary LittleFS file and must
  pass the FNV checksum before the existing slot is replaced.
- The post-audit firmware build was flashed with written-data hash verification;
  after cached state restoration, the user visually confirmed that the physical
  display remained normal.
- LED Face V1 is flashed with written-data hash verification and the firmware
  answers `STATUS` plus `FACE AUTO`. It adds cyan code-rendered eyes, eight
  automatic/manual mood choices, partial-region animation, and Face/mood
  persistence across USB resets. The user visually confirmed the physical Face
  screen looks good.
- V2-A is flashed and its persistent LaunchAgent is active on the Mac. The
  RAM-only `EMOTION` override and `tools/clock_service.py` passed USB protocol
  checks: one stable process owns the serial device, CLI requests use the
  user-only Unix socket without resetting the clock, cached usage/BTC survives,
  and `STATUS` reports Face Auto with `auto_emotion=focus`. The user visually
  confirmed the V2-A activity-driven transitions and animation work correctly.

## Confirmed hardware

- MCU: ESP8266EX, 26 MHz crystal.
- Flash: 4 MB, DIO, 40 MHz.
- USB bridge: CH340/CH341. Normal tools auto-detect its current
  `/dev/cu.usbserial-*` path after the clock moves between USB ports.
- Display: ST7789, 240 x 240, SPI mode 3.
- Display wiring:
  - SCLK: GPIO14
  - MOSI: GPIO13
  - DC: GPIO0
  - RST: GPIO2
  - CS: GPIO15
  - Backlight: GPIO5, PWM, active-low
- The physical screen and all pins above have been validated by the flashed
  custom firmware, not merely inferred from documentation.

## Recovery artifacts: keep private

Before first flashing a device, make a complete 4 MB stock-flash backup and
record its SHA-256. Store both outside the repository. Never commit, upload, or
attach this backup: stock firmware may contain saved Wi-Fi credentials.

Do not erase the flash or restore a stock image unless the user explicitly asks
or recovery is required. Do not reconnect restored stock firmware to a sensitive
network without first assessing its security.

## Important files

- `src/usb_clock.cpp`: custom USB-only firmware and screen renderer.
- `src/thai_greeting_bitmap.h`: compact Noto Sans Thai test bitmap for
  `สวัสดี`; generated from Noto Sans Thai Regular 2.000 under the SIL OFL 1.1.
- `tools/clockctl.py`: Mac USB configuration client.
- `tools/clock_gui.py`: local-only browser control panel for Face moods, screen
  switching, brightness, usage/BTC refresh, reminders, test pattern, gallery
  uploads, and slideshow.
- `tools/clock_service.py`: persistent Mac serial owner, local command/Gallery
  proxy, usage polling host, and V2-A Mac-idle emotion engine.
- `tools/crypto_market.py`: fetches public Coinbase Exchange BTC-USD 1H candles
  and compacts 24 normalized OHLC candles into a bounded USB command.
- `tools/usage_collector.py`: Mac-side polling collector; provider commands are
  private TOML configuration and it publishes complete six-value usage snapshots
  plus public BTC market data when available.
- `tools/claude_usage.py`: Claude provider adapter. Its Keychain mode uses the
  current Claude Code identity only in memory; it prints one percentage.
- `tools/codex_usage.py`: Codex provider adapter. It reads the local profile
  token only in memory and prints the primary weekly usage percentage.
- `tools/usage-collector.toml.example`: safe template for the six local,
  read-only provider adapters. The real `usage-collector.toml` is Git-ignored.
- `platformio.ini`: `clock_usb` build target and conservative upload settings.
- `USB_CLOCK.md`: user commands, build, flash, and restore instructions.
- `.pio/build/clock_usb/firmware.bin`: generated firmware; never treat it as
  source or commit it.
- `CLAUDE.md`: concise session hand-off; keep it aligned with this file.

The repository began as a clone of `giovi321/smalltv-mod` (WTFPL). The custom
USB firmware is intentionally isolated behind the `clock_usb` PlatformIO target;
do not accidentally build or flash the upstream `smalltv` target.

The worktree may be dirty. Treat all existing changes as user-owned unless the
current task clearly created them; never use reset/checkout to clean it.

## Build and validation

Run from the repository root:

```sh
uvx --from platformio platformio run -e clock_usb
git diff --check
uvx --from esptool esptool image-info .pio/build/clock_usb/firmware.bin
```

Expected properties:

- Target: ESP8266.
- Flash size: 4 MB.
- Flash mode: DIO.
- Image checksum: valid.
- Current firmware is about 360 KB; investigate unexpected large growth.

The PlatformIO ESP8266 packaging tools emit Python invalid-escape
`SyntaxWarning` messages during `elf2bin.py`; these are third-party tool warnings,
not firmware build failures. Do not confuse them with compiler warnings from
project code. Keep project code warning-free.

## Flashing rules

The serial link produced corruption/noise at 460800 while reading flash. Always
use 115200 unless a later hardware test proves another rate reliable:

```sh
uvx --from esptool esptool \
  --port "$(uv run --with pyserial tools/clockctl.py find-port)" --baud 115200 \
  write-flash 0x0 .pio/build/clock_usb/firmware.bin
```

Require all of the following before declaring a screen change complete:

1. Build succeeds and the image checksum/layout is valid.
2. esptool reports that written data hash verification passed.
3. The USB firmware answers `STATUS` or another expected command.
4. The user visually confirms orientation, colour, brightness, and smoothness.

Never infer display success from a green build or serial response alone.

## USB configuration

Typical commands:

```sh
uv run --with pyserial tools/clockctl.py status
uv run --with pyserial tools/clockctl.py set title "PHUD CLOCK"
uv run --with pyserial tools/clockctl.py set city "BANGKOK"
uv run --with pyserial tools/clockctl.py set accent "#36D6C4"
uv run --with pyserial tools/clockctl.py set brightness 70
uv run --with pyserial tools/clockctl.py sync-time
```

`clockctl.py set` sends `SET` and `SAVE` in the same session. Raw serial commands
remain staged until `SAVE` is sent.

This board's CH340 auto-reset wiring resets the ESP8266 whenever a new serial
session opens. V1 tools therefore restore state after opening. Once V2-A is
installed, normal CLI/GUI/collector traffic must use the service socket and must
not open a competing direct session. Consequences:

- A one-shot command begins after a device reset.
- The Mac client waits for boot and discards startup chatter.
- Run `sync-time` as the final one-shot command, because opening another serial
  session resets the RAM-only time again.
- For multiple commands or scripts, prefer one persistent session:

```sh
uv run --with pyserial tools/clockctl.py shell
```

## Editing guidance

- Keep the main screen dynamic and code-rendered; avoid a full-screen framebuffer
  on the memory-constrained ESP8266 unless measurements prove it is safe.
- Prefer partial dirty-region updates. Do not clear the full screen in the main
  loop or on every second tick.
- Validate all serial inputs and keep bounded fixed-size buffers.
- Never echo secrets over serial or render them on screen.
- Gallery uploads must stay USB-only. Convert images on the Mac to 240 × 240
  RGB565 and use the binary `GALLERY BEGIN`/data/`END` protocol implemented by
  `clock_gui.py`; do not add Wi-Fi, cloud storage, or a browser-accessible
  network listener.
- Respect Claude's provider rate limit: the collector rotates C1 → C2 → C3,
  retains cached Claude values, and refreshes Codex separately. Do not make the
  GUI force a live Claude request.
- Preserve recovery commands (`TEST`, `SET ROTATION`, `SET BLINVERT`, `SHOW`)
  so a bad visual configuration can be repaired without seeing the screen.
- Do not commit, push, or publish changes unless the user explicitly asks.

## Likely next improvements

- Extend the V2-A persistent Mac service with weather-driven moods, usage
  thresholds, reminder-as-eyes, and optional auto-return from data screens.
- Add richer layouts or additional USB-selectable screens while preserving the
  partial redraw rule.
- Add an explicit machine-readable protocol/version handshake if external
  automation grows beyond the current line command interface.

---
> Source: [r14084/swc-digital](https://github.com/r14084/swc-digital) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
