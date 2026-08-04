## plasma-flipclock

> Context and rules for AI agents working in this repo. Read this before editing.

# AGENTS.md

Context and rules for AI agents working in this repo. Read this before editing.

## What this is

A KDE Plasma 6 plasmoid inspired by a classic skeuomorphic flip clock. The visual
design is rebuilt from QML geometry and gradients.

Target platform is Plasma 6.7 / Qt 6.11 on Arch. `README.md` is the user-facing
doc; this file is the agent-facing one.

## Hard rules

1. **Never redistribute reference assets.** `reference/` is local and gitignored.
   Do not copy its PNGs into `package/`, embed them as data URIs, or commit them.
   Measuring them to derive a colour, coordinate, or gradient stop is fine.
2. **Never trace the original glyph outlines.** Shipping traced letterforms is
   shipping the typeface. Digits use a bundled OFL font chosen by measurement
   (`tools/fontscore.py`). If you change fonts, re-run that tool and update the
   calibration constants in `Style.js`.
3. **`FlipClock.qml` must not import anything from `org.kde.plasma`.** The render
   harness loads it under bare `qml6`; a Plasma import breaks pixel-diffing. All
   Plasma coupling lives in `main.qml`.
4. **Do not set `clip: true` anywhere in the card hierarchy.** `RectangularGlow`
   deliberately draws outside its own bounds, so any clipping ancestor silently
   eats the card shadows. The digits are cropped at the crease by the
   `ShaderEffectSource` texture bound instead.
5. **Measure, don't eyeball.** Every geometric or colour change should be checked
   with `tools/pixdiff.py`. Claims like "looks right" are not acceptable evidence
   in this repo — the whole point is measured fidelity.

## Architecture

```
package/contents/ui/
  main.qml          Plasma wrapper: org.kde.plasma.clock source + config bindings
  FlipClock.qml     the whole 996x566 scene; plain QtQuick, no Plasma imports
  FlipCard.qml      one card: shadow stack, 4 flip layers, perspective, animation
  CardFace.qml      one flap: stacked-deck bezel, face gradient, crease shading
  GlossyDigits.qml  digits, gradient-filled via OpacityMask, cropped to one flap
  Style.js          .pragma library — every measured constant, in reference units
tools/
  snap.sh, Snap.qml   render a component to PNG at an exact size
  pixdiff.py          rebuild the reference frame and diff against a render
  fontscore.py        score candidate digit fonts against the measured glyphs
  probes/             fixed-state scenes for the harness (not shipped)
```

### Coordinate system

Everything is in **reference units**: the 996x566 design space.
Consumers scale by `u = min(width/996, height/566)`.

**Never wrap a fixed-size item in `Item { scale: }`** — that rasterises text once
at base size and then magnifies it. Derive every dimension from `u` instead.

`Style.js` holds scalars (geometry, timings, font calibration). Gradient stops
live as literal `GradientStop`s in the QML that paints them — paint belongs with
the painting, and it keeps one source of truth per value.

### What the original actually does

Not the usual flip-clock design, and easy to get wrong:

- **Two cards, not four digits.** Both hour digits are on one panel, both minute
  digits on another. The hour card flips on the hour; the minute card flips on
  the minute.
- **Four layers per card**: static upper = NEW, static lower = OLD, falling upper
  = OLD rotating about its bottom edge, arriving lower = NEW about its top edge.
- **Two-phase flip, `Easing.OutBounce` on both phases.** 1000 ms hours,
  800 ms minutes; the second flap is held at +90° through phase one.
- **The digit gloss restarts at the crease** so each half is lit independently.
  This is the detail that makes it read as a physical card — do not "simplify" it
  into one continuous gradient.
- The date and alarm strips on the lower third of each card do **not** flip.

## Traps that will silently waste your time

These are all verified on this machine, and each one cost real debugging.

- **`QT_QPA_PLATFORM=offscreen` cannot render this.** It runs, exits 0 and writes
  a valid PNG — but every `ShaderEffect` renders *nothing*, so the gradient-filled
  digits and all card shadows silently vanish and a diff looks catastrophically
  wrong for entirely the wrong reason. `tools/snap.sh` renders on the live
  session. The software backend is worse: no `ShaderEffect` implementation at all.
  Never "fix" a blank render by switching backends.
- **`ffmpeg -pix_fmt rgb24` drops alpha, it does not composite it.** Comparing a
  translucent render against a flattened reference this way produces completely
  bogus numbers. Flatten explicitly over a known background, as `pixdiff.py` does.
- **`MultiEffect { maskSource }` destroys text antialiasing.** It runs the mask
  through a `smoothstep` band, hard-stepping glyph edges. Use Qt5Compat
  `OpacityMask`, which is an exact `As * Am` multiply. (`qt6-5compat` is a hard
  dependency of `plasma-workspace`, so it is always present.)
