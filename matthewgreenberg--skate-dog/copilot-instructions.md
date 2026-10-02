## skate-dog

> React 19 + @react-three/fiber + three.js. A boy rides a dachshund through a

# Skate Dog

React 19 + @react-three/fiber + three.js. A boy rides a dachshund through a
pastel skatepark. Entirely procedural — no image files, no network fetches.
Textures painted into canvases at load, geometry built in code, props and
foliage baked into InstancedMeshes once and never touched.

Assets: `public/boy.glb`, `public/dog_compressed.glb`. No animation clips —
both driven every frame by pose tables (`player/boneRig.js`). Re-decimate after
any asset change:

```
gltf-transform simplify in.glb tmp.glb --ratio 0.2 --error 0.002
gltf-transform draco tmp.glb out.glb    # simplify decodes draco; must re-encode
```

Decoders in `public/{draco,basis}`, intro font in `public/fonts` — drei/troika
default to gstatic CDNs and nothing here touches the network.

## Layout

```
src/game/
  palette.js        art-direction contract — C (albedo), M (roughness), LIGHT, TONE, RAMP
  photo.js          deterministic capture poses
  store.js          useGame = UI state; P = per-frame state (never React state)
  goals.js          the run's challenge table
  level/
    levelData.js    authored layout; renderer AND colliders read it
    rails.js        grind paths (drawn tubes + derived wall/planter/bench lips)
    decals.js       world-space floor detail quads over a texture atlas
    colliders.js    simplified collision built from levelData
    parkGeometry.js plaza / grass / ramp meshes
    bowlGeometry.js analytic bowl — the drawn surface IS the ridden surface
    textures.js     every procedural map: albedo, normal, roughness, baked AO
    foliage.js      plant generation, pure data -> instance rows
    levelEdits.js   editor contract shared by Editor.jsx and EditorPanel.jsx
  components/       Game, Lighting, Skatepark, Props, Player, Effects, UI, Editor
  audio/AudioManager.js   fully synthesised SFX, no files
  player/           PlayerController.js (movement/tricks/grinding), boneRig.js
tools/              capture harness
ref/                reference stills the art is measured against
```

## Palette

> A shadowed surface reads at (0.62, 0.62, 0.78) of its sunlit self → a golden
> key plus a cool violet ambient **whose sum is neutral white**. The scene is
> warm-*painted*, not warm-*lit*.

- `C` is albedo under white light. Never pre-warm it. **Too orange = the light
  is wrong, not the paint.** An albedo must BE the reference's sunlit reading.
- Shadow tints come from `SHADOW_TRANSFER`, never a grey multiply.
- Per-instance variation samples `RAMP.*`; HSL jitter reads as noise.

## The run

A 2:00 clock you extend by playing: bone or challenge +15s, bail −5s, zero →
scorecard (`RUN_TIME`/`TIME_BONUS`/`TIME_BAIL` in store.js). Level blobs may
carry `rules: { time, goalIds, timeBonus, subtitle }`; `setRunRules` syncs
`P.timeLeft` + UI clock, `activeGoals()` is the one filtered list every HUD
count reads. Time is the only resource — there is no `lives`.

- **The clock lives on `P.timeLeft`, not the store.** GameLoop mirrors it only
  on whole-second change. `addTime()` writes both.
- **Every goal is detected from an event the controller already emits** — a goal
  that owns its own timer/collider/probe drifts from the scorer. Two score tiers
  poll at 4Hz.
- **`complete()` must be idempotent** — every predicate is on a repeating event.
- **Grind payout does not go through `award()`** (double-count); grind
  challenges listen for `'trick'`.
- **`P.inBowl` is a SURFACE flag** — false while airborne over the hole.
- Restart is `resetPlayer()` + `resetGoals()` + `restart()`; the `runId` bump
  remounts Bones/Letters/Cans and **a remount IS the reset** for their `useRef`
  "already got" flags.
- **The bowl is a hole with no side walls.** `resolveCollision` only pushes x/z,
  so `clampToBowl()` runs after every integration in `step`/`stepAir`/
  `stepBail`. POSITION ONLY — zeroing `vel.y` there ate descent momentum.
- **Personal best is per LEVEL, one writer** — `highScore.js` keys
  `skatedog.best` by level id, `endRun()` is the only writer. Reads try/catch'd,
  `localStorage` reached lazily (`ls()`) so node can run the checks. Cans
  progress under `skatedog.canBest`.

**Bail is two bodies.** The boy is thrown clear (`P.bailBoyPos/Vel`, world
space) keeping the momentum the dog loses. Below `SPLAT_SPEED` (3.6) he hops
into the `land` crouch; above it (`P.bailSplat`) he belly-slides — `sec.lie`
pitches him about his feet origin, LIE 1.5 stops short of π/2. His origin uses
`P.pos`'s paw-line convention, so grounding is a clamp against `groundHeightAt`.
`sec.eject` blends the world offset in through the inverse of `P.quat` while
fading the `backY` mount out; both SNAP to 0 at bail end (respawn is a teleport).

- **Dog tumble clearance is angle-aware** (`bailClearance()`): pivoting about the
  paw line, a nose-down pitch needs half the dog's LENGTH, an inversion needs the
  back height.
- Tumble→settle is TIME-based (`BAIL_SETTLE`), not contact-based — the clearance
  floor rises faster than the pop climbs, so a contact latch kills the tumble
  before it turns once. Settle damps pitch/roll to the nearest 2π (never π).
- `tumbleW`/`rollW` scale with crash speed; `BAIL_HITSTOP` 0.07s freezes both
  bodies (world/camera/FX keep moving); contact bleeds spin at a RATE (1.1/s) —
  a per-hit multiplier killed it in 0.2s. Descending end-plant converts spin to a
  hop; slams (`vel.y < -2`) rebound and emit `'land'` + `'dust'`.
  `updateAnim`'s dogRoll damp must skip bail.

