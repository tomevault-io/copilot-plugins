## piclock

> _This document is LLM-generated._

# AGENTS.md — PiClock

_This document is LLM-generated._

Guidance for AI agents (and humans) working in this repo. Read this before making changes.

## What this is

An always-on digital bedroom clock built from a **Raspberry Pi Zero W** (or **Zero 2 W**)
driving an **Adafruit 1.2" 4-Digit 7-Segment Display with I2C Backpack** (HT16K33). It lives in a
3D-printed housing. Design goals, in priority order:

1. **Never needs to be reset.** It boots, finds the network, and shows the correct local
   time with no human interaction — ever.
2. **Always correct across DST.** Spring-forward / fall-back transitions happen
   automatically with no code change and no manual adjustment.
3. **Survives power loss.** Unplug it, plug it back in; it recovers on its own.
4. Quiet and legible in a dark bedroom (sensible default brightness, non-blinking colon).

## How "never reset / always-correct DST" actually works

The Pi has **no battery-backed RTC**. We do not add one. Correctness comes from:

- **NTP over Wi-Fi** keeps the system clock accurate (`systemd-timesyncd`, on by default
  in Raspberry Pi OS). No network at boot → time is wrong until Wi-Fi associates, then it
  self-corrects. The clock software should tolerate the brief "time not yet synced" window
  rather than displaying a bogus time (see Software design).
- **The IANA tz database** handles DST. We set the timezone **once** on the Pi
  (`sudo timedatectl set-timezone America/New_York` or wherever it lives) and read **local
  time** in Python. `zoneinfo` applies DST rules automatically; OS package updates keep the
  tz rules current. **Never hardcode a UTC offset.** Never compute DST ourselves.

So: NTP for the instant, tzdata for the offset. That combination is the whole reliability
story — keep it intact.

## Hardware

- **Board:** Raspberry Pi Zero W or Zero 2 W, running **Raspberry Pi OS Lite** (headless, no
  desktop). One code path serves both; see the architecture note under Software design re: the
  arch-selected zeroconf pin. The Zero W is armv6 (32-bit OS only); the Zero 2 W is
  armv7l/aarch64 (32- or 64-bit OS).
- **Display:** Adafruit 1.2" 4-digit 7-segment + I2C backpack, HT16K33 controller.
  - I2C default address **`0x70`** (solderable to 0x71–0x77 if changed — check the board).
  - We drive the HT16K33 **directly over I2C with `smbus2`** — no Blinka/Adafruit stack. The
    segment font and the digit/colon RAM offsets live in [piclock/display.py](piclock/display.py),
    which is the canonical reference; that mapping is the thing to get right.
- **Bus:** I2C is enabled on the Pi (`sudo raspi-config nonint do_i2c 0`); it exposes
  `/dev/i2c-1`. Verify the display address with `i2cdetect -y 1` (expect `0x70`).
- Detailed wiring/build instructions live in [HARDWARE_RUNBOOK.md](HARDWARE_RUNBOOK.md).

## Software design

Default to 24-hour time. The code is split so the logic is testable off-Pi and the only
hardware-touching part is isolated:

- [piclock/clock.py](piclock/clock.py) — **pure** time → `DisplayState` (4-char text + colon). No
  I/O. Shows `----` when the clock isn't synced. The colon is **steady by default** (this is
  a bedside clock — no movement at night); an optional 1 Hz blink is config-gated and derived
  from the wall clock, not a tick counter, so it can't drift.
- [piclock/display.py](piclock/display.py) — the HT16K33 `smbus2` driver, plus a pure
  `build_buffer()` (font/colon → 16-byte RAM) and a `NullDisplay` for off-Pi runs. The
  driver skips redundant I2C writes when nothing changed. The hardware is wrapped in
  `ResilientDisplay`: if the display is absent or a connector is bumped, `show()` never
  raises — it logs once and retries the connection every ~10 s, so the service keeps running
  (no crash-loop) and the correct time appears the instant the display reconnects.