- **A `GradientStop`'s `parent` is the `Gradient`, not the `Rectangle`.**
  `position: parent.someProperty` silently evaluates to `undefined` → 0 and the
  gradient collapses. Address the delegate through its `id`.
- **`font.pixelSize` is an `int`.** `270 * u` truncates; use `Math.round()`.
- **Qt5Compat effects sample source and mask in the *effect's* normalised
  coordinates.** The effect, its `source` and its `maskSource` must all be
  `anchors.fill` of the same parent or the mask stretches silently.
- **Qt Quick has no back-face culling.** A flap past ±90° renders mirrored. Gate
  `visible` on the angle. `OutBounce` never overshoots, so the current curves are
  safe — but swapping to `OutBack`/`OutElastic` would break this immediately.
- **The crease must land on a whole device pixel**, and both halves must use the
  same value, or the glyph gains or loses a row where they meet. See
  `FlipCard.creaseY`.
- **In `tools/fontscore.py`, never build a `Pango.FontDescription` from a family
  string.** Pango parses trailing style words, so `"Roboto Condensed"` becomes
  family `Roboto` + stretch `Condensed` and silently falls back to another font.
  Use `set_family()`. Six different families scored identically before this was
  caught.

## Plasma 6.7 specifics

- Time comes from **`org.kde.plasma.clock`** (`import org.kde.plasma.clock`,
  versionless). Not `Plasma5Support.DataSource`, not a `Timer`. `trackSeconds:
  false` gives minute-aligned updates handled by the C++ side — which is what a
  flip clock wants, since the flip must fire *on* the boundary.
- Root type is `PlasmoidItem` from a versionless `import org.kde.plasma.plasmoid`.
- Every QML file starts with `pragma ComponentBehavior: Bound`.
- `metadata.json` needs `KPackageStructure`, `KPlugin{Id,...}` and
  `X-Plasma-API-Minimum-Version: "6.0"`. Do **not** add `ServiceTypes`,
  `X-Plasma-API` or `X-Plasma-MainScript` — all legacy, absent from every modern
  installed package.
- Config aliases must be `cfg_<entryName>` matching `config/main.xml` exactly,
  case-sensitively.
- Use `u` for the artwork and `Kirigami.Units.*` only in the config UI. They are
  different scales; mixing them will look wrong.

## Workflow

```bash
export PATH=/usr/lib/qt6/bin:$PATH        # Arch does not symlink qmllint

qmllint -I /usr/lib/qt6/qml -I package/contents/ui package/contents/ui/*.qml
plasmoidviewer -a io.github.pruefsumme.flipclock -f planar -l desktop -s 996x566
kpackagetool6 --type Plasma/Applet --upgrade ./package

tools/snap.sh DiffProbe.qml 996 566 tools/out/render.png
tools/pixdiff.py tools/out/render.png --write-diff tools/out/diff.png
```

`qmllint` reports "unqualified access" warnings for `Repeater` delegates; adding
`pragma ComponentBehavior: Bound` and addressing the delegate by `id` clears them.

### Reading the diff

`pixdiff.py` reports mean absolute error per region on 0–255. Current baseline:

| region | MAE |
|---|---|
| whole frame | 13.6 |
| card frames (digits excluded) | 7.2 / 7.4 |
| panel margin / gap / below | 2.7 / 3.3 / 3.8 |
| digits (not gated) | 26.4 / 24.3 |

The `--gate 8.0` default is a **regression guard set just above the current
baseline, not a claim of pixel-perfection**. Do not raise it to make a change
pass; if a change pushes a region over, that is the tool doing its job. Two
residuals are known and expected: the digits (font substitution) and a 1–2px
bezel ring around each card.

`--write-diff` amplifies differences 6x, so it always looks worse than reality —
read the numbers, not the picture.

## Status

Built and working: the clock, the flip, the panel, the date/alarm strips,
configuration, and the measurement tooling.

Not built: the weather readout and its animated sprites. Panel geometry and all
six text positions are already measured and in `Style.js`; five Plasma weather
providers (`bbcukmet`, `dwd`, `envcan`, `noaa`, `wettercom`) are installed and can
drive it via `Plasma5Support.DataSource { engine: "weather" }` — that legacy
engine is the only weather API, since the Plasma weather applet is compiled C++.
Map the engine's `Condition Icon` field (freedesktop names) rather than parsing
condition prose.

The weather sprites are painterly raster art, so they need a separate illustration
workflow rather than measurement. Be honest about that distinction; do not describe
them as a 1:1 recreation.

## Related

Reference assets and analysis helpers are local-only and are not part of the
package or distribution.

---
> Source: [pruefsumme/plasma-flipclock](https://github.com/pruefsumme/plasma-flipclock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-31 -->
