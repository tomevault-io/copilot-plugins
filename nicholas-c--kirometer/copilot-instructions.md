## kirometer

> Kirometer is an ESP32 desk display paired with KiroCrew over Bluetooth LE. It

# Kirometer project context

Kirometer is an ESP32 desk display paired with KiroCrew over Bluetooth LE. It
is a hard fork of Clawdmeter; retain the attribution and licensing constraints
in [`NOTICE`](NOTICE).

## Architecture

- `firmware/` is shared ESP32 firmware: LVGL UI, board HALs, BLE peripheral,
  HID keyboard, and the typed v3 protocol parser.
- `app/` is the only companion. The installed KiroCrew app runs its BLE writer
  inside KiroCrew, obtains its own scoped gateway token, and owns collection of
  approvals, chat activity, credits, configuration, and request responses.
- There is deliberately no standalone host client, Claude-era daemon, or root
  installer. Do not recreate one.

## Hardware and boards

Only `waveshare_amoled_216` is hardware-validated for Kirometer. It is the
480×480 ESP32-S3 AMOLED board with 8 MB PSRAM. Other board folders are inherited
and must remain buildable: keep shared firmware board-agnostic, use `BoardCaps`
and the HAL, and never add `#ifdef BOARD_*` to shared code.

C6 boards have no PSRAM. Their Splash path uses the direct renderer and their
serial screenshot command is disabled. Do not assume a framebuffer is available.

## Build and test

```bash
pio test -d firmware -e native
pio run -d firmware -e waveshare_amoled_216
pio run -d firmware -e waveshare_amoled_216 -t upload --upload-port /dev/cu.usbmodem1101
python3 app/backend/test_device_contract.py
python3 app/backend/test_cross_instance.py
npm --prefix app/ui run build
```

Call `resource_status` before a PlatformIO build, upload, full native suite, or
other heavy work. On a tight host, serialize target builds and prefer focused
tests. Use `/dev/cu.usbmodem1101` for this device unless the user explicitly
changes it.

Opening the board's USB CDC serial port normally resets it. Do not use the
standard screenshot command to claim a populated live screen when a reset would
make the evidence stale.

## Firmware constraints

- Board initialization owns pins, I2C, display, touch, and power. Shared code
  calls only the HAL.
- CO5300 rotation is CPU remapping; flush areas must be even-aligned.
- LVGL RGB565A8 assets are planar RGB565 followed by alpha bytes.
- UI geometry belongs in pure helpers when it needs host tests. Preserve truthful
  unknown states; never turn unavailable credits, titles, or approvals into data.
- Pairing/bond clearing and approval actions are explicit user-facing operations;
  do not trigger them for testing.

## App and protocol

The app sends v3 JSON frames no larger than 480 bytes over the Kirometer data
service. It subscribes before refresh requests, reconciles canonical config
before emitting pending proposals, and uses stable generations. Keep app changes
self-contained in `app/backend/app.py`: KiroCrew loads hook modules as fresh
instances, so shared lifecycle state must use `ctx.storage`.

Read [`app/README.md`](app/README.md) before changing gateway calls, permissions,
BLE transport, or frame semantics. App tokens must stay least-privilege; never
work around a missing endpoint by granting transcript access.

## Documentation

Keep `README.md`, this file, and `app/README.md` aligned with the current
firmware-plus-KiroCrew-app architecture. Preserve `docs/porting/`; do not commit
generated planning reports or local agent artefacts.

---
> Source: [nicholas-c/kirometer](https://github.com/nicholas-c/kirometer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-18 -->