- [piclock/timesync.py](piclock/timesync.py) — `is_ntp_synced()` via `timedatectl show -p
  NTPSynchronized`. Any error → treated as not-synced (so we show dashes, never a bad time).
- [piclock/config.py](piclock/config.py) — `Config` dataclass with `PICLOCK_*` env overrides
  (brightness, schedule on/off + times + wake/night brightness, 24h, blink, I2C bus/address,
  tick, and `PICLOCK_DISPLAY=null` for dev). Validates its inputs at construction, including
  that the schedule edges are in daily order.
- [piclock/schedule.py](piclock/schedule.py) — **pure** day/night schedule logic: wall-clock time →
  phase (`off` / `wake` / `night`). No I/O, unit-tested like `clock.py`.
- [piclock/ctl.py](piclock/ctl.py) — optional CLI control: a Unix **datagram** socket
  (`PICLOCK_CTL_SOCKET`; the unit sets `/run/piclock/ctl.sock` via `RuntimeDirectory=`)
  receives one-line commands from [scripts/piclockctl](scripts/piclockctl) (`on`, `off`,
  `brightness 0-15`; the client's `bright`/`dim` are aliases for levels 15/0). A pure
  `parse()` validates them; a daemon thread queues them for the loop, which applies them to
  `DisplayControl` and mirrors them to HomeKit — same override semantics as Siri. The socket
  is mode 0660, group `clockctl` (install.sh creates the group and adds `clocker` to it so
  the app can chgrp its own socket; humans join the group to use `piclockctl`). Startup is
  fail-soft like HomeKit.
- [piclock/app.py](piclock/app.py) — the loop, wiring the above together; SIGTERM/SIGINT blank the
  display and exit cleanly. When the schedule is enabled (`PICLOCK_SCHEDULE=on`) the loop
  evaluates `schedule.phase()` each tick and applies each phase change via `schedule.edge()`
  — that edge-triggering is what lets a HomeKit override stick until the schedule's next
  on/off edge (the dim edge only adjusts brightness, so an evening "off" stays off
  overnight), and since the first tick counts as an edge, a reboot or deploy mid-window
  comes up in the correct state with no external reconciliation.
- [piclock/control.py](piclock/control.py) + [piclock/homekit.py](piclock/homekit.py) — display on/off +
  brightness control. `DisplayControl` is a tiny lock-guarded object the loop reads each tick
  (blank when "off"); it's **always present**. It is driven by the optional in-app day/night
  schedule (default edges: 08:00 blanks, 17:00 shows at the wake brightness — default full —
  and 20:00 dims to the night brightness — default the dimmest lit level; times tunable via
  `PICLOCK_OFF_AT` / `PICLOCK_ON_AT` / `PICLOCK_DIM_AT`, levels via
  `PICLOCK_WAKE_BRIGHTNESS` / `PICLOCK_NIGHT_BRIGHTNESS`). The schedule is switched on by a
  systemd drop-in ([systemd/piclock-schedule.conf](systemd/piclock-schedule.conf), installed by
  `install.sh --schedule`) — there are no schedule timers or helper units. Also
  **optionally**, by HomeKit. HomeKit
  (`PICLOCK_HOMEKIT=on`) runs a HAP-python Lightbulb accessory in a daemon thread that mutates
  the same `DisplayControl`, and schedule-driven changes are pushed back to the Home app via
  `HomeKit.notify_state` so the tile stays in sync (scheduled changes also force brightness to
  the wake level at the on edge and the night level at the dim edge). `pyhap` is imported lazily so the core runs without the extra deps
  (separate `requirements-homekit.txt`; the zeroconf pin is selected by CPU architecture via
  PEP 508 markers — 0.39.4 on armv6/Zero W, last pure-Python release with an armv6 wheel; the
  current release on Zero 2 W and up — so pip picks the right one with no install flag). Both
  features are opt-in via `scripts/install.sh` flags. See
  SOFTWARE_RUNBOOK.md §4–5.

Guarantees worth preserving: the display layer must **fail soft** on I2C errors (a missing or
flaky display logs + reconnects, never crashes the service); the clock must keep showing
`----` until `is_ntp_synced()` is true; and **HomeKit and control-socket startup are
non-fatal** — if either can't start (missing deps, unwritable state dir, missing socket
dir, …) the app logs and runs the clock without it, since the 7-seg's job must never depend
on an optional feature. `Config` validates its inputs
at construction (positive tick, known backend, 0-15 brightness levels, ordered schedule
edges) so misconfiguration fails fast instead of busy-looping or silently running the
wrong backend.