**HUD.** One `GoalList` in two places: briefing card on the start overlay, `☰
n/8` pill opening a sheet mid-run. Ranked SCORE > clock > menu, not three equal
widgets. `--rim` is a hairline; pill shadows are contact shadows. Clock is a rose
sticker, `is-urgent` a deeper rose (not a hue change). The menu is never faded.
Score icon is a PAW. Esc toggles the sheet and sets `P.paused`, which gates the
whole `g.started` branch in GameLoop — the effect must clear it on UNMOUNT too.
Under `(pointer: coarse)` `.hud-pill`/`.hud-menu` redefine `--u` at 0.74.

**Start frame** — title, dog, briefing card are three independently anchored
claims on the frame (hanging the title off `P.pos` followed the dog off-screen).
Intro.jsx scales the title to 0.42 of frame WIDTH via
`viewport.getCurrentViewport`. One clock (`P.intro`, 1→0 over 1.5s) drives the
camera blend, title dissolve and Player.jsx's rig slerp; the sim runs THROUGH
the swoop. PHOTO pins `P.intro` to 0 and renders no title.

## Seeing your changes

**Do not eyeball it, and do not trust a diff.** `?shot=<pose>` freezes the sim,
parks the camera, pins the wind clock, flips `window.__shotReady`.

```bash
node tools/shoot.mjs --tag mywork plaza bowl   # -> shots/mywork-{plaza,bowl}.png
node tools/px.mjs shots/mywork-bowl.png open,0.62,0.90,16
node tools/compare.mjs ref/ref-plaza.png shots/mywork-plaza.png out/ key.json
```

Poses: `plaza bowl hero props grove deck pipe bench lamp`. `compare.mjs` builds a
**blind** A/B sheet. Harness runs its own dev server on **3210**.
`tools/chrome.mjs` excludes Chrome **148.x on purpose** — its headless shell
never fires ResizeObserver, so R3F never builds a renderer and the page renders
nothing with no error. `?ao=1` renders the AO buffer alone.

## Self-checks

Plain `node`, no framework. Run after touching what they cover.

```bash
node src/game/level/foliage.check.js      # crowns, branch coverage, colour space
node src/game/level/benches.check.js      # bench facing + footing
node src/game/level/rails.check.js        # rail/post clearance; samples the DRAWN curve
node src/game/level/bones.check.js        # float band/spacing, can clearance
node src/game/level/decals.check.js       # on flat plaza, in bounds, inside the atlas gutter
node src/game/level/levelEdits.check.js   # commits reach colliders+paths, undo unwinds, save round-trips
node src/game/goals.check.js              # each challenge pays once; predicates discriminate
node src/game/level/ramps.check.js        # every ramp/stair enterable, climbable, qp1 pops vert
node src/game/level/collision.check.js    # ~40s: broad phase, penetration, seams, dt consistency
node src/game/input.check.js              # touch stick converges on the stick angle
node src/game/player/steering.check.js
node src/game/player/scoring.check.js     # live grind payout + combo chain
node src/game/player/bail.check.js        # tumble clears the floor, settles flat
node src/game/player/boneRig.check.js     # rider joint angles, in world space
node src/game/components/shadowfit.check.js
node src/game/components/clearCoat.check.js
node src/game/components/recolor.check.js
```

`boneRig.check.js` rebuilds boy.glb's skeleton from the glTF node tree (no
loader, no DOM) and runs Rider.jsx's call sequence in world space.
`foliage.check.js` asserts a **grain** check (clump under 32% of its mass,
`n * ratio^2` over 0.9) and bed **coverage** of 82%+ — deliberately not solid.
When you add a visual invariant, add the assertion.

## Sim / player

`updatePlayer` consumes the substep accumulator remainder as one variable-size
step (whole 1/120 steps stuttered 11cm at speed). Safe because every response in
`step()` is rate-based and dt-scaled.

- **Scoring.** `airTrick()` is the one table of what an air is worth; stepAir
  flushes every 0.1s and scoreAir reads the same function at landing, so the
  popup can never name a trick the landing doesn't pay. Air points bank at
  landing; grind pays LIVE (42 pts/s x chain, flushed every 0.15s, remainder
  settled at exit without `award()`). `CHAIN_GRACE` 0.6s, ground only. Pool Gap
  (+400) needs all three of leave-clear, k<0.7, land-clear.
- **A grind ollie with a direction held kicks sideways** (`GRIND_HOP`) — one
  impulse at the pop, not air steering. Exit velocity is pure rail tangent, so on
  a cap level both sides a straight pop has no exit. With no direction held it
  auto-ejects toward whichever side `groundHeightAt` says is lower
  (`GRIND_PROBE` 1.8m).
- **Air tricks are on the direction keys** (spin, back kickflip — the dog IS the
  board). No air steering, no air throttle: 4.5 m/s² over a one-second vert air
  exactly cancelled the drift back into the transition. A fresh grab press rolls
  a random `GRAB_STYLES` entry, never the same twice.
- **Carve** is TURN_LOW/HIGH x GRIP — raising one alone reads as understeer.
  Steering against your held lean boosts both by `shift` (= -steer * P.lean).
  Concave creases preserve speed through the ground snap; a bare projection ate
  1 - cos(slope) each way.
- **Clean landing pays momentum** (CLEAN_BOOST + airtime bonus, cap 1.25x
  MAX_SPEED) — the pump loop.
