## dota-bootbreaker-auto

> Python bot that auto-plays the Dota 2 "Bootbreaker" arcade minigame (a

# Bootbreaker Autoplayer

Python bot that auto-plays the Dota 2 "Bootbreaker" arcade minigame (a
Breakout/brick-breaker variant: bounce a flying "boot" ball with a cart/paddle).
**Primary goal: keep the ball alive as long as possible.** Survival = winning;
no other objectives matter.

## Platform & tooling constraints (hard requirements from the user)

- **Windows only.**
- **Use `uv` for everything.** Deps live in `pyproject.toml`.
- **Do NOT pin versions** — always use latest.
- **Keep it lean** — only add a dependency if actually needed.
- Run: `uv run python -m bootbreaker` (add `--debug` or `--recalibrate`).
- Tests: `uv run pytest` (currently ~43 tests, all passing).

## Scope: fully autonomous (auto-throw + tracking)

The bot both **launches/aims the ball and tracks it.** Three states (`bot.py`):
- **AIM** — the on-screen pre-throw prompt (the ␣ key icon) is visible and no
  ball is in play -> returns `"launch"`. Main's `_launch()` locks the cart (tap
  Space), then **steers the aim to vertical with A/D in a closed loop** and
  throws (tap Space). The aim is NOT an auto-sweep we wait on — it's driven by
  A/D, so `_launch` pulses A/D (`_AIM_STEP_HOLD=0.05s` per pulse), re-reads
  `detect_aim_angle`, and stops when within `_AIM_TOLERANCE=5°` of straight-up
  (deadband can't be 0 or it oscillates forever) or after `_AIM_MAX_ADJUST=40`
  pulses. We don't know which of A/D rotates which way, so the loop **self-
  corrects**: if a pulse doesn't reduce the tilt, it flips the key. `_launch`
  logs the last aim samples for tuning. **The prompt is the ONLY launch trigger.**
- **PLAYING** — a ball is in play -> track and steer the cart to intercept.
- **WAIT** — no ball and no prompt (a brief transition): hold (steer to the last
  target for `chase_gap` frames first, in case of a blind-spot blip).

Launch used to fire from a ball-absence heuristic (miss counting), which threw
at random moments. It's now driven by `detect_prethrow` (template-matching the
grey ␣ key icon), so it only throws when the game is actually asking for it.
If the near-vertical aim reading proves unreliable, the fallback is to simplify
`_launch` to lock+throw immediately (tracking handles angled launches).

## Controls (in-game)

- `A` = move cart left, `D` = move cart right (bot presses these).
- `Space` = lock cart + throw — **pressed by the bot** in `_launch()`.
- `F8` = global hotkey to start/pause the bot. **Starts PAUSED.** User opens
  Dota, presses F8, which triggers play-region auto-detection on first run.

## Dependencies

