## dji-4g-module-mac

> This repository documents one narrow task: converting a verified DJI first-generation 4G module from USB identity `2ca3:4006` to `2c7c:0125` directly on macOS.

# Agent Notes

## Purpose

This repository documents one narrow task: converting a verified DJI first-generation 4G module from USB identity `2ca3:4006` to `2c7c:0125` directly on macOS.

Do not expand the repository into a modem-management, NAS, SMS, calling, eSIM, VoWiFi, Docker, virtual-machine, or proxy project.

## Safety Rules

- `convert` and `restore` are persistent writes to hardware. Run either action only after the user explicitly asks for that exact operation in the current conversation.
- Before a write, run `inspect` and `query`. Stop if the device ID or the complete USB configuration differs from the README.
- Never use a destructive action merely to test the script. The conversion path was verified on a real device; the restore path is intentionally marked as unverified.
- Keep the two confirmation strings unchanged. They are part of the accidental-write guard.
- If the device disappears after reboot, first wait and rerun `inspect`; do not repeat a write command blindly.

## Public Repository Boundary

- Keep this repository source-only: README, the Python script, license, and small tests or documentation are allowed.
- Never add closed-source binaries, firmware, ZIP/TAR files, PDFs, web downloads, credentials, private logs, phone numbers, NAS details, SIM information, or third-party screenshots.
- Do not copy prose from other repositories. Cite the original source as a link and write new documentation.
- Do not use DJI logos or suggest official support, endorsement, compatibility guarantees, or a way to bypass accounts, paid services, carrier controls, flight restrictions, or safety features.

## Verification Before Publishing

Run:

```bash
uv tool run ruff check .
uv tool run ruff format --check .
uv run dji_modem_usb.py inspect
uv run dji_modem_usb.py query
```

On a device that is already converted, verify that both `convert` and `restore` refuse to write without their exact confirmation strings. Do not run either confirmed write as a routine test.

---
> Source: [keru-s/dji-4g-module-mac](https://github.com/keru-s/dji-4g-module-mac) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-01 -->