Tests live in [tests/](tests/) and cover the pure parts (`render`, `build_buffer`) with no
hardware. Run with `pytest`.

## Running on the Pi

Managed by **systemd** (unit at [systemd/piclock.service](systemd/piclock.service)) so it
starts at boot and restarts on failure.

- The service runs as a dedicated **`clocker`** system account — no login shell, **no sudo**,
  member of the `i2c` group only (its sole privilege, for `/dev/i2c-1`). The app needs no
  elevated privileges: `timedatectl show` is readable by any user, and I2C access comes from
  the group. If something ever *does* need privilege, add a narrow allowlisted entry in
  `sudoers` for `clocker` — never grant it blanket sudo.
- Deploy target is **`/opt/piclock`**, owned by `clocker` (kept out of `/home` so the service
  account can read it). [scripts/install.sh](scripts/install.sh) (run with `sudo`) creates
  the account, rsyncs the code there, builds the venv as `clocker`, and installs + enables the
  unit. Idempotent; re-run after pulling code.
- `sudo systemctl {start,stop,restart,status} piclock` · `journalctl -u piclock -f`

## Dev environment

- Runtime deps are pinned in [requirements.txt](requirements.txt) — just `smbus2`. Dev/test
  deps in [requirements-dev.txt](requirements-dev.txt) (`pytest`). Install them however you
  prefer (a virtualenv is the usual choice, but that's your call).
- Logic runs on any dev machine (macOS/Linux); use `PICLOCK_DISPLAY=null` to run the loop
  without hardware. Treat the Pi as the source of truth for anything touching I2C. A reference
  Pi is Debian 13 / Python 3.13; the architecture is armv6 on a Zero W, armv7l/aarch64 on a
  Zero 2 W. On 32-bit Raspberry Pi OS `pip` resolves prebuilt wheels via piwheels (configured in
  `/etc/pip.conf` on the stock image); a 64-bit image resolves manylinux wheels from PyPI.

## Deployment / updating

There is no built-in remote-deploy pipeline — deployment is up to you. The supported path is
to get the code onto the Pi (clone, `git pull`, `scp`, etc.) and run
[scripts/install.sh](scripts/install.sh) with `sudo`. It is idempotent: re-running it after
pulling new code re-syncs `/opt/piclock`, refreshes the `clocker` venv, and restarts the
service. If you want CI push-to-deploy, an adaptable Gitea Actions template (tag-triggered,
least-privilege: a forced-command receiver plus a single allowed sudo apply step) lives in
[deploy/](deploy/) — see [deploy/README.md](deploy/README.md). Whatever you build, have it run
as an unprivileged account — never give a deploy key blanket root.

## Conventions & guardrails

- **Don't reinvent time/DST.** Use `zoneinfo` + system NTP. No manual offset math, no DST
  tables, no scraping time from the internet in app code.
- **Don't commit secrets** (SSH keys, tokens) or local build artifacts (virtualenvs, caches).
- Keep new runtime deps out of `requirements.txt` unless genuinely needed — boot time and
  SD-card space on a Zero W (the most constrained target) are scarce.
- Match the existing minimal style; this is a small appliance, favor clarity over
  abstraction.
- When changing display code, state how you verified it (real hardware vs. logic-only), since
  most of it can't be exercised off-Pi.

---
> Source: [code-a-saurus/PiClock](https://github.com/code-a-saurus/PiClock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-23 -->
