## dither-blur-carousel

> Notes for AI agents (and people) working on this repo. Read

# AGENTS.md

Notes for AI agents (and people) working on this repo. Read
[README.md](README.md) first for what the thing is and how the passes fit
together — this file is about the traps.

## Ground rules

- **Every tunable goes in [`gl/config.js`](gl/config.js) and gets a control in
  [`gl/gui.js`](gl/gui.js).** No magic numbers in shaders. If you add a uniform,
  it needs a config key, a `post.js` entry, a push in the `tick` loop (or an
  `onChange` handler for colours), and a slider. Miss one of those and it
  silently does nothing.
- **Comments explain *why*, not *what*.** The code says what it does. Most
  comments here record a decision and the alternative that was tried and didn't
  work. Don't add comments that restate the line under them.
- **Don't reformat or "tidy" code you aren't changing.** Match the surrounding
  style.

## Verify shader changes — the build won't

`next build` and `eslint` only see JavaScript. Shaders live inside template
literals, so a GLSL error passes both and fails at runtime as a black screen.

Two ways to break the build that look fine to the linter:

1. A **backtick inside a shader comment** ends the template literal. Write
   `` `result` `` in a GLSL comment and you get a parse error a hundred lines
   away from the real cause.
2. Any GLSL syntax or type error at all.

Compile them for real before claiming a change works. The shaders import
cleanly in plain Node (no bundler needed), so:

```js
// scratch/shadertest.mjs — writes an HTML page that compiles each shader
import { compositeFragment, streakFragment, fullscreenVertex } from "./gl/shaders/composite.js";
import { trailFragment } from "./gl/shaders/trail.js";
```

Render that page in any browser with a WebGL2 context, call `gl.compileShader`
on each source with `precision highp float;` prepended, and check
`COMPILE_STATUS` plus `LINK_STATUS`. Headless Chrome works:

```bash
"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --enable-unsafe-swiftshader \
  --use-gl=angle --use-angle=swiftshader \
  --virtual-time-budget=5000 --dump-dom "file://$PWD/shadertest.html"
```

Note ShaderMaterial prepends `precision` and any `defines` itself, so a
standalone test has to add them by hand.

## Cross-check the wiring

A uniform declared in the shader but missing from `post.js` renders as zero
rather than erroring. A config key nothing reads sits dead in the panel. Both
are easy to grep for:

```bash
# uniforms declared in the composite that post.js never provides
node -e '
const fs=require("fs");
const comp=fs.readFileSync("gl/shaders/composite.js","utf8");
const post=fs.readFileSync("gl/post.js","utf8");
const decl=[...comp.matchAll(/uniform\s+\w+\s+(u\w+)\s*;/g)].map(m=>m[1]);
console.log(decl.filter(u=>!new RegExp("\\b"+u+"\\s*:").test(post)));'
```

## Colour space

This bites constantly. The rules:

- The **scene pass works in linear light**. Card textures are
  `SRGBColorSpace`, `uBackground` there is a `THREE.Color`.
- The **composite writes straight to the default framebuffer** with no automatic
  output conversion, so it calls `linearToSRGB` itself and everything after that
  is display space.
- **Dithering must happen after that conversion.** Quantizing in linear light
  puts every visible step down in the shadows.
- Palette colours therefore have to arrive **already in display space**.
  `THREE.Color` would convert hex to linear on the way in, which is wrong here —
  that's what [`gl/color.js`](gl/color.js) exists for. Use `hexToSRGB`, not
  `new THREE.Color()`, for anything the composite mixes in.

## Things that look like bugs but aren't

- **The hovered card streaks a ghost of itself.** Its own blur is suppressed,
  but its light still feeds the blur chain. Deliberate. Holding it out properly
  needs a premultiplied weight carried through the whole chain; substituting a
  colour instead punches a hard-edged hole. This was built and reverted on
  purpose.
- **`maxLevels` and `minLevels` are equal.** Intentional — the palette stays put
  as the dissolve deepens and only its coverage grows.
- **`ink` and `paper` are the same colour.** Also intentional. It makes the ramp
  a band; see the README.
- **`trailSpeedRange` (6) sits below `hoverSettleSpeed` (8).** Deliberate, so
  the trail brush is at full size by the time hover intent calls the motion a
  sweep and the two hand off cleanly. Don't "fix" the ordering.
- **Frustum culling is off on every card.** Positions come from the vertex
  shader, so the CPU-side bounding sphere is meaningless.
- **The first moment after load is deliberately empty.** The entry waits for
  every texture to settle before it opens — a card with no map renders black, so
  starting sooner spends the arrival on empty rectangles. A hung request can't
  hold it forever; `ENTRY_WAIT_LIMIT` in `scene.js` opens anyway.
- **The entry reveal is compiled into two passes.** `revealGLSL` goes into both
  the visible card shader and the card buffer, and both `discard`. That's not
  duplication to clean up — the buffer is what hit testing reads, and a card
  that can be clicked before it appears is worse than one that can't be clicked
  after. It also stops the composite printing where nothing is drawn.

## Timing behaviour can't be screenshotted

Several things here are about how something changes over time — the dither
fade-out, the trail clear-out, hover intent during a sweep. A screenshot can't
show any of it, and "looks right" isn't verification.

The easings are plain arithmetic, so replay them in Node instead. `approach()`
in `scene.js`, the idleness ramp in `trail.js` and the pointer-speed smoothing
all reduce to a few lines you can run frame by frame and print a curve from.
That's how the current defaults were chosen.

Exponential easing never reaches zero, which is a recurring source of "it
lingers" bugs — anything gated on a Bayer threshold turns that tail into a
scatter of cells that hang around long after the rest has gone. The fix used in
both places is a `smoothstep` floor (`hoverDitherCutoff`, `trailCutoff`,
`trailIdleDecay`), not a longer ease.

## Performance notes

The composite can do up to four `composed()` calls per fragment — the base plus
one cell-snapped sample for each dither — at five texture fetches each. That's
fine next to the blur chain's ~136, but if you need to claw it back, setting a
dither's cell size to 1 skips its extra sample via a uniform branch.

`uTrailAberration > 0.0` is likewise a uniform branch and costs nothing at its
default of 0.

## Assets

**Don't add images to `public/` without checking their licence**, and don't
present the existing ones as freely reusable — they're third-party placeholders
from Behance. See [CREDITS.md](CREDITS.md). If someone opens an issue asking for
credit or removal, that request is legitimate and should be actioned.

## Checks before you call something done

```bash
npm run build            # JS only
npx eslint gl components app
# plus the shader compile above, if you touched GLSL
```

---
> Source: [Yousuf-developer/dither-blur-carousel](https://github.com/Yousuf-developer/dither-blur-carousel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
