## esp32s3-dongle-deploy

> After building ESP32-S3 dongle firmware, deploy to USB when possible


# ESP32-S3 dongle: build and deploy

When you change code under `firmware/esp32s3-dongle/`, run a full build, then **deploy to hardware** if a serial port is available:

1. `idf.py build` (or already done).
2. Run `firmware/esp32s3-dongle/scripts/deploy.sh` (from repo root) after a successful build, or with `ESPPORT=/dev/cu.usbmodemXXX` if autodetect fails.
3. If `idf.py` is not in PATH, `source` the user’s `esp-idf/export.sh` first; the script will use `IDF_PATH` if set.

The script runs `idf.py build` then `idf.py flash`. Skip flashing only if the user is clearly working offline or says not to use USB.

For CI, GitHub Actions builds and uploads `minicore_dongle_fw` artifacts; that is not a USB deploy.

---
> Source: [Saint-Francis-Robotics-Team2367/minibots-2026](https://github.com/Saint-Francis-Robotics-Team2367/minibots-2026) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
