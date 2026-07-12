## ptsd-gi-renderer

> Read [NOTES.md](NOTES.md) (terse portable context + roadmap) and

# CLAUDE.md — working conventions for this repo

Read [NOTES.md](NOTES.md) (terse portable context + roadmap) and
[ARCHITECTURE.md](ARCHITECTURE.md) (data layouts, invariants) for all
technical state — this file is only the *how we work* layer that doesn't
belong in those docs.

## The project

Two self-contained single-file WebGL2 HTML apps — no build step, zero
dependencies, shaders inline, written to be read. Keep it that way.

- `ptsd-gi-renderer_8.html` — main app (bake + progressive GI + everything)
- `ptsd-viewer.html` — presentation-only player for baked exports; must stay
  minimal (plain WebGL2, no extensions — it runs on phones and Quest)

## Conventions

- **Bump the in-app `build YYYY-MM-DDx` tag on every renderer change — both
  apps have one** (top of the panel). It's how staleness is diagnosed.
- **Commit only when Doug asks** — he tests first. Don't fold unrelated
  working-tree changes into a requested commit without saying so.
- The repo is deployed on **GitHub Pages** (main branch root:
  https://dougmcnabb.github.io/ptsd-gi-renderer). Its CDN caches ~10 minutes
  per edge (`max-age=600`), so "the fix isn't live" right after a push is
  usually edge lag — check the build tag before debugging.
- **file:// pages can't `fetch()`** — the viewer's scene picker (and anything
  else fetch-based) only works served over http(s): the Pages URL, or the
  local server below.
- Media samples live in `media/`; zips holding a baked geo+light pair also
  get an entry in `media/index.json` (the scene-picker manifest). A light
  file pairs with a geo file only when their `geoId` hashes match — verify
  before committing a pair.
- Doug is a graphics veteran: discuss at rendering-engineer depth, and for
  design questions give options + a recommendation before writing code. He
  drives by *look* — verify numerically first, but expect look-based feedback
  ("too bright") to often indicate a real physics bug; chase the math before
  reaching for knobs.

## Self-verification (agent workflow)

`tools/serve.ps1` is a dependency-free PowerShell static server for the repo
(port 8377); `.claude/launch.json` wires it to the preview browser
(`preview_start "ptsd"`). Proven workflow:

- **Test in an isolated iframe** appended to the preview page, and remove it
  afterwards — Doug uses the same preview page interactively; never disturb
  or repurpose his session state.
- **After touching any shared renderer/viewer codec function, run
  `tools/parity-check.html`** (serve, press Run) — it fails on undocumented
  divergence between the two apps' duplicated functions. New intentional
  divergences get an allowlist entry WITH a reason.
- **New lighting-relevant UI controls wire through the invalidation helpers**
  (`invalidateTransport/SunLive/SunCommitted/Albedo/Resolve`), never raw
  dirty flags — see ARCHITECTURE §7 for the dependency rules.
- **Docs discipline:** ARCHITECTURE/NOTES describe the CURRENT generation
  only; finished or retired designs move to HISTORY.md (roadmap item numbers
  stay stable). Stale docs generate wrong patches — that is how the retired
  depth-map GI text nearly outlived the path tracer by three days.
- **GPU driver resets can be ADAPTER-WIDE.** On fragile drivers (see the
  loop-condition lesson) a heavy shader compile in the agent's embedded
  browser may reset the device under Doug's own Chrome — whose
  `exit_on_context_lost` workaround then kills his GPU process outright
  (suspected contributor on 2026-07-11, never strictly proven; the rule
  stands regardless). While Doug is actively testing on the same machine, do
  NOT run verifications that compile the BVH-family shaders; batch them for
  idle windows and say when they're running. Also: "compiles in the embedded
  browser" does NOT clear a shader for Doug's Chrome — each browser ships
  its own ANGLE + d3dcompiler, so the driver sees different DXBC from
  identical GLSL.
- **Never install a `requestAnimationFrame` shim** (e.g. setTimeout-based).
  The preview tab is hidden so rAF is paused, but a shim persists into the
  interactive session and stacks render loops (the "1000 fps but choppy"
  incident). To drive frames from eval, per frame do:
  `lastAccepted = -1e9; render(performance.now());`
