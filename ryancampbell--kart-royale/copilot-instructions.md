## kart-royale

> A kart racer in Three.js with **zero art assets** — every texture, mesh, material

# Working on this repo

A kart racer in Three.js with **zero art assets** — every texture, mesh, material
and sound is generated in code at load time. Written almost entirely by Claude
agents working in parallel. This file is the handover note: the contract to code
against, the traps that have already cost real rounds, and how to tell whether a
change actually helped.

Read `README.md` for what the project is, and `ART_DIRECTION.md` for what it is
supposed to look like — §9 of that file is the scoring rubric, and it is the bar.

## Commands

```bash
npm run dev      # vite dev server
npm run build    # tsc --noEmit && vite build  — this is the gate, it must pass
npm run preview  # serve the production build
```

Never start a bare `npx vite` for a test harness. Use `tools/vite-server.mjs`,
which spawns the binary directly, kills the whole process group on teardown, and
refuses to adopt a server that is already serving a *different* working tree. An
orphaned dev server from a worktree once silently served stale code into every
measurement for a full round.

## The contract

`src/types.ts` is the interface every subsystem codes against — `ITrack`,
`IKart`, `IItems`, `IRace`, `IInput`, `Ctx`, `Surface`, `SURFACE_PROPS`.

**Do not edit `src/types.ts` as a side effect of another change.** It is what
makes parallel work possible; changing it invalidates everyone else's
assumptions at once. If a change genuinely needs the contract widened, do that
as its own commit and say so.

Rough ownership, one concern per directory:

| path | what lives there |
|---|---|
| `src/render/` | renderer, post chain, procedural textures/materials, sky |
| `src/world/` | track layout + geometry, scenery, foliage, water |
| `src/kart/` | chassis, suspension, tyres, model, liveries |
| `src/game/` | race state, AI, camera, items, projectiles |
| `src/fx/` | particles, trails, decals |
| `src/ui/` | HUD, menus, minimap |
| `src/audio/` | synthesis, music, engine |
| `src/core/` | input, settings, diagnostics, prewarm, event bus |

## Traps that have already cost a round

These are not style preferences. Each one is a bug that shipped, got measured,
and got fixed. The comment explaining each is load-bearing — if you remove the
guard, you reintroduce the bug.

**MSAA is incompatible with the AO pass.** `N8AOPostPass` samples the composer's
input buffer as a *texture*, which a multisampled target cannot resolve to.
`multisampling = 2` produced black frames on 7.6% of frames; `0` produced 0.2%.
`Renderer.ts` returns `0` samples whenever SSAO is on. Do not "optimise" this
back on.

**Steering handedness is corrected once, at the input boundary.** This chassis
uses `forward = (sin yaw, 0, cos yaw)`, so a rising yaw turns *left*, while the
input contract says `steer > 0` means the player wants to go *right*.
`Kart.ts` negates at the boundary and `Race.ts` negates the AI's command at its
call site, because the AI solves in the chassis frame. These two negations are
correct together. Flipping only one inverts steering for the player or sends
every AI kart into a wall. There is a probe: `tools/steer-test.mjs`.

**`DRIFT_CARRY_TIME` already exists** in `Kart.ts`. It preserves the mini-turbo
clock across a brief dip in slide angle so that a corner exit does not reset the
charge. Several agents have "discovered" this problem and added a second,
competing carry window. Check before adding one.

**A single global constant can defeat thousands of lines of work.**
`envMapIntensity` set flat across all materials silently deleted every metal
reflection and clearcoat highlight in the game. It is now per-material via
`envFor()` in `Liveries.ts`. Be suspicious of any one value applied uniformly to
things that are not uniform.

**Reading the presented frame requires `preserveDrawingBuffer`.** Without it,
`readPixels` after present returns discarded contents — which reads as "100% of
frames are black" and is completely false. `FrameWatch` now refuses to start
unless the context was created with it (`?debug=frames` sets it). This cost a
day of chasing a bug that did not exist.

**Shader pre-warm must bind a render target.** `WebGLPrograms.getParameters()`
reads `outputColorSpace` and `toneMapping` off the *currently bound* target, so
`renderer.compile()` with nothing bound compiles default-framebuffer variants
that the composer never uses, and every real shader still stalls on first use.
`Prewarm.ts` binds a 1×1 scratch target while compiling.

**Workflow scripts must use a literal `ROOT`.** `tools/*.js` run in a sandbox
with no `process` global — reading `process.env` throws on line 1 and kills the
whole run. Every one of them has `const ROOT = '/Users/ryan/dev/personal/kart-game'`
with a comment saying why. A well-meaning "make this portable" commit broke all
of them at once.

**Worktrees need a `node_modules` symlink** or the `tools/*.mjs` harnesses will
not start.

## Verifying a change

The harnesses are the point of this repo more than the game is. Each one answers
a question that is genuinely hard to answer by looking.

| harness | the question it answers |
|---|---|
| `drift-bench.mjs` | Is a drifting lap actually faster than a clean one, and does the mini-turbo ladder pay out? |
| `tear-hunt.mjs` | Are any presented frames torn or partially rendered? |
| `hitch-check.mjs` | Where are the frame-time spikes? |
| `camera-probe.mjs` | How much does the camera swing, in °/s? |
| `mobile-soak.mjs` | Does it survive a phone without a memory kill? |
| `context-loss-test.mjs` | Does it recover from a lost WebGL context? |
| `steer-test.mjs` | Does the kart go where the input says? |
| `ai-health.mjs` | How often does the AI touch a wall? |
| `touch-test.mjs`, `touch-lazy-test.mjs` | Do touch controls mount and steer, including when the browser lies about being a desktop? |
| `autoplay.mjs`, `shot.mjs` | Play a race unattended; capture frames. |

Rules learned the hard way:

- **Validate the instrument before trusting the reading.** Two harnesses here
  produced confident, precise, entirely fictional numbers before anyone checked
  them — one projected a camera-relative vector as if it were a world point, and
  one had a frame-rate knob wired to nothing at all, which manufactured a "17×
  frame-rate dependence" that did not exist.
- **A screenshot cannot find a gameplay bug.** Inverted steering, unusable mobile
  controls and a pause menu that permanently ended your race all survived three
  full rounds of six reviewers scoring beautiful stills. If a change affects how
  the game *plays*, it needs a harness or a human, not a critic looking at a PNG.
- **Draw calls are not proof of rendering.** All five known black-screen failure
  modes still issue a full frame's worth of draws into a canvas that is never
  painted. `Diagnostics.ts` samples the luma spread of the presented image
  instead; trust that, not the counter.

Before calling anything done: `npm run build` must pass, and the harness that
covers the thing you touched must pass.

## Style

Match the surrounding code — it is fairly heavily commented, and deliberately so.
Comments here explain *why*, and more than once a correct-looking cleanup that
deleted a "wrong" comment also deleted the guard it justified and reintroduced a
7.6%-of-frames rendering bug. If a comment seems wrong, the guard may still be
right for a different reason. Work out which before removing either.

---
> Source: [ryancampbell/kart-royale](https://github.com/ryancampbell/kart-royale) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
