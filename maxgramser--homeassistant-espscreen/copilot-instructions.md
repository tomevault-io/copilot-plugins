## homeassistant-espscreen

> The preferred route for new users is docs/EASY_SETUP.md: ESP Screen Manager

## Easy Setup and releases

The preferred route for new users is docs/EASY_SETUP.md: ESP Screen Manager
plus remote ESPHome packages. No token or blueprint needed. Tiles live in the
persistent add-on data; Wi-Fi/API/OTA stay in the device's own ESPHome YAML. Read
docs/RELEASING.md before publishing updates. Main distributes both boards.
Every push to GitHub is a release: always also bump the add-on version in
screen_manager/config.yaml (with a CHANGELOG line), otherwise HA won't see an update.
Generate packages with tools/generate_packages.py; don't edit them by hand.
A screen gets its tiles from the add-on while it runs: every card works in all
twenty positions, and no Home Assistant entity belongs in a board profile.
Preserve the data schema, protocol compatibility, unique keys, and CYD preferences.
Test updates against existing data. Don't publish an unknown storage version without
a migration. Production Ingress needs no long-lived token or public port.

## Guition board

The Guition 4848S040 has a separate profile `guition-4848s040.yaml` with
480×480, ST7701S RGB, and GT911. Read docs/GUITION.md. Don't carry over the CYD
layout or the XPT2046 calibration.
Verify touch with tools/verify_gt911.py and keep the CYD regressions green.
Don't configure wallbox relays as part of display support.

# Working instructions for LLMs and developers

This project drives an ESP32-2432S028 with ILI9341 + XPT2046 (320×240,
LVGL 90°). Read README.md and docs/ before installing. The owner can
physically tap; an agent cannot replace that with software coordinates.

## Installing a new screen

1. Install it from ESP Screens (docs/EASY_SETUP.md): the add-on writes the screen's
   own ESPHome YAML with its name, Wi-Fi reference and unique keys, and flashes it
   over USB. Identify the USB port and board variant first, and check whether a
   profile for this screen already exists; never overwrite a working one.
   Use ESPHome 2026.6.2 with Python 3.11-3.14. Don't show keys in logs or chat.
2. CYD: the screen shows its calibration on first boot; the owner taps the
   crosshairs. `tools/calibrate.py` with docs/CALIBRATING.md is the USB route for a
   panel that needs measuring; ask for physical taps per target, and wait for
   confirmation that the measurement screen is visible. A successful build is not a flash.
   Guition: GT911 reports pixels; verify with tools/verify_gt911.py, no ADC calibration.
3. Pair with the owner's own Home Assistant through the ESPHome integration.
   Read real entity IDs and supported attributes; don't make up entities.
4. Choose the tiles in ESP Screens. Don't test real device actions without the
   owner's permission, and report which checks were actually carried out,
   the limitations, and how long the screen stayed up.

## Code and regressions

- Keep base hardware and UI in `home-like-2432s028.yaml`; personal data belongs
  in the gitignored local profiles.
- Screen settings live in one table: `components/smart_display/settings_screen.h` draws the
  page on the screen, `SETTING_RULES` in the add-on validates the same keys. Firmware 0.2.49+
  owns them: every writer (the page, the entities in both profiles) goes through
  `settings_screen::set()`, and the add-on changes them through the entities in
  `SETTING_ENTITIES`, never through the layout message. docs/SETTINGS.md is the end-to-end
  recipe for adding one; never widen the eleven-key `settings` block older firmware insists on.
- Preserve fixed pages, hidden navigation at six tiles or fewer, and a
  minimum default standby of 600 seconds. Don't reintroduce free scrolling
  without physically testing for touch/navigation regressions.
- Preserve the touch filter and event guard before actions; calibration processes
  filtered physical ADC values. Don't apply the affine correction twice, in both the driver and the UI.
- The calibration wizard assumes swap_xy=false, mirror_x=true, mirror_y=false,
  and LVGL 90°. A changed orientation also requires a new projection/tests.
- Run the Python tests, both C++ test suites, and ESPHome validation/build on code changes.
  Firmware tests and hardware acceptance are different checks.
- `diagnostics/run_ui_test.py` renders without HA actions; don't touch the screen
  during that test. Use `--name` for the expected device identity.
  `diagnostics/send_layout.py` pushes a demo layout with every card type to a
  screen via the API inbox (no HA actions); the manager restores the real
  layout within ~25 s. Guition: `capture_ui.py` saves the LVGL render as a PNG.
  Without a screen: `tools/render_topbar.py` renders the real top-bar code for both
  boards via the ESPHome host + SDL2 to `.esphome/render-topbar/out/sheet.png`.
- Share through Git. Don't stage secrets, measurements, binaries, logs, build
  caches, or local device profiles.
- No automatic firmware upload to an arbitrary connected port.
  With multiple boards, first determine the intended port.
- Profiles with the same `DEVICE_NAME` share `.esphome/build/<name>`.
  Never compile or upload them concurrently; check
  the `firmware.bin` path in the upload log, and then the compile time via
  `device_info`. A wrong profile knocks the screen out of ESP Screen Manager.

Historical diagnostics are background context, not proof that a new panel
works correctly. During onboarding, don't make claims about physical tests that weren't actually performed.

---
> Source: [MaxGramser/homeassistant_espscreen](https://github.com/MaxGramser/homeassistant_espscreen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
