## project-marvin

> Hand-tracking + projector interface: flick a model out of the monitor, carry it

# Working in this repo

Hand-tracking + projector interface: flick a model out of the monitor, carry it
across a projected surface by hand, drop it on a Bambu Lab X2D to slice and print.
See `README.md` for setup and usage.

## Layout

```
gesture/   reusable gesture stack — keep projector and printer concepts OUT
app/       interaction logic: carry state machine, drawing, mesh rendering, sound
ui/        display enumeration, camera preview
printer/   slice and send: Bambu Studio CLI, FTPS upload, MQTT control
tools/     entry points: flick_test, calibrate, wall, send_to_printer
tests/     synthetic — no camera, projector or printer required
```

`app/carry.py` is pure logic: a hand position in wall coordinates plus a pinch state
in, everything including physics out. Keep it that way — it is what makes the paths
that must *not* start a print testable without hardware.

## Invariants

Break any of these and something either lies to the user or prints when it shouldn't.
Each already has a test; add one alongside if you extend the area.

- **Only a released object arms the printer.** Carrying a pinched object over the
  anchor does nothing. A landing that overlaps arms a cancellable countdown.
- **Drawing never changes reach.** `object_scale`, `drawn_position()` and
  `drawn_size()` are cosmetic; collision, grab distance and the printer's catch area
  stay tied to `object_radius` and `position`.
- **Printer collision is swept, not a point test.** A thrown object covers enough
  ground per frame to skip the zone otherwise.
- **Never present a number the printer did not send.** Absent percent means an
  indeterminate ring, not 0%. No report for `STALE_S` means draw nothing. An
  unrecognised stage renders as its number, never a guessed name.
- **The preview crop is display-only.** MediaPipe always receives the full frame;
  cropping its input would offset every landmark and invalidate `calibration.json`.
- **Scenery cannot start a print.** There is exactly one anchor that can.

## Frame rate is a correctness constraint

Every detector measures a rate over a time window, so below ~25fps they run out of
*samples* and go dead rather than degrading — a flat-zero readout looks identical to
a broken detector. The camera is usually the bottleneck (`cap.read()` dominates;
MediaPipe is ~10ms). Anything CPU-bound must pace itself off the render loop, and
`--debug` splits `cam` from `work` because the two causes need opposite fixes.

## Environment gotchas

- **mediapipe 0.10.35 on Python 3.14 has dropped `mp.solutions` entirely.** Only the
  Tasks API exists, which needs `hand_landmarker.task` on disk. Tutorials using
  `mp.solutions` do not apply.
- **Windows DPI:** call `ui.displays.set_dpi_aware` before reading display sizes or
  Windows reports scaled logical pixels and skews the whole mapping.

## Slicer gotchas

- **Bambu Studio never prints to stdout on Windows** — GUI-subsystem binary. Don't
  parse its output; read `result.json` in the working directory instead.
- **Never pass `--outputdir` with `--export-3mf`.** Set the working directory. There
  is a test asserting the flag stays out of the command.
- **Always verify the 3mf contains `Metadata/plate_1.gcode`** plus its `.md5` before
  uploading. A project 3mf and a sliced one are indistinguishable from outside.

## LAN protocol gotchas

All established empirically against a real X2D; none are documented upstream.

- **FTPS needs TLS session reuse.** Bambu's vsFTPd rejects data connections that
  negotiate a fresh session (`522 ... session reuse required`). Stock `ftplib` never
  reuses, so `ntransfercmd` is overridden. Separate from the implicit-TLS wrapping.
- **The print URL is `brtc://usb/<file>`.** Not the X1/P1 `file:///sdcard/` family.
  FTPS on 990 is chrooted to the removable stick, so that is where uploads land.
- **`reason: "ERROR STATE"` means the command was never parsed.** A standing
  `print_error` makes the printer refuse everything unread until cleared on its
  screen, so payload fixes appear to do nothing. Read the MQTT `reason` field, never
  the printer's screen text — they disagree.
- **paho-mqtt 2.x passes a `ReasonCode`, not an int.** `int(reason_code)` raises
  inside paho's own thread. Use `reason_code.is_failure`.
- **`553 Could not create file` on every path means no storage inserted**, not a
  permissions problem.

## Testing

```bash
python -m pytest
```

Everything is synthetic — no hardware. Tests are weighted toward what must *not*
happen: what must not print, what must not be drawn, what must not be claimed about
the printer's state. Keep that weighting.

---
> Source: [almostinventor/project-marvin](https://github.com/almostinventor/project-marvin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
