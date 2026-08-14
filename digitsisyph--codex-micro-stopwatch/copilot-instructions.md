## codex-micro-stopwatch

> - Target hardware is M5Stack StopWatch Dev Kit C152 only.

# Project boundaries

- Target hardware is M5Stack StopWatch Dev Kit C152 only.
- Treat Codex Micro compatibility as experimental and undocumented.
- Do not claim a build, flash, pairing, button action, or quota update is
  validated until it is observed on physical C152 hardware.
- Never infer an upload port. Resolve and report the exact `/dev/cu.*` target
  immediately before any flash operation.
- Do not flash without a known factory-firmware recovery path.
- Keep OpenAI/ChatGPT authentication and rate-limit reads on the Mac companion.
  Never place account tokens in firmware or BLE payloads.
- Keep the emulated vendor HID path and the private quota GATT path separate.
- Do not imply that this is official OpenAI, Work Louder, or M5Stack hardware.
- Preserve upstream MIT attribution and `NOTICE.md` when redistributing.
- The default target intentionally uses the Mac's selected microphone. The
  optional isolated `usb-mic` PlatformIO project is input-only USB Audio: never
  add a USB speaker/output endpoint. A short local-only chime may use the
  physical speaker while the host microphone stream is idle. Only build or
  flash this target after the user explicitly chooses it.

## Installation workflow for coding agents

- Read `README.md`, this file, and M5Stack's official StopWatch recovery guide
  before changing the machine or the device.
- Confirm the connected target is an M5Stack StopWatch Dev Kit C152. Default to
  `m5stack-stopwatch`; use `pio run -d usb-mic` only when explicitly
  selected.
- Begin with read-only checks. Explain any missing dependency before installing
  it, and use the narrowest standard installation method available on the Mac.
- Require macOS 14 or newer, Swift 5.10 or newer, and PlatformIO Core. Treat
  dependency downloads and system permission changes as user-visible actions.
- Build the `m5stack-stopwatch` PlatformIO environment before flashing.
- For the optional microphone install, build the isolated `usb-mic` project,
  explain that the local completion chime is skipped while the host is
  streaming microphone audio, and verify macOS audio enumeration separately
  from BLE behavior.
- Resolve the exact newly connected `/dev/cu.*` device immediately before an
  upload. Report it to the user and obtain confirmation before flashing.
- After flashing the default target, verify the serial marker
  `CODEX_MICRO_STOPWATCH_READY` on the same exact port with
  `python3 scripts/serial_probe.py <port> --seconds 30 --expect
  CODEX_MICRO_STOPWATCH_READY`. The USB-mic image has
  no normal USB serial console; verify its UAC enumeration, short capture, BLE,
  and HID/RPC separately instead.
- Require ChatGPT Input Monitoring on macOS. Preserve another physical Micro's
  pairing record, but disconnect or power it off: this port supports only one
  active Micro during installation, validation, and normal use.
- Build the quota companion from source. Use demo discovery to obtain this
  Mac's CoreBluetooth UUID, require that UUID for real writes, and store it only
  in local ignored output or the user's `~/Library` files.
- Never request an OpenAI API key for this project. Never write credentials,
  account identifiers, prompts, task text, device MAC addresses, CoreBluetooth
  UUIDs, usernames, home-directory paths, or captured audio into tracked files.
- Keep any microphone validation recording temporary and outside the repository.
- Do not claim success from a build alone. Report build, flash, pairing,
  controls, Agent status, and quota sync as separate validation layers.

---
> Source: [digitsisyph/codex-micro-stopwatch](https://github.com/digitsisyph/codex-micro-stopwatch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