- **Two halfpipe aids.** PUMP is gated on `surf.curv` (nonzero only on a
  quarter's arc), applied along TRAVEL, fading to zero at MAX_SPEED — not on the
  throttle. ALIGN eases the heading onto the fall line when steer is exactly
  zero, off a `PIPE_HOLD` 0.9s timer (not the live slope) so it carries across
  the flat. The bowl is excluded — it wants carving lines. `resetPlayer` clears
  the hold.
- **Landing in a transition moving opposite your facing auto-turns**, gated on
  facing UPHILL and aiming at downhill = +(nx,nz). Flat fakie untouched.
- `P.surfLift` lifts the rig by analytic path curvature (cap 0.12). Rig
  up-tracking: ground damp 26 / air 7. Colliders ignore coping lips, so lip
  geometry must protrude less than surfLift clears.

**Mobile.** `input.js`'s `TOUCH` pins quality 'low' (PerformanceManager never
inclines it — a mid-play flip rebuilds the composer), no N8AO, shadow map 1024,
dpr [0.75, 1.5], camera zoom 0.9, MAX_SPEED/ACCEL x0.7. Left stick is
WORLD-directional on the ground; `applyTouchStick` derives steer from heading
error every SUBSTEP with the live heading, or a held stick stops steering as the
dog turns. Saturates at 55% deflection. In the air the raw axes are the trick pad.

## The level editor (`?edit` or `/edit`)

`?edit` makes the editor AVAILABLE; `useEditor`'s `editing` says which half you
are in. While editing the chase rig is an orbit camera and the sim is paused
before `sampleInput`. `levelEdits.js` is the contract both halves share.

**The editor mutates `levelData`'s exported arrays in place — the level IS the
document.** Downstream is either a pure function of those arrays
(Skatepark/Props, content-keyed per table) or a module-load snapshot
(`colliders.js`'s `cols` + grid, `rails.js`'s `PATHS`) — those got
`rebuildColliders()`/`rebuildPaths()`, which refill the **same arrays in place**
because PlayerController holds them by reference. `bumpLevel()` calls both.

- **A TOOL IS NOT A TABLE.** `TOOLS` is the one palette both halves read —
  labels, tints, 1..9 bindings, ghost footprint, `patch`. SOLIDS is four tools
  behind one `kind`, so `DEFAULTS.SOLIDS` carries no `top`/`style` (heal()'s KIND
  table fills them) and `addRow` renames the row after the TOOL.
- **A tool can be a GROUP, and a group is ONE object.** The halfpipe tool places
  five rows in one begin/commit under a shared `grp` stamp.
  `moveGroup`/`rotateGroup` live in levelEdits so gizmo, panel and node check
  share one implementation. Translate write-back is a DELTA. Width/height/length
  are group properties. Duplicate/delete take it all.
- **The ghost's red state is authoritative.** `placementInfo()` is a pure query
  → `{ warn, matchTop, block }`; every `warn` also sets `block`. Covers burial
  (per `rampTopAt`), blocked run-up, footprint in the pool, `matchTop`.
  `ghostShapes()` geometry is DISPOSED on tool switch.
- **A quarter's rise can never outgrow its run** — past h = d the arc puts the
  drawn top below y1 while coping and collider stay at y1. `heal()` grows `d`.
- **Size steppers are per-AXIS**, each reading whichever field the row carries
  (`top`/`h`/`y1`); `y1` floored against `y0`.
- **A rail has no fields to step** — `railLength`/`extendRail`/`turnRail`/
  `liftRail` mutate `pts` in place. Extend pushes the last point along the final
  segment, floored at 0.5m; the check asserts it stays GRINDABLE.
- **Walls and rails bend differently.** `bendRail` resamples to 5 points, yaws
  each segment, recentres on the old centroid, so Bend and Length stay
  independent. A wall is a BOX, so `bend` expands via `wallSegments()` into
  overlapping chords (overlap 0.25; chords take NO end trim in `lipEdges` or a
  bent wall stops being grindable). `bend` 0 returns the row itself. Pick proxy
  and footprint use the straight box.
- **Snapping is grid PLUS flush faces, and the object snap wins** (the grid is
  always within half a cell). Rotated rows skipped as sources; only axes the
  handle moved are snapped; `translationSnap` unset because TC rounds relative to
  the drag start.