Runtime: `mss` (screen capture), `opencv-python` + `numpy` (HSV color
detection), `pydirectinput` (scancode key output — works in games where normal
key injection doesn't), `keyboard` (global F8 hotkey). Dev: `pytest`.
Build: `hatchling`, `packages = ["bootbreaker"]`.

## Config (per-machine, git-ignored)

- `config.json` holds the play region `{left, top, width, height}`. It is
  **git-ignored** — different on every machine (different resolution/DPI).
- `config.example.json` is the committed template.
- `bootbreaker/config.py` loads/saves `config.json` (`DEFAULT_CONFIG_PATH`).
- First run (or `--recalibrate`) auto-detects the region via
  `detect.detect_play_region` and saves it.

## Architecture / files

- `bootbreaker/main.py` — CLI (`--debug`, `--recalibrate`), F8 pause toggle,
  and the capture→decide→act loop. Each frame: `bot.step()` returns
  "launch"/"left"/"right"/"hold"; `"launch"` runs `_launch()` (lock+aim+throw)
  then `bot.prime_playing()`, otherwise the loop holds `A`/`D` or releases keys.
  `--debug` shows a live always-on-top overlay window (ball=cyan, cart=red line
  + deadzone band, target=green line, HUD with state/action/coords) refreshed
  every 3rd frame, and prints an fps readout. During `_launch` (debug) the same
  window shows the **aim overlay**: detected aim dots (yellow), the fitted aim
  line (magenta), the search band (grey), and a true-vertical reference at the
  cart (green) — magenta not parallel to green means the throw isn't straight up.
- `bootbreaker/capture.py` — `grab(region, sct)`, `grab_fullscreen`,
  `calibrate(config_path, grabber)` (auto-detects + saves; raises if not found).
- `bootbreaker/detect.py` — **the most-iterated, most-fragile file.** HSV
  color detection of region, ball, cart, aim angle. See notes below.
- `bootbreaker/strategy.py` — pure functions: `decide_move(target_x, cart_x,
  deadzone)` → "left"/"right"/None (bang-bang w/ deadzone);
  `predict_intercept_x(ball_x, ball_y, vx, vy, paddle_y, width)` reflects the
  ball's path off side walls (triangle wave) to predict the landing x;
  `_reflect`.
- `bootbreaker/bot.py` — `Bot`, the AIM/PLAYING state machine + tracker. See
  notes below.
- `bootbreaker/input.py` — `Controller(backend=pydirectinput)`:
  `hold(key)` / `release_all()` / `tap_space()`, tracks held keys. `tap_space`
  holds Space down for `_TAP_HOLD=0.06s` (down→sleep→up), NOT `press()` — see
  the PAUSE/tap gotcha below.
- Tests in `tests/`; screenshots the detection is tuned against in `docs/`.

## Detection notes (`detect.py`) — hard-won, don't regress

- **Ball = flying boot.** It has a **cyan glow ring** + a **brown boot body**.
  `detect_ball` finds cyan blobs (HSV `_CYAN_LO`–`(110,255,255)`,
  then `MORPH_CLOSE` with a 15×15 ellipse to merge the *broken* glow ring into
  one blob — CLOSE not OPEN) and keeps the blob with the most **brown boot
  pixels adjacent** (`_MIN_BOOT_BROWN=400` in a `_BOOT_PAD=30`px window).
  - **Returns the centroid of the brown boot body, not the cyan glow** (the glow
    sits to one side, so its centroid is ~20-30px off; the target must be the
    boot itself).
  - **Rejects steel-blue bricks by saturation+value, NOT brown-adjacency.** A
    blue brick beside a *brown brick* satisfies brown-adjacency (`brown_near`
    ~1178). The bricks are a flat dull cyan (S=110, V=162) vs the glow (S~214,
    V>=180), so `_CYAN_LO=(80,130,170)` rejects them. Verified on
    full-game-with-ui.png (tmp/tune_cyan.py, tmp/probe_hue.py). The cart's teal
    glass is handled by the cart blank (below).
  - `_MIN_BALL_AREA=60`, `_MAX_BALL_WIDTH=75` (wider cyan = a brick).
  - Do NOT add an aspect-ratio filter — motion blur makes the glow wide
    (frame with w59×h21 got wrongly rejected). Only the width cap is safe.
  - Cart-aware blanking: blanks a **tight** box around the cart only (its teal
    glass), not the whole bottom strip, so a low/side ball stays visible. Size
    matters — a wide/tall blank swallowed the ball on tight-angle descents onto
    the cart (it vanished in the blank's shadow, re-emerged too low to catch).
    Measured (`tmp/probe_cart_blank.py`): glass only false-triggers at
    half-width < ~70 and never above the cart centroid, so `_CART_BLANK_HALF_W=80`
    (must stay ≥80) and `_CART_BLANK_UP=45` (shallow — keeps the descending ball
    visible to just above the cart). Falls back to blanking bottom 18% if cart
    unknown.
  - Validated across 9 real playing frames + 3 no-ball frames (parametrized in
    `tests/test_detect.py`). Zero false positives.
- **Play region** (`detect_play_region`) is **panel-anchored**: the gold
  BOOTS/LEVEL/SCORE top panel is the widest gold structure in the top 17%;
  the play field is derived from fixed ratios of the panel width
  (`_PLAY_LEFT_OFF`, `_PLAY_TOP_OFF`, `_PLAY_WIDTH`, `_PLAY_HEIGHT`). The
  outer frame is a dark rope, NOT bright gold — don't detect on that.
- **Aim angle** (`aim_fit` + its wrapper `detect_aim_angle`, used by `_launch`):
  warm dots in the band y∈[0.52,0.83] (`_AIM_BAND`, below bricks, above cart),
  area≥8, small (skip bricks), `fitLine`, returns signed degrees from vertical
  (0 = straight up). None if <3 dots. `aim_fit` also returns the raw dots + the
  fitted unit vector for the debug aim overlay; `detect_aim_angle` is the thin
  angle-only wrapper.
- **Pre-throw prompt** (`detect_prethrow`): the "LOCK CART POSITION" and "THROW
  BOOT" states both show the same grey ␣ space-bar key icon. It's a dull-grey
  sprite (hard to color-threshold, and a joker decoration at the right edge is
  more grey), so we **template-match** it: `bootbreaker/key_icon.png` (56×60,
  extracted from `main.png`) matched in band y[0.5,0.72] with
  `TM_CCOEFF_NORMED`. Scores ~1.0 in pre-throw frames vs ~0.6 mid-play; threshold
  `_PRETHROW_THRESHOLD=0.8`. Template loaded lazily + cached. NOTE: relies on the
  popup rendering at a consistent pixel scale (region ~830 wide on both the test
  screenshots and the user's 2559×1439 shot — verified). `tmp/verify_keyicon.py`.
- **Cart** (`detect_cart`): red mask (two HSV ranges — red wraps the hue
  circle) in the bottom `strip_frac=0.25`. The red top is **split into two
  blocks** by the central ornament; keep all blocks ≥`_CART_PART_FRAC=0.4` of
  the largest (both awning halves, dropping decorations/arrows) and return the
  **midpoint of their combined span**, NOT one half's centroid (that put the
  centre ~38px right of the true cart centre — it caught off-edge).

## Bot logic notes (`bot.py`)

`Bot.step(frame)` returns "launch"/"left"/"right"/"hold". States: **AIM**
(prompt visible + no ball -> "launch"), **PLAYING** (ball -> track), **WAIT**
(neither -> hold). Launch is driven entirely by `detect_prethrow`; there is no
miss-counting relaunch heuristic anymore (it fired at random moments).
`prime_playing()` (called by main after a throw) just resets tracking.
Constructor: `deadzone=40, paddle_frac=0.9, chase_gap=6`.

- **Static false-positive rejection:** a detection pinned to the same spot
  (±`_STATIC_EPS=3`px) for `_STATIC_FRAMES=5` frames is dropped — the real ball
  always moves, so a still blob is a boot-shaped brick / UI icon (matches the
  cyan+brown signature). Without this the cart oscillated forever under a
  phantom at the top of the field after the ball was lost.
- **Tracking:** always steer toward the ball's x (pre-position). Once the ball
  is **descending** (`ball_y > prev_ball_y`), lead to `predict_intercept_x`
  (with wall-bounce reflection). Only-care-about-downward was a deliberate ask.
- **Brief loss:** keep the last target through short gaps (a catch dips the
  ball into the cart's blind spot). But once gone > `chase_gap=6` frames, the
  ball has left play → return "hold" and clear the target (stop thrashing a
  stale target) until relaunch.
- Exposes `last_ball`, `last_cart`, `_target_x`, `last_prethrow` for debug
  drawing/HUD.

## Solved: the frame-rate stalls (pydirectinput PAUSE)

Play logs showed the loop dipping to ~7 fps (130ms) on the exact frames where
the cart *changed direction*, staying at 30–45 fps when the key was unchanged.
Root cause: `pydirectinput`'s global `PAUSE` (~0.1s) sleeps after **every** key
press/release. So each cart reversal froze the whole capture loop for 130ms —
the ball jumped 100+px and detection dropped mid-catch. Fix (in `input.py`):
`pydirectinput.PAUSE = 0` at import. A regression test in `test_input.py` locks
it at 0. **If you ever see periodic ~130ms stalls again, check PAUSE first.**

**Corollary — the tap gotcha (bit us on lock/throw):** `pydirectinput.press()`
sends keyDown then keyUp back-to-back; the ONLY hold time it ever had came from
PAUSE. With `PAUSE=0` a `press()` tap lasts microseconds, which Dota's per-frame
(~16ms) input polling misses — the bot reached "launch" and called `tap_space`,
but the cart never locked/threw. Fix: `tap_space` now does keyDown → `sleep(0.06)`
→ keyUp so Space is held for a few frames. Only runs in `_launch` (already slow),
never in the tracking loop. Locked by `test_tap_space_holds_then_releases`.

Note: the `--debug` OpenCV window (imshow + waitKey every 3rd frame) adds its
own smaller hitches — run **without `--debug`** for real play.

## Prediction robustness (learned from logs)

`predict_intercept_x` takes `max_lookahead=4.0`: it only trusts the wall-bounce
landing when reaching the paddle is ≤4 frames of travel; otherwise (ball high /
falling slowly, tiny `vy`) it just tracks the ball's x. Without this, a few px
of single-frame jitter got amplified by a huge look-ahead into a landing
hundreds of px off, driving the cart the wrong way. `bot.py` also averages
velocity over the last `_VEL_WINDOW=3` sightings (single-frame velocity is far
too noisy) and drops history across detection gaps.

## Working style with this user

- Iterative test-and-report loop: user plays, pastes a log, we diagnose and fix
  one thing. Follow `systematic-debugging` — find root cause before fixing.
- User manages their own git identity ("only do the rest"). Commit when asked;
  don't set `git config` user/email.
- Prefer running shell commands one at a time when the user asks.
- Design/spec + plan live in `docs/superpowers/specs/` and
  `docs/superpowers/plans/` (2026-07-06 bootbreaker files).

---
> Source: [jackblk/dota-bootbreaker-auto](https://github.com/jackblk/dota-bootbreaker-auto) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