- Top-level `const`/`let` aren't on `window`, but indirect eval reaches the
  global lexical scope — `iframe.contentWindow.eval('scene')` works.
- `gl.finish()` doesn't block in Chrome — use a 1-pixel `readPixels` to sync
  before timing.
- Read render targets with `readPixels` on their FBOs for numeric checks
  (payload is RGBA8; accumulators/pT are float — `gl.FLOAT` reads).
- Big test meshes: copy to a `_tmp_*` name in the repo dir to serve them, and
  delete before committing.

## GPU/GLSL lessons that will bite again

- `precision highp int` is REQUIRED in every fragment shader — GLSL ES
  defaults fragment int to mediump, and mobile GPUs (Quest's Adreno) honor
  16 bits, silently wrapping payload addresses. Desktop hides the bug.
- Payload addressing is exact int32 (`RGBA32I`/`isampler2D`/`texelFetch`,
  `flat` varyings). Never carry addresses in interpolated floats — perspective
  interpolation rounds at fp32 ULP and flips texels.
- Shader-string constants must be defined before the template literals that
  concatenate them (TDZ kills the whole script block silently — the iframe
  error listener won't catch it because navigation replaces the window).
- WebGL2 fragment shaders max out at 16 sampler bindings — count before
  adding a uniform sampler.
- **No `&&`/`||` in GLSL LOOP CONDITIONS** — the confirmed root cause of the
  2026-07-11 crash saga (Doug's Ice Lake Iris Plus, Intel 31.0.101.2130 — the
  FINAL Gen11 driver, no update exists — under Chrome/Edge 150).
  Chromium-150-era ANGLE rigorously expands short-circuit operators (HLSL's
  `&&` doesn't short-circuit); in a loop condition that rewrite produces an
  opaque condition-variable loop (`for(;bool;)` with the condition manually
  re-evaluated at every continue/tail) that the old Intel D3D compiler dies
  on AT LINK: empty info logs, usually device removal, and — because Chrome
  flags this driver `exit_on_context_lost` — the whole GPU process exits.
  Diagnosed by translate-only HLSL capture (`compileShader` runs only
  ANGLE's translation; the driver compiles at LINK — so capture is
  crash-safe) and A/B-confirmed: probe [8] (single-clause conditions, extra
  clause moved to a `break`) links where the byte-identical [7] dies. The rt
  chunk's traversal loops are written that way now — keep any new hot loop
  single-clause. Secondary insurance, unconfirmed but free: hop loops around
  traversal calls use uniform bounds (`uMaxHops`/`uHops`) so FXC can't
  unroll-clone traversal bodies.
- **A crashed GPU process poisons the whole browser session.** After a
  device-removal on this driver, Chrome's replacement GPU process fails
  ~every nontrivial compile with empty logs until the browser FULLY restarts
  (all windows). Corollaries: diagnostic verdicts from post-crash sessions
  are noise (this laundered several wrong theories — "compile-storm budget",
  "one-traversal ceiling" — before the real fix); empty-log failure followed
  seconds later by CONTEXT_LOST is ONE crash reported lazily, not two
  events; the app boots a trivial CANARY program first and, if it fails,
  says "restart the browser fully" instead of cascading.
- **Weak-GPU resilience ladder** (all in the app, keep it working): named
  compiles with full diagnostics → canary → heavy programs deferred
  (`ensureHeavy`) behind a crash-loop breaker (`ptsdGpuBans`, armed before
  each risky compile) → BVH-family cohort suspension after one crash →
  `ptsdWeakGpu` marker (survives `ptsdRetryGPU`; `ptsdResetGPU` wipes) puts
  GI-lite in the FIRST compile slot, defaults reflections off (re-enabling
  is user consent), and never auto-retries the full tracer → gi-accum falls
  back full → lite (single trace, cache terminal — converges to the same GI
  via cache iteration) → disabled-with-notice. One failed program must never
  take the app down. `tools/gpu-probe.html` is the crash-resumable bisector
  + HLSL capturer — extend its ladder when accumFS changes materially.

---
> Source: [dougmcnabb/ptsd-gi-renderer](https://github.com/dougmcnabb/ptsd-gi-renderer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
