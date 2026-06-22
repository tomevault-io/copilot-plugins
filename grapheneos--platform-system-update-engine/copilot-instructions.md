## platform-system-update-engine

> * `m update_engine_nonetd` to build

## Building and Running

* `m update_engine_nonetd` to build
* run `scripts/reload_update_engine.sh` to stop currently running update_engine
  and run locally built update_engine
* If `scripts/reload_update_engine.sh` fails with .so errors, run
  `adb push $OUT/system/lib64/*.so /data` to push local .so to device.
* `python3 scripts/update_device.py <path to ota.zip> --no-postinstall` to
   install OTA, look for `onPayloadApplicationComplete(ErrorCode::kSuccess (0))`
   as an indicator for successful OTA install. If `--no-slot-switch` is passed
   to `update_device.py`, look for `onPayloadApplicationComplete(ErrorCode::kUpdatedButNotActive (52))`
   instead.
* On failed OTA install, use logcat `adb logcat -d -s update_engine` to inspect
  update_engine logs. logcat output tend to be long, so either pipe to a file or
  use `tail` command to check the last few lines.
* Cleanup any install OTA with `adb shell update_engine_client --reset_status`

---
> Source: [GrapheneOS/platform_system_update_engine](https://github.com/GrapheneOS/platform_system_update_engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-22 -->
