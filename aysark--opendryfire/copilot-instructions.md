## opendryfire

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
uv sync                                   # install (Python 3.12+, uv required)
uv run pytest src/tests -q                # full suite, 154 tests, no hardware needed
uv run pytest src/tests/test_detect.py -q                        # one file
uv run pytest src/tests/test_detect.py::test_finds_a_dot_and_locates_it -q   # one test
uv run pytest src/tests -q -k "chime or click"                   # by name
uv run dryfire --help                     # CLI smoke checks, same as CI
uv run dryfire plan
uv run dryfire targets
```

`pyproject.toml` sets `pythonpath = ["src"]` and `testpaths`, so bare `uv run pytest` works too.
There is no linter or formatter configured — don't invent a `ruff`/`black` step.

The importable package is `opendryfire`; the installed command is `dryfire`
(entry point `opendryfire.cli:main`). CI (`.github/workflows/tests.yml`) runs the
suite on macOS and Ubuntu.

## The central idea

**The microphone answers *when*; the camera answers *where*.** Every design
decision downstream follows from that split, and violating it is the most
common way to break this codebase subtly:

- Timing is `(click_sample - beep_sample) / 48000`. The start beep is played and
  recorded on the *same* full-duplex stream as the striker click, so input and
  output latency cancel exactly. **Video timestamps are never used for timing** —
  only for pairing within a ±150 ms window (`session.PAIR_WINDOW_S`).
- Scoring is a homography from image pixels to target-plane inches. Camera
  pipeline delay is irrelevant to it.
- A shot requires **both** signals. Flash + click = scored shot; click alone =
  slide rack or off-target miss; flash alone = rejected. That fusion is what
  makes detection tractable in a red-lit room, and why thresholds only need to
  be good rather than perfect.

## Module map

| Module | Owns |
|---|---|
| `detect.py` | Blob finding + shot assembly. `LaserDetector.push(frame, t)` returns a `Shot` on the frame where a pulse *ends*. `calibrate_scene` derives room thresholds. |
| `audio.py` | `ShotTimer` (beep-and-listen, one rep), `LiveListener` (continuous, untimed drills), `ClickDetector`, `find_beep_onset`, `calibrate_click_floor`. |
| `geometry.py` | `TargetFrame` — the pixels→inches homography, its persistence, its scoring mask, and tripod-drift signatures. |
| `targets.py` | Target specs in **full-scale inches**, `ScaledTarget` (scale + stand-off → simulated distance), `group_stats`, shape classification. |
| `session.py` | `Shot`/`Rep`/`Session` records, `pair_events` fusion, JSON persistence, `~/.dryfire` path helpers. |
| `plans.py` | TOML plan loading and strict validation. |
| `drills.py` | Rules that are the same whoever wrote the plan: `ParLadder`, `evaluate_rep`, `coaching_note`. |
| `feedback.py` | Outcome chimes, constrained so the click detector cannot hear them. |
| `devices.py` / `camera.py` | Camera discovery by *name*, Continuity Camera preflight, capture that reports what the driver actually granted. |
| `ui.py` | The calibration click window (with magnifier) and the live drill overlay. |
| `cli.py` | Every subcommand; the only place modules are wired together. |
| `vault.py` | Optional Markdown/Obsidian export, off by default. |

## Coordinate and scale chain

Three spaces, and applying scale twice is the bug to watch for:

1. **Image pixels** → `TargetFrame.to_target()` →
2. **Printed-target inches** — origin at the A-zone centre, +x right, **+y up**
   (the homography flips image y). This is what `ShotRecord.x/y` stores.
3. **Full-scale inches** — `ScaledTarget.score()` divides by scale internally,
   so pass it printed inches. `to_sim_inches()` converts a *distance* to the
   simulated range.

Zone geometry in `targets.py` is written once at full scale for this reason.

## Runtime state — `~/.dryfire/`

`config.json` (camera + vault prefs) · `detector.json` (from `autocal`) ·
`audio.json` (from `audiocal`, and its existence is the "is timing calibrated?"
flag) · `calibration/<profile>.json` (the `TargetFrame`) · `sessions/*.json` ·
`plan.toml` (user plan, overrides the bundled one) · `spike/`.

Workflow order is load-bearing and documented that way everywhere:
**calibrate → autocal → spike → audiocal → run**. `autocal` is far better
informed once a calibration exists, because it can mask to the target plane.

## Two run paths

`cli.cmd_run` branches into `_run_timed` and `_run_untimed`, and they differ
more than they look. Timed mode opens a fresh full-duplex recording per rep and
has no live listener, so the guard against a chime being heard as a click is a
**wait** for the chime to finish; untimed mode runs `LiveListener` plus an
OpenCV window, and the guard is `listener.mute_for(...)`. Anything touching rep
flow, sound, or fusion usually has to change in both.

## Invariants that are easy to break

- **Reference points must be coplanar with the scoring surface.** The default is
  the target's own A-zone corners. Markers on a backer board are off by ~0.3″ —
  a quarter of a 1/5-scale A-zone, invisible in the output. ArUco is only for
  genuinely flat setups.
- **Shot boundaries are decided by time, not distance.** `max_pulse_s` (350 ms)
  closes a shot; `link_radius_px` only picks *which* blob to follow when several
  are visible. A dot that jumps mid-pulse is drift, not a second shot.
- **Red dominance is measured over the dilated halo, not the peak pixel.** A
  close dot saturates its core to white.
- **Redness has exactly one definition: the mean over a blob's dilated halo,
  as computed by `find_blobs`.** Anything that needs a redness number must get
  it from there — `calibrate_scene` derives the margin by running a probe
  detector with the margin disabled. Deriving it separately from per-pixel
  values is what shipped a margin no laser could clear (DESIGN.md §10a).
- **Detection margin is capped at `MAX_USABLE_MARGIN = 55`, in halo-mean
  units.** Above that the laser itself stops registering — silent blindness is
  worse than visible false positives, so the cap binds and the report says so.
- **A calibration must be shown to still see a laser, not merely to produce no
  false positives.** A blind detector passes a false-positive count perfectly.
  `sees_reference_dot` stamps a modelled dot into a real frame and runs the
  real `find_blobs`; `describe_calibration` reports `BLIND` and never "Clean"
  when it fails.
- **`push` returns a Shot when the pulse *ends***, several frames after the dot
  went dark — the frame you are holding then contains no laser. Anything that
  needs the pixels of the break (a crop, an overlay, a snapshot) must act when
  `LaserDetector.open_shot` becomes non-None. `spike` cropping the closing
  frame is how it saved eight blank crops and made a real bug unfalsifiable.
- **Chime fundamentals stay ≤ `feedback.MAX_SAFE_FREQ` (700 Hz) with a 12 ms
  raised-cosine attack.** A sharp ping *is* a striker click acoustically.
  `test_chime_is_not_heard_as_a_click` pins this; don't raise the ceiling
  without re-running it.
- **Cameras are addressed by name, not index.** A Continuity Camera enters and
  leaves the index list, so a saved index silently scores through the wrong
  lens' homography.

## Rules from CONTRIBUTING.md that shape review

**Measure, don't guess.** Every threshold came from a measurement recorded in
`docs/DESIGN.md`, including the approaches that were tried and killed. Read the
relevant section before changing a constant, and say what you measured.

**Fail loudly, never plausibly.** A wrong number that looks reasonable corrupts a
training log for months. `find_beep_onset` returns `None` rather than guessing;
`build_homography` reprojects its own points and rejects a bad fit because
`cv2.findHomography` does not reliably return `None` for degenerate input;
calibration scale is checked against the block's scale before a session starts;
camera property writes are read back and reported as `IGNORED`. A change that
makes something *silently* degrade is the wrong shape.

**Safety framing is not editorial.** This scores dry fire; it does not make dry
fire safe. Nothing here verifies a firearm is unloaded and no feature should
imply it does.

## Tests

Synthetic frames and synthesised audio only — no camera, mic, or cartridge. They
deliberately model the awkward cases (blown-out dot core with red only in the
halo, dropped frame mid-pulse, cream target on warm red-dominant wood, striker
click as a broadband burst over room tone). Name tests for the behaviour:
`test_a_dot_that_jumps_far_mid_pulse_is_still_one_shot`, not `test_push_2`.

`test_integration.py` holds the round-trips — a rendered dot at a known target
position scored back into inches — and is the only thing that catches the class
of bug where every unit passes and a low-left group is still reported as
high-right. Add to it when changing anything in the pixels→inches→score path.

---
> Source: [aysark/opendryfire](https://github.com/aysark/opendryfire) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-28 -->