- **Materials are swatches, not dropdowns.** `LOOKS` is a FUNCTION of the row
  (SOLIDS' options hang off `kind`). Swatch hexes are hardcoded mirrors of `C.*`
  so the module stays node-loadable.
- **Scene settings**: `useSceneSettings` holds `{ time, ground, pattern }` with
  `TIMES`/`GROUNDS`/`PATTERNS`. **The defaults ARE the shipped art** — `sunset`
  carries no overrides, `classic` tints white, so a plain visit and the harness
  render byte-identical. Pattern swaps `map`/`normalMap` with no `needsUpdate`
  (both slots always occupied). Lives in a Plaza `useLayoutEffect`; Lighting
  subscribes directly, so a settings click never rebuilds the park.
  `Environment` is keyed on the time preset id (`frames={1}` bakes per mount).
  The sun DIRECTION deliberately doesn't move — `SUN`/`LIGHT_BASIS` are what the
  shadow fit and shader cookie are built on.
- **The pool is not a row** — `BOWL` gets `setBowl(patch)` and its own selection
  slot (`bowlSel`).
- **Everything is a proxy layer**, one pickable box per row in a sibling group of
  Skatepark's root. Idle proxies are `opacity 0`, not faint. `<Grid>` sits at
  y = −0.02, UNDER the plaza.
- **Adding is click-to-place, and while a table is armed nothing else raycasts**
  — proxies take `raycast = () => null` and the gizmo unmounts, or an opacity-0
  box eats the placement click. The ghost is mutated directly. `select()` clears
  `add`.
- **SPAWN is gizmo-able but is not a row** — its selection lives in Editor.jsx's
  `spawnSel`, kept in sync by SUBSCRIBING to the store. Drag-end writes
  `SPAWN.x/z` and calls `commit()`, not `setSpawn()` (two snapshots = two undos).
- **Undo is whole-level snapshots.** `restore()` replaces row objects wholesale,
  so gizmo and panel re-resolve the selection by `__k`.
- **`__k`** is a stable per-row editor key (tables lack `id`; array indices break
  on the first delete). Inspector inputs are keyed on `__k` AND the version, or
  React shows the previous row's drafts. Fields commit on blur/Enter.
- **`derived: true`** marks rows recomputed from another (handrails);
  `editable()` filters them out. TREES/SHRUBS aren't tables.
- **Rotate is gated per table** (rows with `rot`); no scale gizmo. `ROTATABLE`
  lives in levelEdits.js because the panel shows Move/Rotate only for rotatable
  selections. Y translate handle gated the same way (`SHOW_Y` =
  BONES/LETTERS/RAILS). **R turns what you are HOLDING** (`addRot`, sticky);
  RAILS has no `rot` so R rotates its `pts`; nothing armed → gizmo rotate.
- **`TransformControls` needs `makeDefault` on `OrbitControls`** or they fight.
  Write-back once on `dragging-changed` false.
- **A row built outside levelData's `box()`/`wall()` arrives without `rot`/`base`
  and nothing downstream defaults them** — `Math.cos(undefined)` is NaN,
  `l < 1e-4` is false for NaN, and `findGrind` then dereferenced null and killed
  the sim near any new wall. Three fixes, all needed: `DEFAULTS` carry every
  field their consumers read, `heal()` refills on load and on kind/style switch,
  and both tests are NaN-safe (`!(l > 1e-4)`, `if (!hit.tan) continue`).
- **Edits survive a refresh.** `saveLevel()` runs from `bumpLevel()`,
  undebounced. `loadLevel()` runs once at import in a try/catch. Restored rows
  must be re-stamped by `keyOf()`. `SHIPPED` is a deep snapshot taken BEFORE the
  load.
- **Named user levels are a second store** — `skatedog.levels` holds
  `{ id, name, at, thumb, data }`; `/?level=<id>` plays one via `applyBlob`.
  Playing or building from home is a full navigation on purpose (`EDIT`,
  `loadLevel`, music duck all run at import). `thumbCapture` renders a fresh
  frame first (preserveDrawingBuffer is off) and retries without the thumb on
  quota.
- **Dog Bowling** (`dog-bowling`) is a protected shipped mode, not in
  localStorage: empty plaza, 151 cans in 3x3 bundles on four serpentine
  straights, 30s, cans only, zero bonus. Its dog is 2.4 (boy stays 1.58) — the
  dog reads as the bowling ball without bypassing size-aware code. Cans-only mode
  turns the score pill into a live can count; the last can ends the run. Saved
  and built-in end cards offer GO HOME; the shipped park omits it.
- **`setEditing(on)` is the whole build→play→build transition** — `bumpLevel()`,
  `resetPlayer()`, `resetGoals()`, `restart()`, `started: true`,
  `P.paused = false`, `P.intro = 0`. Its Esc listener is CAPTURE-phase with
  `stopPropagation` so GameUI's own Esc doesn't also fire.
- **`clearAll()` gives a blank canvas.** BOWL isn't a row, so it gets `BOWL.on`,
  checked in exactly three places: `sampleSurface`, the plaza cutout, `<Bowl/>`.
  `BowlProbe` stays mounted either way — it bakes the reflection the coping and
  rail materials read and `Warmup` blocks on its ready signal.
- **`parkAOMap` is the one map not cached forever** — commits refill
  `AO_FOOTPRINTS` and defer the bake; entering play-test invalidates once.
- **Fog drops while editing** (FOG_NEAR 22). **Held SPACE is the hand** — swaps
  OrbitControls' left button to PAN; `groundDown` returns early while held; keyup
  is its own listener and `blur` clears it (alt-tab never delivers keyup).
- **The soundtrack ducks to 30% while editing** — duck, not pause, a separate
  factor from `muted`.
- **The editor is DESKTOP ONLY**, gated on `(pointer: coarse)` being false,
  inlined in levelEdits.js rather than imported from input.js because the node
  checks pull this file in and it must stay game-module-free.
- **Export is the CLIPBOARD, not codegen** — levelData rows carry derived
  expressions a round-tripper would flatten.
- **`loadLevel()` runs at import only under `?edit`**, so a plain visit is always
  the shipped park. Going to PLAY still carries the edits.
- Panel order is WHAT (palette), WHERE (one hint line naming the next ACTION),
  IS IT RIGHT (steppers). Numbers hide in `<details>`. Only the middle strip
  scrolls. Armed tools read as LIFTED, not recoloured. Two CSS traps:
  `.ed-panel button` sets a `font:` SHORTHAND so a bare `.ed-play
  { font-family }` loses on specificity; and `--e` is the panel's own unit,
  deliberately not the HUD's `--u`.

Nothing in the editor touches `?shot=` or the node checks.

## Level content

- **hpN/hpS are a halfpipe** on a raised 0.35 `hpDeck`: two facing quarters,
  `HP_FLAT` 3.2m, `HP_H` 2.0 (h == run is dead vertical and the arc degenerates).
  All four boxes are style `'solid'` at width 12. `SolidSlab` uses `texBox`, not
  `RoundedBox`, whose extruded-shape UVs put the flat's planks 90deg off.
  `hpDeckN/S` are top decks behind the coping — a freestanding quarter is
  zero-thickness at its lip. Keep the run-in clear.
- **deckA's north/east and deckB's east walls are THICK (3m) on purpose**, flush
  to the kerb. A collider face at the play edge puts probes OUTSIDE the clamp, so
  collision.check skips those (`inPlay`).
- **The plaza centre is deliberately open.** Don't refill it with props.
  `HP_CLEAR` keeps ring ellipses out of the halfpipe; TREES drops any ring tree
  inside PERIMETER (shrubs may dip in).
- **Before adding a transition, check it has a face AND a run-up** (bank1 was
  buried inside deckB and deleted).
- **Every wall cap, planter rim and bench seat is a grind path** (78), derived in
  rails.js from the same boxes. Both top EDGES of a cap, not the centreline.
  Offsets are measured off the DRAWN stone; a wall cap's top is `w.h` absolutely
  (`base` does not add). Planter rims are four separate runs, not a loop (a loop
  snaps the heading 90deg at each corner). A bench is ONE run along the FRONT
  edge, trimmed 0.15 a side. findGrind's dy window (0.75 up / 0.45 down) stops a
  cap overhead grabbing you.
- **Handrail offset is `w/2 - 0.7`** and rails.check walks the drawn catmull-rom
  at 10cm — a handrail doesn't know what it's standing in.
- **Bones** (5) are solved against measured launch heights; collection is a 1.1m
  sphere on `P.pos + 0.45`. Exports R2 and POP because Letters.jsx collects on
  exactly those.
- **Letters** (D-O-G, S-K-A-T-E) are BILLBOARDED troika text — reading which one
  you need is the objective. Hidden until `started` via `visible`, NOT an unmount
  (troika builds its SDF asynchronously).
- **Cans are NOT colliders** — you ride through and they burst. The hit is a
  horizontal circle gated on the FEET being in the can's height band, or a big
  air over the top. The dot field is an alphaMap (alphaTest, not transparent).
  Two z-fights paid for: parts lifted 8mm off the paving, and the drum's bottom
  cap recessed 2.5cm into the foot ring (`FOOT_IN`) — everything above offsets
  with it, LID_Y included. The wreck is ballistic and never asks the level a
  question after the first frame; its height floor is the body's half-DIAGONAL.
  Emits `'smash'`; AudioManager clangs an INHARMONIC drum (equal-tempered
  partials read as a bell, and a bell reads as a reward).
- **Floor decals are NOT baked into plazaMap** (it tiles every 8m). World-space
  quads over a 4x4 `decalAtlas` with a transparent GUTTER, merged into ONE
  geometry (~350 tris). Per-decal fade rides vertex ALPHA — scaling RGB renders a
  faint chalk mark as a DARK one. Placement is `groundHeightAt` and nothing else;
  the footprint is sampled as a 0.35m GRID, not four corners, and the perimeter
  test adds the quad's HALF-DIAGONAL. Mesh sits 6mm up, transparent with
  depthWrite off — not alphaTest, which stencils a hard edge onto every line.
- **Lamps** (`lampModel.js`): `lampParts()` is the part list Props instances;
  `createLampPostModel()` is preview-only (a Group per lamp is 9x the draw
  calls). Flat-shaded hex vs smooth turned parts is the whole material split;
  mullions sit ON hex corners with a composed YXZ yaw-then-tilt (XYZ read as
  BENT). One shadowless PointLight each, gated on TOUCH, **not on quality** —
  quality starts low and inclines, which would pop lights mid-run.

## Characters

- **The dog's fit is measured, not typed.** `Box3.setFromObject` on a SkinnedMesh
  returns skinned rest bounds; the raw position attribute is quantized and
  pre-skinning. The dog is authored nose at +X and yawed -90, so a bone delta
  swings fore/aft about **Z**, an ear about X, the tail about Y. `BACK_Y` (0.355)
  is where the rider's feet go.
- `dogFit.js` holds the shared numbers (a component file may not export shared
  state under react-refresh). `useCharacterSize` is the one reactive source both
  the editor panel and Player subscribe to; it must NOT call `bumpLevel()`. Dog
  and boy sizes are independent — cohesion comes from the planted-foot mount
  (`backY()`) and size-aware trick offsets, never a forced ratio. `LEG_DROP`
  comes out of both the model lift and `backY()`.
- **An imported bone does not rest at identity** (boy.glb's `L_Thigh` rests near
  180deg about Y). Angles go on as world-space deltas conjugated into the rest
  frame (`setBone`).
- **The bind pose is not the pose the numbers mean.** `alignBone` bakes a
  per-bone correction so zero is straight down on both sides. Leg length can't be
  corrected the same way — pelvis height uses the mean, floating one foot ~1.5cm.
- **The rider's crouch is MIRRORED across the pair**; `body.position.y` takes the
  mean, or an asymmetric crouch floats one foot. `recolor.js` hue-rotates
  shirt/sleeves +168deg and shoes -28deg — a `material.color` tint can't do it
  (orange x blue is grey). Garments picked by `map.name`.
- **The dog's KTX2 pixels never reach the CPU**, so no hue-rotate. The coat gets
  procedural `dogNormal`/`dogRough` plus `coatShader` (onBeforeCompile): hue via
  Rodrigues about the grey axis, saturation 1.4, contrast 1.14 about a **0.2
  LINEAR** pivot (sRGB's 0.5 crushes to black), violet-sky rim (fresnel^3). Knobs
  live in one module-scope `COAT_ADJ` uniform with its own leva folder (two
  useControls on the same PATH clash). No roughness slider — assigning
  `mat.roughness` writes to a useMemo return value.
- **Carving bends the whole body** — rear legs hang off tripoRoot and the front
  half off tripoSpine_0, so yawing spine + chest swings shoulders, front legs and
  head into the turn while the hips hold heading. Split 0.8/1.2, `lag` whips a
  reversal. LONG stretches x outside these bones, so a longer dog needs a bigger
  bend.
- **Ears, tail and tongue trail a carve off one `lag` signal.** Ear springs are
  also kicked by rig acceleration (P.vel differenced), clamped to ±60 — a landing
  kills 9 m/s in one substep, so the clamp makes it an impulse and swallows
  respawn teleports. WiggleBone was passed over: these bones carry authored
  motion a solver would own.
- **The tongue is a capsule** authored at the bind-pose mouth then `attach`ed
  (not `add`ed) to the skull bone, so offset, lay-forward rotation, the bone's
  rest frame and HEAD scale all carry over. Outer group is the mount, inner is
  free for the frame loop.
- `clearCoat.js` is **RIDER ONLY** — a coated dog reads as wet plastic.
  `MeshPhysicalMaterial.copy` cannot read a standard material (every physical
  param lands `undefined`), so it borrows `MeshStandardMaterial.prototype.copy`
  and puts the `PHYSICAL` define back. `USE_CLEARCOAT` is keyed on
  `clearcoat > 0`; the renderer re-picks a program only when the material version
  moves.
- There is deliberately **no dog voice**. Don't reintroduce barking.

## Collision

- **A ramp is not a box, and it is a hole in the deck it feeds.** One rule,
  `rampTopAt()`: a ramp's footprint suppresses any FLAT no taller than the ramp's
  top, and the ramp is measured at the nearest point of its footprint, not its
  peak.
- **Arc length starts at the low edge the footprint kept.** The footprint grows
  by `RAMP_OVER` uphill only, so `s = lz + hd`. Subtracting `RAMP_OVER/2` again
  slid every transition 0.5m uphill of its drawn mesh and hid 0.1–0.78m trenches
  STEP_UP silently jumped.
- **The broad phase must be dilated by the query radius** — raw AABB bucketing
  left 22 of 58 colliders unreachable. `GRID_PAD` 0.6 >= player RADIUS.
- **Ground adhesion must be geometric** — a real convex lip drops ~speed·dt per
  substep, so the branch uses `max(0.12, speed·dt·2)` (a constant 1.5m glued you
  to every deck edge). Same family: air landing samples with the same `feetY` the
  resolver used; grind entry projects velocity onto the rail tangent (the 3D
  magnitude turned a 9 m/s fall into 9 m/s of rail speed); `doJump` zeroes the
  coyote window (a double-tap inside 0.13s stacked a 5.2m ollie);
  `slideAlongWall` only steers the heading on the ground.
- **Crossing a ramp's top, `gap` is ~0**, so the ground-snap branch caught it and
  `reproject` deleted the vertical velocity. The test is the velocity's
  separation from the NEW normal (`sep`), positive only at the lip. Past that
  `launchOffLip()` rotates the exit toward vertical, conserving speed,
  overshooting to a slightly NEGATIVE horizontal so a vert air re-enters the
  transition. Banks sit below `VERT_LO` and still launch forward.
- **The launch branch sets state to `air`**, so `if (jumpBuffer && state ===
  'ground')` never ran and ollies at the coping were eaten. That branch spends
  the jump itself now, AND `doJump` calls `launchOffLip()`. Do not apply the
  redirect in both paths — the second flips the horizontal forward. The coyote
  jump zeroes `P.slope` first. Flatground ollie is 1.28m.
- **A wall cap is landable, never steppable.** `CAP_STEP` splits by where the
  body is: 0.3 inside the footprint (a landing frame sinks ~0.12m before
  stepAir's land check fires) but ~0 outside (pressing into the FACE is always a
  wall). Solid caps create corner pockets, so the resolver runs 8 passes with an
  early break.
- **Wall response is rate-based, or the camera lurches.** Bleed ~0.25s, turn
  6-14 rad-eq/s. Two traps: a near-dead head-on hit has no tangential velocity to
  pick a tangent from, and in an inside corner each face's tangent points into
  the other wall. The turn target blends the OUTWARD normal by `headOn`, with a
  facing-based fallback below 0.05 m/s. CameraController aims off a
  ~0.2s-smoothed copy of P.vel, never the raw value.
- **Ground drive scales by `n.y`** and air throttle is deleted — free thrust made
  ramps feel wrong twice.
- **A downhill must pay gravity visibly** — the tangent gravity term scales by
  `SLOPE_GRAVITY` while grounded and rolling resistance scales with `n.y`.
- **The shadow fit plane is latched, not live** — `planeY` only updates when
  `P.state !== 'air'`.

## Foliage

- **A foliage instance is a LEAF BLADE, not a ball.** `leafBlade` is a succulent
  paddle along +Y, 7 stations x a 6-point ring (72 tris, both ends pinch so no
  caps). The ring is 6 because a 4-gon section is a rhombus whose crease smooth
  normals turn into a specular LINE; it's PHASED half a step (`RING_PHASE`) so a
  vertex lands on the crest, and `RING_K` divides cos(30) back out. PROFILE is a
  true lanceolate — 35% of max at the base, crest at 45%, long taper to a point.
  Broad and fleshy is measured: 2.18:1 at max width, 3.28:1 mean. Do NOT narrow W
  to fix a fat bed — that's a profile problem. T matters MORE than W, because the
  roll slot is 0 so half of any rosette presents its SECTION to the camera.
- **Trees use a SECOND geometry** (`CROWN_LEAF`, W 0.32 / L 2.55 / T 0.26, 32
  tris) sharing the material — one extra draw call, not a second wind program.
  Its lanceolate point IS the read. **Do not unify the two blades.** Thin costs
  coverage (bare fraction goes as exp(-n·w·l·r²)). A perf trim traded coverage
  back across all species (massClumps x0.72, clumpR x1.18, n·r² = 1.002): park
  total 9.1M → 4.6M triangles. Every species' `bloom.per` is per LEAF ROW, so all
  three divide by 0.72. clumpR is 1.27x its pre-thinning value — if crowns read
  as faceted boulders again, that's the number.
- **A clump is AIMED.** `clump()` takes the outward vector of its mass. Slot 8
  (roll) stays 0 because bake() reads YXZ and applies Z FIRST, so a roll swings
  the blade off the aim. Per-leaf variety is a jitter cone ramping DOWN at the rim
  (0.34 → 0.24) — the only thing that can throw a blade past the envelope.
- **A canopy mass is NOT aimed outward** — a radial blade presents its TIP and a
  dome of those is a pin-cushion. `leafMass` swings the aim 55-70deg into a
  tangent frame with a downward hang RAMPING with `edge` (0.26 middle, 0.88 rim).
  The underside easing knee is -0.75 — foliage.check's crown floor reads centre
  and radius and cannot see an aim, so that knee is the only thing keeping the
  skirt off the ground.
- **A bed is COUNTABLE ROSETTES on a JITTERED GRID** — a Poisson scatter fused a
  third of the stars. Pitch is `gap` 2.9 x plan reach. **The bed is not supposed
  to close**: target ~92% mean / 87% worst (gap 2.0 → 99.9%, 2.9 → 91.6, 3.2 →
  89.2). Jitter ±9%. 10 blades leave one shared centre (OFF_R 0.14), splayed
  45-75deg. The FINE layer is thin (1.35): its job is stopping soil showing
  THROUGH a plant, not filling space between two. Height ramp tuned for RANGE at
  fixed mean (0.20 + 0.58q). foliage.check models a blade as a plan CAPSULE — the
  single-number disc broke twice.
- **Every species has a `core`** — one leafMass over the crown's centre, or the
  cone over the fork is open to the 40deg camera. Branch-tip masses are pulled
  12% back down the shaft (`TIP_IN`) or their outer clumps read as detached
  specks.
- **A planter tree is `pushTree(..., 'blossom')` BY NAME** — a random draw put a
  3m lawn-species trunk in a 1.4m planter.
- **Flowers.** White and pink are 5-petal `daisy` with a YELLOW EYE — most of
  what separates a daisy from a blob of cream at distance. The pip rides a
  vertex-colour attribute carrying flowerYellow/flowerWhite against WHITE, so
  `MAT.flowerWhite/Pink` need `vertexColors: true` and the yellow bucket must
  NOT. YELLOW is not a daisy: it's `budCluster`, four two-thirds-overlapping
  beads plus a crown bead, 82% of the bed. Bead ring radius is UNDER bead radius
  so they fuse. Scatter 0.055, not the daisies' 0.22. The albedo is a saturated
  GOLD — cream renders as a bleached patch. No lilac in the bed: pale pink is 4%
  because against the lavender rim it read as lilac popcorn. Pink belongs on the
  TREE.
- **Every species carries a `bloom` table** that speckles pink by SAMPLING the
  leaf rows just emitted — a crown built from masses is not a sphere, so solving
  a radius puts a third of them in mid-air.
- **One InstancedMesh set PER 24m CELL** (Props.jsx `byCell`) — that is the
  entire frustum-culling story, since three culls by an InstancedMesh's own
  bounding sphere. `byCell` carries each item's ORIGINAL index because every seed
  is `base + i*stride`. There is deliberately no LOD.
- **`FoliageControls.jsx` is a SEARCH TOOL**, not a shipping feature — bake a
  winner back into palette.js/foliage.js/Props.jsx with the measurement. Knobs
  live in `foliageKnobs.js` (shared mutable state can't live in a component under
  react-refresh). A knob can't be a uniform write (matrices bake once), so
  `bumpFoliage()` re-runs the useMemos through a useSyncExternalStore version;
  `rebuildFoliageGeo()` DISPOSES the old geometry pair. Green knobs deliberately
  do NOT tint MAT.foliage/MAT.crown — LEAF_MAT is white and bake() divides the
  base out, so tinting applies the ramp twice; they reach the screen via
  `refreshStops()`, which rewrites SPECIES against a module-load SNAPSHOT.
  Defaults mirror the shipped art byte-identically.
- **Foliage colour is set by histogram, not by patch.** Bin every green pixel
  (hue 40-140, sat > 0.14) by lightness AND saturation AND hue, and crop the TREE
  and BED separately (the reference's bed is hue 90, its crown 78). An uncropped
  grove capture is mostly lawn, which passes the same green filter and reports
  nothing wrong. Leaf stops carry s74-86 to render at s48-50.
- **A canopy ramped by height alone reads as noise** — from a 40deg camera height
  bands resolve into rings. foliage.js also ramps along the key's bearing
  (`SUN_FACE`); keep it under ~0.25. The shaded end is lifted a third toward the
  mid (`SHADE_LIFT`) — a leaf in shadow is still lit by the violet sky.

## Effects and audio

- **Effects.jsx is a cartoon particle kit** — grind sparks, star pops, shockwave
  rings on jump/grind-start (none on land, rejected), speed streaks, dust, carve
  marks. Fixed-size instanced pools, zero alloc in the frame loop, fades are
  scale-to-zero (no per-instance alpha).
- **Big air** fires once per air past 1.0s (a flat ollie is ~0.69s) — halo +
  trail, shimmer, hang-time-scaled bonus.
- **Ambient air (220 dust motes) has NO pool** — position is analytic in (index,
  time) and the field WRAPS around the camera. CARRIED by a DAMPED copy of the
  camera position (1.1/s): world-anchored, motes streamed past as debris in a
  gale; camera-exact gives zero parallax. `wrapTo` wraps the OFFSET about 0;
  PHOTO snaps hard. `toneMapped: false` but deliberately UNDER the bloom
  threshold — a mote glints, it does not glow.
- **Smash litter** is the one pool with a LIT MeshStandardMaterial. Tumbles about
  a fixed axis in the pool's `q` slots, never asks where the floor is.
- **Clearing a SET** (`'goal'` id `fetch` or `cans`) adds `sfxFanfare` in ONE
  pooled voice — the pool is 8 and a note-per-voice fanfare evicts itself.
- `sfxPlace`/`sfxDelete` call `unlockAudio()` themselves and are node-guarded,
  because the level checks import levelEdits → AudioManager.
- **ui.css's upper-frame haze was REMOVED by request.** Don't reintroduce it via
  three's Fog — that's keyed on DEPTH, and the top of a chase frame is sky at
  every depth.
- **ToonFX.jsx** passes are all OFF by default so the harness is untouched.
  TiltShiftEffect's fragment shader is replaced with a crossfaded ramp driven by
  a `bandParams` vec4 whose slots are NOT ascending (`inner-, outer-, OUTER+,
  inner+`) because the shader reads the upper edge as `smoothstep(w, z)`. Built
  at module scope and mutated — the r3f wrapper rebuilds render targets per tick.

## Textures

- **The PLAZA is drawn at double everyone else's resolution** (albedo 2048,
  normal/rough 1024 = 256 px/m) — the only thing seen at a grazing angle for the
  whole run. Resolution is not a free knob: `grain()` counts go with AREA,
  `blur(Npx)` and lineWidths double. normalFrom's sobel is a FIXED 3px kernel, so
  at 2x it reads a half-width slope — do NOT raise strength to compensate (it
  turned the paver pad's soft shoulder into a bright BEVEL RING). `crack()`'s
  wander is `steps x len`: doubling STEPS is the HD move.
- Scuff and MOSS live in plazaMap only. Moss is seeded ON a joint (one axis
  snapped to the grid, elongated, blurred) — a hard-edged blob mid-slab reads as
  paint. Neither is in plazaRough yet; that's the next thing to add.
- **`dogNormal`/`dogRough` are one fibre height field** sobelled into a normal and
  remapped into a roughness, so the crest the normal bumps is the crest the
  roughness makes glossier. Strokes near a tile edge are redrawn wrapped. They
  ride the ALBEDO's uv channel (Tripo's arbitrary atlas).
- **A bench slat is one board, so its grain runs along its LENGTH.** Slats wear
  the ramp ply maps, whose planks run along canvas *v*, so `slatGeo` writes v from
  local x. Rides plank 6 (darkest), parked on its CENTRE with ±0.04 span (or the
  black plank-edge line runs down every slat), v clear of the butt seam. The tint
  is NOT `C.benchWood` — it multiplies a map that already carries a mid-tan and
  was SOLVED against a capture; re-measure if the light rig moves. Bench-only
  MeshPhysicalMaterial for `sheen`. Ramp materials deliberately go without —
  they're what the halfpipe captures are compared against.
- **Tone-map operating point.** Khronos PBR Neutral is identity below linear 0.76
  then compresses hard. Sitting the plaza at 1.6 gives a slope of 0.05, where no
  shading can move a sunlit pixel.
- **Cylindrical UVs converge** — the bowl's flat uses a planar mapping; the
  cylindrical one puts a singularity at the centre and, with no tangent
  attribute, three derives the frame from screen-space derivatives (a starburst
  even where the texture is one colour).
- **`v=1` is the TOP of the canvas** under three's default `flipY`.
- **`bake()` reads row slots 9-11 as an *absolute linear* colour** and divides the
  material base out. Emitting a multiplier asks for `instanceColor` ~3.2 and
  renders white.
- **Wind and non-uniform scale.** The wind shader inverts the instance basis
  per-column. Dividing all three axes by column 0's length squared is only right
  for uniform scale — branches scale (0.085, 0.9, 0.085) and got 139x the
  displacement.
- **N8AO's `quality` prop** silently overwrites `aoSamples`, `denoiseSamples` and
  `denoiseRadius` in a second layout effect. Set them by hand.
- **The `prng` LCG needs `Math.imul`** — `seed * 1103515245` overflows 2^53 and
  falls into a short orbit (692 distinct values in 5000 draws).
- **One stream per PLANT, never per planter** — sharing one `rnd` between
  `pushTree` and `pushBed` made any tree edit re-randomise every bed. It's
  `bedRnd` (7331) and `treeRnd` (8101). `flowerPink` is a SHARED bucket so its
  array order still moves; the bed's own rows don't.

## Conventions

- `P` (store.js) is mutable per-frame state read inside `useFrame`. Never React
  state.
- The stable Skatepark parent bakes its world matrix once; content-keyed table
  subtrees use normal child updates when a commit replaces one.
- Zero allocation inside the frame loop. Fixed-size pools, round-robin, reuse
  module-scope temporaries.
- Target 60fps at 1600x1000 (currently ~120). `quality === 'low'` scales
  expensive work down.
- **A prop's `rot` is a facing, not a normal** — it yaws local +Z to world
  `(sin rot, cos rot)`; a bench's local +Z is the seat front.
- Comments explain **why**, with the measurement that forced the value.

## Licence

PolyForm Noncommercial 1.0.0 (`LICENSE`); commercial use by separate paid
licence. No copyleft dependencies. `public/{boy,dog_compressed}.glb`,
`public/songs/*` and `ref/*` are carved OUT of the grant. README.md is the
public-facing version of this file.

---
> Source: [MatthewGreenberg/skate-dog](https://github.com/MatthewGreenberg/skate-dog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-30 -->
