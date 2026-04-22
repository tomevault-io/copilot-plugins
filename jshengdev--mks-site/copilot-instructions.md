## mks-site

> **Every session must end by updating this file.** If you learned something new — a preference, a pattern that works, a mistake to avoid, a decision made — append it to the relevant section below. If a section doesn't exist, create one. This file is the living memory of the project. Treat it as the source of truth that compounds over time.

# MKS Site — CLAUDE.md

## Self-Updating Rule

**Every session must end by updating this file.** If you learned something new — a preference, a pattern that works, a mistake to avoid, a decision made — append it to the relevant section below. If a section doesn't exist, create one. This file is the living memory of the project. Treat it as the source of truth that compounds over time.

When updating:
- Add to `## Learnings Log` at the bottom with the date and what was learned
- If the learning is a durable rule, also add it to the appropriate section above
- If a previous entry is wrong or outdated, correct it in place
- Keep entries concise — one line per learning when possible

---

## What This Is

An immersive music experience for composer **Michael Kim-Sheng**. Five distinct 3D worlds, each tied to a composition and an emotional state from real listener descriptions. The music IS the router — selecting a track in MiniPlayer enters that track's world. Each world is a scroll-driven WebGL scene where the user moves a camera along a spline path through a unique landscape. A Canvas 2D entry page serves as the threshold before WebGL loads.

## Tech Stack

- **React 19** + **Vite 7** — no Next.js, no SSR, pure SPA
- **Three.js** (vanilla, NOT React Three Fiber) — all 3D rendering
- **Lenis** — smooth scroll, exposes progress (0→1) + velocity
- **pmndrs/postprocessing** — bloom, chromatic aberration, vignette, film grain, SSAO, DOF, motion blur
- **Raw GLSL shaders** — adapted from GitHub reference repos (Nitash-Biswas, al-ro, James-Smyth, Alex-DG, daniel-ilett)
- **Vanilla CSS** — no Tailwind, no CSS-in-JS
- **Web Audio API** — AnalyserNode for MiniPlayer visualizer

### Key Dependencies
```
three, lenis, postprocessing, react-router-dom
```

## Architecture

### Active Branch: `feature/environment-worlds` (from `v0.1.0-meadow-stable`)

```
src/
  App.jsx                    — Music-as-router shell: MiniPlayer + WorldContext + react-router-dom
  EnvironmentScene.jsx       — React mount point for config-driven WorldEngine
  index.css                  — Global reset + Lenis CSS
  useScrollAudio.js          — Hook: ramps audio volume from Lenis scroll progress
  useWorldTransition.js      — Portal transition hook (FBO-based GLSL dissolves)
  DevTuner.jsx + .css        — Live parameter panel (backtick to toggle, freeze/live atmosphere)

  entry/                     — Canvas 2D entry page (zero Three.js weight)
    EntryPage.jsx            — Dithered botanical flower, cursor parallax, audio context gate

  environments/              — World configs (pure data, no Three.js imports)
    golden-meadow.js         — Landing world (/) — rolling hills, golden hour, "Innocent Awakening"
    ocean-cliff.js           — /listen — cliff geometry, stylized water, dusk DOF, "Person Against Infinity"
    night-meadow.js          — /story — same terrain at night, stars, 800 fireflies, "Small But Not Alone"
    storm-field.js           — /witness — sharp terrain, rain, lightning, "Searching Through Chaos"
    ghibli-painterly.js      — /collect — cel-shading, Kuwahara, hyper-vivid, "Never Felt Brighter"
    index.js                 — Environment registry

  meadow/                    — Three.js engine (vanilla, class-based)
    WorldEngine.js           — Config-driven orchestrator (accepts environment config objects)
    MeadowEngine.js          — Backward-compat wrapper around WorldEngine
    constants.js             — Shared constants (SECTION_T_VALUES)
    ScrollEngine.js          — Lenis wrapper, exposes progress + velocity
    CameraRig.js             — CatmullRomCurve3 S-curve spline + damped lerp + terrain follow
    TierDetection.js         — Performance tier detection (1=desktop, 2=laptop, 3=mobile)
    MeadowScene.js           — Sky dome (Preetham), fog, directional + ambient light
    TerrainPlane.js          — Configurable terrain (sin/cos hills, cliff+ocean, diamond-square)
    CloudShadows.js          — Multiply-blended shadow plane with glacial UV drift
    GrassGeometry.js         — Blade mesh generator (7-segment high LOD, 1-segment low)
    GrassChunkManager.js     — Chunk pool (20x20 units), activate/dispose/fade-in, LOD swap
    FlowerInstances.js       — 6 color types x ~133 each, clearing avoidance, toon shader
    FireflySystem.js         — Configurable count (400 optimal), additive blending, vertical bob
    AtmosphereController.js  — 5-keyframe scroll-driven interpolation (sky, grass, fog, post-FX)
    CursorInteraction.js     — Mouse-to-world raycast (y=0 plane), lerped worldPos + smoothed velocity
    PostProcessingStack.js   — EffectComposer: bloom, CA, vignette, grain, SSAO, DOF, fog, color grade
    GodRayPass.js            — Screen-space radial blur (GPU Gems 3), half-res FBO
    ScoreSheetCloth.js       — Wind-driven score sheets tumbling through meadow
    ClothSolver.js           — Verlet integration cloth physics (used by ScoreSheetCloth)
    ArtistFigure.js          — 2D cutout billboard (needs seated/walking variants)
    PortalHint.js            — Shimmering spots teasing future worlds
    DustMotes.js             — Floating particles catching sunlight
    MusicTrigger.js          — BotW discovery moment at scroll threshold
    AudioReactive.js         — FFT analysis for music-driven effects

    shaders/
      grass.vert.glsl        — 4-layer wind + cursor wind brush + wave wind, billboard, fake normals
      grass.frag.glsl        — Translucent lighting (al-ro), iquilez fog, cloud shadows
      firefly.vert.glsl      — Point particle vertical bob (Alex-DG)
      firefly.frag.glsl      — Inverse-distance radial glow, warm amber
      flower.vert.glsl       — Gentle sway + instanceMatrix
      flower.frag.glsl       — 3-band toon diffuse + rim light (daniel-ilett)
      star.vert.glsl         — Procedural star field (pow(rand,3) sampling)
      dust.vert/frag.glsl    — Dust mote particles
      portal.vert/frag.glsl  — Shimmer portal effect
      score-sheet.vert/frag  — Cloth rendering
      transition.vert/frag   — GLSL dissolve transitions between worlds
      god-ray-blur.frag.glsl — Radial blur pass
      color-grade.frag.glsl  — SEUS-style lift/gamma/gain/split-tone
      fog-depth.frag.glsl    — 3-zone depth fog
      motion-blur.frag.glsl  — Velocity-based motion blur

    effects/ (custom pmndrs Effect subclasses)
      FilmGrainEffect.js, RadialCAEffect.js, MotionBlurEffect.js,
      KuwaharaEffect.js, GodRayCompositeEffect.js, ColorGradeEffect.js,
      FogDepthPass.js, SSAOSetup.js, DOFSetup.js

  content/                   — React DOM overlays (opacity driven by engine, not CSS)
    ContentOverlay.jsx       — Container, registers section DOM refs with engine
    LandingContent.jsx       — Title + subtitle (placeholder)
    MusicContent.jsx         — Glass-card placeholder
    AboutContent.jsx         — Glass-card placeholder
    StoreContent.jsx         — Glass-card placeholder
    FooterContent.jsx        — Copyright
    content-overlay.css      — Fixed overlay styles, glass-card

  MiniPlayer.jsx + .css      — Audio player + world navigation (dispatches route changes on track select)
  MoonlightCursor.jsx        — Custom cursor effect

  assets/
    textures/cloud.jpg       — Perlin FBM noise texture (from al-ro)
    textures/score-sheet.jpg — Score sheet texture
    textures/mks-portrait.jpg — Artist portrait
    audio/In a Field of Silence.mp3 — Golden Meadow composition (4:37, 320kbps)

docs/
  superpowers/specs/         — Design specifications
  superpowers/plans/         — Implementation plans
  webgl-reference/           — 26 reference shader/code files from GitHub repos
  mks-design-philosophy/     — 19+ design documents
```

### Music-as-Router Pattern

Navigation is music-driven, not URL-driven. The MiniPlayer IS the navigation system.

| Route | World | Track | Emotional Atlas |
|-------|-------|-------|-----------------|
| `/` | Golden Meadow | "In a Field of Silence" | Innocent Awakening |
| `/listen` | Ocean Cliff | TBD | Person Against Infinity |
| `/story` | Night Meadow | TBD | Small But Not Alone |
| `/collect` | Ghibli Painterly | TBD | Never Felt Brighter |
| `/witness` | Storm Field | TBD | Searching Through Chaos |

- Selecting a track in MiniPlayer = entering that track's world
- URLs update to reflect current world but are secondary
- Portal transitions (GLSL dissolves) fire when the track changes
- The scroll arc maps to the structure of the current piece

### Entry Page

Before WebGL loads, a Canvas 2D entry page:
- Procedural dithered flower (botanical specimen aesthetic)
- 6-layer cursor parallax on flower parts
- Audio context confirmation gate
- Click Enter → music starts → dissolve into WebGL world
- Zero Three.js weight, instant load

### Environment World Config Schema

Each world is a pure data config in `src/environments/`. The config defines:
- Terrain type and parameters
- Sky/atmosphere settings
- Vegetation (grass density, flower types, colors)
- Particle systems (fireflies, rain, petals, dust, stars)
- Post-processing recipe (bloom, DOF, grain, color grade, Kuwahara)
- Camera path (spline points, speed, height)
- Atmosphere keyframes (5 emotional arc positions)

`WorldEngine.js` reads the config and wires all subsystems accordingly. `MeadowEngine.js` is a backward-compat wrapper that passes the golden-meadow config.

### How the Render Loop Works
1. Lenis updates `scrollEngine.progress` (0→1)
2. CameraRig lerps camera position along CatmullRom spline, offset by terrain height
3. AtmosphereController interpolates 5 keyframes (STILLNESS→AWAKENING→ALIVE→DEEPENING→QUIETING), pushes to all subsystems (skipped when `paused` flag set by DevTuner)
4. Subsystems update: cloud shadows drift, grass wind animates, fireflies bob, flowers sway, cloth physics, cursor wind
5. GodRayPass renders occlusion to half-res FBO
6. PostProcessingStack renders via EffectComposer (bloom, CA, vignette, grain, SSAO, DOF, fog, color grade, kuwahara, god ray composite)

### Content Visibility Formula
```
dist = |cameraT - sectionT|
opacity = 1.0 - smoothstep(0.03, 0.08, dist)
pointerEvents = opacity > 0.1 ? 'auto' : 'none'
```

### Performance Tiers
| Tier | Criteria | Grass | FX |
|------|----------|-------|-----|
| 1 (Desktop) | >1366px, >4 cores, maxTex>4096 | 60K (6 chunks) | Full |
| 2 (Laptop) | 769-1366px or ≤4 cores | 18K (4 chunks) | Reduced |
| 3 (Mobile) | ≤768px or no WebGL2 | 0 | Static fallback |

## DevTuner

Toggle with backtick (`). **Freeze/Live button** pauses AtmosphereController so slider changes stick. `data-lenis-prevent` on the scroll area prevents Lenis from hijacking scroll inside the panel.

When AtmosphereController is NOT frozen, it overwrites all subsystem values every frame from keyframe interpolation. This is by design — the scroll drives the atmosphere. Freeze to tune individual values.

### DevTuner Wiring Gotchas
- **Exposure** goes to `colorGrade.uniforms.uExposure` (pre-grade multiplier), NOT `renderer.toneMappingExposure` (which does nothing when toneMapping=NoToneMapping)
- **FOV** must set `cameraRig.baseFov` + `cameraRig.currentFov`, NOT `camera.fov` (CameraRig overwrites camera.fov every frame)
- **Firefly/Dust brightness** must also toggle `points.visible` — atmosphere sets visible=false at STILLNESS
- **SSAO radius** — use `effect.radius` getter/setter, not `ssaoMaterial.uniforms.radius.value` (pmndrs API varies by version)

## Design First Principles

These are non-negotiable. Every decision filters through them.

### YES — What We Want
- **Cinematic pacing** — everything reveals slowly, nothing pops in
- **Dark dominance** — 85% dark, 15% light. Black is not absence, it's atmosphere
- **Surfaces, not pops** — opacity transitions only. No scale/bounce/slide entrances
- **Nature as structure** — the meadow IS the site, not decoration
- **Expensive gallery feel** — generous whitespace, single focal point per viewport, restraint
- **Real photography only** — no AI-generated artist images, ever
- **Invisible design** — if you notice the CSS, it failed
- **Earned warmth** — cold is the default. Warmth appears through interaction/engagement
- **Two-voice typography** — serif for titles/artist name (classical), sans-serif for body (modern)
- **Imperfection budget** — slight rotations (1-2deg), soft blur, grain. Never pixel-perfect
- **prefers-reduced-motion** — every animation needs a fallback. Always.

### NO — What We Never Do
- No flat black (`#000`). Use breathing blacks (`--void: #0a0a0a`, `--warm-black: #1a1208`)
- No neon colors. Cool luminance only (`--text-primary: #c8d4e8`)
- No grid layouts for products. Each album gets its own cinematic world
- No "epic" (too Marvel), "vibes" (too casual), "content" (these are works), "minimal" (reducing to trend)
- No spectacle. Craft over flash
- No AI-generated images of the artist
- No transform-based entrance animations (scale, translateY). Opacity only for surfacing

### Color System
| Token | Hex | Usage |
|-------|-----|-------|
| `--void` | `#0a0a0a` | Primary background |
| `--warm-black` | `#1a1208` | Warm section backgrounds |
| `--text-primary` | `#c8d4e8` | Body text (cool luminance) |
| `--text-secondary` | `#90a0a0` | Secondary text |
| `--teal` | `#4a6a68` | Audience color — the viewer's emotional space |
| `--amber` | `#d4c968` | Artist color — warmth, presence, earned moments |
| `--red-felt` | `#983028` | Used exactly ONCE in the entire site |

## Reference Code Sources

All shader code is adapted from real GitHub repos. Reference files live in `docs/webgl-reference/`.

| Author | Repo | What We Stole |
|--------|------|--------------|
| Nitash-Biswas | grass-shader-glsl | 4-layer wind deform(), billboard rotation, fake curved normals |
| al-ro | grass (WebGL) | ACES tonemapping, iquilez fog, translucent lighting, quaternion blade bend |
| James-Smyth | BotW grass | Cloud shadow UV scrolling, vertex color wind weights |
| Alex-DG | vite-three-webxr-flowers | FirefliesMaterial, additive blending, vertical bob particles |
| daniel-ilett/maya-ndljk | toon shader | Step-function toon lighting (3-band + rim) |
| spacejack | terra | FOG_COLOR, GRASS_COLOR, scene orchestration constants |

**Rule:** All shader code must be stolen from real repos and adapted. No original GLSL from scratch.

## Orchestration System

- **Location:** `/Users/johnnysheng/mks/orchestration/`
- **Meadow orchestrator:** `orchestration/meadow-orchestrator.sh`
- **Commands:** `init`, `launch`, `status`, `monitor`, `merge`, `build`, `cleanup`
- **Worker structure:** `orchestration/meadow-workers/wN-name/{CLAUDE.md, TASK.md, CORRECTIONS.md, output/}`
- **Window status files:** `orchestration/window-N-status.md` — each parallel window writes status here
- **Session reports:** `orchestration/session-report-YYYY-MM-DD.md`
- Uses `tmux` sessions + `git worktree` for parallel Claude Code workers
- **Critical:** Must `unset CLAUDECODE` before launching nested Claude instances in tmux
- Workers write DONE.md and FILES.md to their output dir when complete
- Workers must NOT modify MeadowEngine.js / WorldEngine.js — integration is post-merge

## AutoResearch Pipeline

- **Location:** `/Users/johnnysheng/mks/research/pipeline/`
- **Methodology:** `program.md` — Karpathy autoresearch pattern adapted for visual/shader research
- **Harness:** `prepare.js` — evaluation framework (FPS, draw calls, emotional atlas scoring)
- **Results:** `results.tsv` — 24 experiments logged
- **Winners:** `winners/` — 12 technique documents with integration instructions
- **Scoring:** Each experiment scored /70 against emotional atlas criteria (beauty, nostalgia, aliveness, intimacy, whimsy, innocence, heartache)

### Per-World Best Scores (as of 2026-03-14)
| World | Score | Key Technique | Experiment |
|-------|-------|--------------|------------|
| Ocean Cliff | 61/70 | DOF v3 (focus=8, range=1.5, bokeh=5.5) + split-tone | exp-022 |
| Night Meadow | 58/70 (62 potential) | 400 fireflies + stars + camera arc at t=0.75 | exp-013 |
| Ghibli Painterly | 56/70 | Atmosphere fix (ambient 0.20→0.45) + cel+Kuwahara | exp-023 |
| Storm Field | 47/70 | Rain + volumetric clouds + lightning | exp-012 |
| Golden Meadow | 45/70 | Wave wind + golden hour atmosphere | exp-002+018 |

## Web Extractor Tool

- **Location:** `/Users/johnnysheng/mks/research/web-extractor/`
- **Usage:** `node src/extract.js <url> [--scroll-positions 50] [--wait 8]`
- Captures: shaders (GLSL), textures (PNG), uniforms, draw calls, scene graphs, scroll behavior
- Requires system Chrome (Playwright bundled Chromium lacks headless WebGL)
- 7 sites extracted, validated against WebGL Aquarium (32 shaders, 93 textures)

## Where to Read More

Design philosophy: `mks-design-philosophy/` — Read `BRAND-ESSENCE.md` and `STYLE-DECISIONS.md` first.

Specs:
- `docs/superpowers/specs/2026-03-12-webgl-meadow-design.md` — original meadow spec
- `docs/superpowers/specs/2026-03-13-environment-worlds-prd.md` — multi-world PRD (596 lines)

Plan: `docs/superpowers/plans/2026-03-12-webgl-meadow.md`

Session reports: `orchestration/session-report-2026-03-14.md` — comprehensive build log

Research: `research/pipeline/` — 24 experiments with scoring, `research/web-extractor/` — site extraction tool

## What's Built vs. What's Next

### Built (Phase 1 + Phase 2A: Meadow Engine) ✓
- Full Three.js meadow engine with 17 subsystems wired
- 60K instanced grass with 4-layer wind + cursor wind brush + wave wind
- Procedural terrain with rolling hills
- Sky dome (Preetham model) with golden hour lighting
- Cloud shadow plane with glacial drift
- 800 instanced flowers with toon shading
- Configurable firefly particles (400 = emotional optimum) with additive blending
- Post-processing: bloom, CA, vignette, grain, SSAO, DOF, 3-zone fog, color grade (SEUS), motion blur, kuwahara painterly, god ray composite
- GodRayPass — screen-space radial blur (GPU Gems 3)
- AtmosphereController — 5-keyframe scroll-driven interpolation with 38 params each
- Cursor interaction — mouse→world raycast, lerped worldPos, smoothed velocity
- Score sheet cloth — Verlet physics, wind-driven
- Artist figure — 2D billboard at far end
- Portal hints — shimmering spots for future worlds
- Dust motes — floating particles
- Music trigger — BotW discovery moment
- Audio reactive — FFT analysis
- Content overlay with 5 sections (DOM-driven opacity from scroll)
- Lenis smooth scroll → CameraRig spline path
- Performance tiers (3 levels) with LOD switching
- Camera terrain-following (prevents clipping into hills)
- DevTuner — live parameter panel with freeze mode
- MiniPlayer + MoonlightCursor preserved

### Built (Phase 3: Environment Worlds — `feature/environment-worlds` branch) ✓
- **Config-driven WorldEngine** — abstracts MeadowEngine into configurable system
- **5 environment world configs** — golden-meadow, ocean-cliff, night-meadow, storm-field, ghibli-painterly
- **Music-as-router** — MiniPlayer dispatches route changes, track = world
- **Entry page** — Canvas 2D dithered flower, cursor parallax, audio context gate
- **Night Meadow** — same terrain at night, procedural star field, 800 fireflies, night atmosphere
- **Ocean Cliff** — cliff geometry + flat ocean plane, stylized water shader, dusk sky, DOF
- **Storm Field** — rain particles, lightning system, urgent wind, dramatic atmosphere
- **Ghibli Painterly** — cel-shading, Kuwahara post-processing, petal particles, hyper-vivid
- **Portal transitions** — FBO-based GLSL dissolves between route changes
- **Per-world terrain algorithms** — rolling hills, cliff+ocean, diamond-square, smooth
- **Audio asset** — "In a Field of Silence" (320kbps) wired to Golden Meadow
- **react-router-dom routing** — `/`, `/listen`, `/story`, `/collect`, `/witness`

### Built (Research Infrastructure)
- **AutoResearch Pipeline** (`/research/pipeline/`) — Karpathy-style autonomous experiment loop
  - 24 experiments completed, 12 winners documented, 3 IMPROVE cycle iterations
  - Best scores: Ocean Cliff 61/70, Night Meadow 58/70 (62 potential), Ghibli 56/70
- **Web Extractor** (`/research/web-extractor/`) — Playwright-based WebGL site extraction
  - Captures shaders, textures, uniforms, draw calls, scene graphs
  - 7 sites extracted, validated (32 shaders from WebGL Aquarium alone)
- **MKS Immersive Site Skill** (`.claude/skills/mks-immersive-site/`) — 853-line Claude Code skill
  - Encodes full architecture, design philosophy, 15 bug learnings
  - `/build-world` slash command for generating new environment configs

### Built (Phase 4: Ralph Loop — Refactor & Quality)
- **Route split** — `/` = professional site (231KB), `/experience` = WebGL (lazy loaded, 1.16MB)
- **ESLint zero-error baseline** — 98 errors fixed, 5 unused deps removed (a69604b)
- **Design rule enforcement** — reduced-motion fallbacks, opacity-only transitions, color tokens, verify-all Rules 4-6 (6d7cfbd)
- **Dispose lifecycle** — terrain/sceneSetup/textures/AudioContext/quad geometries/passes (2843013), 45 subsystem scene.remove() sweep (dc7e3f5)
- **Shared GLSL utilities** — `_fog-utils.glsl`, `_rim-light.glsl`, `_particle-utils.glsl` across 16 shaders (d9d598b)
- **Ralph infrastructure** — 13 specs (86 ACs), verify-all.sh with 54 checks
- **7 design skills** installed from taste-skill repo

### Stripped (decided against / needs redo)
- **GhibliClouds** — toon-shaded hemisphere dome. Removed: flat blobs. Needs real volumetric approach.
- **CursorCreatures** — textured butterflies. Removed: looked terrible. User wants simple 3D geometry with wing flap.

### Next Phase: Polish & Integration

**Tier 1 — Blocking core experience:**
1. `CliffTerrainGenerator.js` — Ocean Cliff needs real cliff island geometry
2. Seated figure variant for `ArtistFigure.js` — the DOF emotional anchor
3. `VolumetricCloudSystem.js` — Storm Field sky needs real volumetric clouds

**Tier 2 — High value (7 research winners ready to integrate):**
4. Stars + fireflies integration → Night Meadow (56/70, zero perf cost)
5. Wave grass wind keyframe ramp → Golden Meadow (needs DevTuner testing)
6. Kuwahara + cel-shading combo verification → Ghibli (56/70 with atmosphere fix)
7. Intimate DOF v3 config → Ocean Cliff (61/70, zero additional cost)
8. Rain system keyframes → Storm Field
9. Bezier flower geometry → all worlds (6 archetypes prototyped, drop-in ready)
10. Spray + fog wisps → Ocean Cliff ambiance

**Tier 3 — Polish:**
11. Shadow-map god rays (`three-good-godrays` integration)
12. Entry page improvement (better source art — current scores 38/70)
13. Split-tone color grading → Ocean Cliff (warm shadows, cool highlights)
14. Camera arc fix at t=0.75 → Night Meadow (ground-level is emotional peak)
15. Real content — migrate actual album art, bio text, products
16. `prefers-reduced-motion` — freeze camera lerp, jump directly to target

### Known Issues
- Flower geometry is still procedural (cylinder+sphere), not Bezier or .glb models
- Content sections are placeholder text
- God rays render the full scene twice per frame (doubling draw calls when enabled)
- Entry page dithering source art is "too clean/symmetric" (38/70)
- Track-to-world mapping only defined for Golden Meadow ("In a Field of Silence")

## Refactor Plan

### Completed (Tier 1) ✓
1. ~~Extract shared `SECTION_T_VALUES`~~ → `src/meadow/constants.js`, imported by MeadowEngine, FlowerInstances, ContentOverlay
2. ~~`GrassChunkManager.setUniform(key, value)`~~ → propagates to base material + all chunk clones. Used in AtmosphereController and DevTuner.
3. ~~`PostProcessingStack.setGodRayTexture(tex, intensity)`~~ → god ray wiring moved out of `_tick()`
4. ~~Return ambient light from `setupScene()`~~ → `sceneSetup.ambientLight` replaces fragile `scene.children.find()`

### Tier 2: Do Soon (moderate effort, reduces maintenance pain)

5. **Move effect files into `meadow/effects/`** — 10 effect files only imported by PostProcessingStack. Grouping reduces cognitive load. *Claude can do autonomously.*

6. **DevTuner param builder cleanup** — `buildParamGroups()` still ~500 lines but grass setters are now 1-line each (via setUniform). Consider subsystems exposing `getDevParams()`. *Claude can do autonomously, human should review param organization.*

### Tier 3: Consider Later (larger refactor, needs taste)

7. **Per-parameter atmosphere locking** — freeze is all-or-nothing. Per-param locking would let tuned values survive scroll changes. *Human taste: is the complexity worth it?*

8. **Audio-reactive composition** — `_tick()` mutates bloom/CA additively on atmosphere values. Fragile. Should use multiplier or layer system. *Human taste: decide the musical interaction model.*

9. **Subsystem lifecycle audit** — systematic audit of all `addEventListener` / `new THREE.*` for clean teardown. *Claude can do autonomously.*

### Where Human Taste Is Critical

| Step | What Needs Human Input |
|------|----------------------|
| **Any visual feature** (butterflies, clouds, sky) | How it looks, feels, what reference to chase |
| **Atmosphere keyframe values** | The emotional arc (cold→warm→peak→exhale) is artistic |
| **Post-FX intensity curves** | Bloom, grain, vignette darkness at each scroll position |
| **Content layout** | What goes in each clearing, how text interacts with meadow |
| **Audio integration** | What sounds play, when, how they interact with visuals |
| **DevTuner param ranges** | What min/max makes sense for each slider |

### Where Claude Can Act Autonomously

| Task | Why It's Safe |
|------|--------------|
| Dead code removal | No ambiguity — unused = delete |
| Allocation optimization | Module-level reusable vectors follow established pattern |
| Method extraction | Pure mechanical refactor, no taste needed |
| Constant extraction | No behavior change |
| Dispose/cleanup methods | Preventing leaks, following existing patterns |
| Build verification | Objective pass/fail |

## Context Preservation

If a session runs out of context, the next session should:

1. **Read this CLAUDE.md first** — it's the complete project state
2. **Check `git log --oneline -20`** — see what was done recently
3. **Check `git diff --stat`** — see uncommitted work
4. **Check `git branch`** — you're likely on `feature/environment-worlds`
5. **Read `docs/superpowers/specs/2026-03-13-environment-worlds-prd.md`** — the multi-world spec
6. **Read `orchestration/window-2-status.md`** — research pipeline state and winner recipes
7. **Run `npx vite build`** — verify the build is clean before starting

The codebase now has two key entry points:
- **WorldEngine.js** — the config-driven engine (read this + environment configs for world architecture)
- **AtmosphereController.js** — the keyframe system (read this for emotional arc tuning)
- **This file** — complete project state and all learnings

---

## Learnings Log

- 2026-03-12: `three-good-godrays` exports `GodraysPass` (a Pass), not `GodRaysEffect` (an Effect). It needs a DirectionalLight with castShadow=true, not a Mesh. Params use `raymarchSteps` not `samples`.
- 2026-03-12: `BufferGeometryUtils` must be imported from `three/examples/jsm/utils/BufferGeometryUtils.js`, not from `THREE.BufferGeometryUtils` (doesn't exist on the THREE namespace)
- 2026-03-12: When cloning ShaderMaterial for per-instance uniforms, cloned materials get independent uniform objects. Must iterate all clones to update shared uniforms like `uTime`.
- 2026-03-12: Shaders must output linear values when post-processing (bloom etc) is in the pipeline. Per-shader gamma/ACES causes double correction. Let renderer handle tonemapping+gamma.
- 2026-03-12: `scene.background = color` overrides Sky dome visuals. Don't set it when using Sky from three/examples.
- 2026-03-12: Texture paths like `/src/assets/textures/foo.jpg` work in dev but break in production (Vite hashes filenames). Use ES module imports: `import url from '../assets/textures/foo.jpg'`
- 2026-03-12: Camera spline Y must account for terrain height — constant Y will clip into rolling hills. Use `getTerrainHeight(x, z) + offset`.
- 2026-03-12: InstancedMesh vertex shaders MUST use `instanceMatrix` — without it, all instances render at origin. Also compute normals from `mat3(instanceMatrix) * normal`.
- 2026-03-12: `instanceMatrix` on InstancedMesh: don't replace with `new InstancedBufferAttribute(data, 16)`. Instead: `mesh.instanceMatrix.array.set(data); mesh.instanceMatrix.needsUpdate = true`
- 2026-03-12: tmux workers launched from Claude Code fail with "nested session" error. Fix: `unset CLAUDECODE && claude --dangerously-skip-permissions`
- 2026-03-12: tmux correction files (CORRECTIONS.md) may not be read by workers if they've already started. Send corrections early or pre-merge fix.
- 2026-03-13: User wants all shader code stolen from real GitHub repos, never written from scratch. Sources matter — verify they contain real creative techniques, not generic AI-generated examples.
- 2026-03-13: GhibliClouds (hemisphere dome + FBM toon shader) produced flat blobs. Don't attempt procedural clouds without a real reference implementation.
- 2026-03-13: Textured plane butterflies look terrible. User explicitly wants simple 3D geometry with wing flap, no textures. Don't try PNG-based approaches.
- 2026-03-13: AtmosphereController overwrites all subsystem values every frame. DevTuner changes won't stick unless atmosphere is paused (freeze mode). This is the root cause of "sliders don't work."
- 2026-03-13: Lenis hijacks scroll events globally. Use `data-lenis-prevent` attribute on elements that need independent scrolling (DevTuner panel).
- 2026-03-13: Always avoid per-frame allocations (`new THREE.Vector3()` etc). Use module-level reusable objects prefixed with underscore (`const _lookTarget = new THREE.Vector3()`).
- 2026-03-13: When iterating a Map and deleting entries, snapshot keys first (`[...map.keys()]`) to avoid mutation-during-iteration bugs.
- 2026-03-13: Old site files (LandingSection, FlowerField, Overlays, FlowerVisual, App.css, noise.js, flowers.js) were deleted — they were never imported by the current App.jsx.
- 2026-03-13: PostProcessingStack.dispose() was only cleaning up 5 of 12 effects. Always dispose ALL effects/passes in cleanup methods.
- 2026-03-13: TerrainPlane had duplicated height formula in createTerrain and getTerrainHeight. Single-source the formula by calling getTerrainHeight from createTerrain.
- 2026-03-13: User wants to use DevTuner to dial in visual values before committing to them in code. The workflow is: freeze atmosphere → tune sliders → export JSON → apply values to keyframes. Human taste step.
- 2026-03-14: `renderer.toneMappingExposure` does NOTHING when `renderer.toneMapping = NoToneMapping`. Since we use pmndrs ToneMappingEffect in the post-processing pipeline, exposure must be a uniform in the color grade shader (`uExposure`).
- 2026-03-14: CameraRig.update() sets `camera.fov = this.currentFov` every frame (for scroll-velocity FOV boost). DevTuner must set `cameraRig.baseFov` (not `camera.fov`) or the value gets instantly overwritten.
- 2026-03-14: Cursor grass push was jittery because worldPos snapped to raycast hit every frame. Fixed with `worldPos.lerp(hitPoint, 0.12)` + velocity smoothing. Reset `_initialized` flag on mouseleave so re-entry doesn't lerp from stale position.
- 2026-03-14: Score sheets were invisible because they were at Y:4.5-8.5m (camera is ~1.5m above terrain). Lowered to Y:2.0-4.5m and enlarged (1.8x1.3) to be visible.
- 2026-03-14: pmndrs SSAOEffect property access varies by version. Use `effect.radius` getter/setter rather than digging into `ssaoMaterial.uniforms.radius.value`. Add try/catch with fallback for robustness.
- 2026-03-14: User's first DevTuner tuning session (JSON export at scroll ~0.48). Key taste: wants more golden/desaturated colors, liked cloud shadows, wants smoother grass push, FOV should go more extreme. Effects need to be visually obvious or user can't tell they exist.
- 2026-03-14: Music-as-router pattern: MiniPlayer dispatches route changes → portal transition fires → new WorldEngine config loads. Track selection IS navigation. URLs are secondary.
- 2026-03-14: Entry page must be Canvas 2D (zero Three.js weight). Solves: audio autoplay, GPU warm-up time, emotional threshold. The user chooses to enter.
- 2026-03-14: Environment configs are pure data objects — no Three.js imports. WorldEngine reads the config and wires subsystems. This separation is critical for the config-driven architecture.
- 2026-03-14: Each world needs genuinely different terrain algorithms, not just color swaps. Sin/cos (golden), cliff+ocean (ocean), diamond-square (storm), smooth (ghibli). Without unique geometry, worlds feel like reskins.
- 2026-03-14: Research pipeline IMPROVE cycle delivers +3 to +4 point gains on highest-scoring worlds. Key insight: score against emotional atlas (feeling), not technical quality (pixels).
- 2026-03-14: Firefly emotional optimum is 400 count. 1000 = solid amber band (screensaver feel, 28/70). Void between fireflies IS the grief — don't fill it.
- 2026-03-14: Camera position at t=0.75 (ground-level among fireflies) scores HIGHER than t=0.50 aerial view (33/40 vs 31/40). Being AMONG the lights is more intimate than being above them.
- 2026-03-14: Ghibli "never felt brighter" failed at ambient 0.20 because cel-shading's 4-band system snapped shadow-facing grass to near-black. At 0.45, shadows are dark but recognizably colored — matching actual Ghibli films.
- 2026-03-14: Split-tone color grading (warm amber shadows at 15% + cool blue highlights at 10%) creates "faded memory" quality. Zero performance cost — just uniforms.
- 2026-03-14: Web extractor requires system Chrome (`channel: 'chrome'`) — Playwright's bundled Chromium lacks SwiftShader for headless WebGL rendering.
- 2026-03-14: 4-window parallel orchestration (build, research, tools, skills) is effective. Status files in `orchestration/window-N-status.md` enable cross-window communication.
- 2026-03-15: **FULL PROJECT AUDIT** — 9,968 LOC across 100 src files (66 JS/JSX, 23 shaders, 6 CSS). Build passes clean in 1.63s. 30 commits on feature/environment-worlds. 72 research experiments (19 IMPROVE cycles). 12 winner docs (20+ undocumented above 60/70). 10 sites extracted (949 shaders, 578 programs). MKS skill is comprehensive at ~850 lines. Full audit at `orchestration/audit-2026-03-15.md`.
- 2026-03-15: JS bundle is 998 KB — needs code splitting via dynamic imports before production. Audio asset (11 MB) needs streaming/lazy-load.
- 2026-03-15: Winner documentation backlog is significant — 20+ techniques above 60/70 lack formal docs. Key breakthroughs: GOLDEN RUINS (multiplicative convergence), VIGIL (absence as presence), gravitational lensing (largest weighted gain ever).
- 2026-03-15: Web extractor Tier 1 techniques ready for implementation: Active Theory's analytical curl noise (3x faster), scroll-driven particle lifecycle, Immersive Garden's wave propagation and dissipation model.
- 2026-03-26: **Ralph Loop bootstrapped** — 13 specs (86 ACs), verify-all.sh (54 checks), 5 build iterations completed (lint, design rules, dispose x2, shared GLSL). Commits a69604b through d9d598b.
- 2026-03-26: Route split live — `/` = professional site, `/experience` = WebGL experience. Lazy loaded: 231KB entry vs 1.16MB WebGL chunk (commit 6210585).
- 2026-03-26: 7 new design skills installed from taste-skill repo (creative layering, creative coding principles, etc.).
- 2026-03-26: Shared GLSL utilities (`_fog-utils.glsl`, `_rim-light.glsl`, `_particle-utils.glsl`) refactored 16 shaders across 13 JS files. Actual duplication less than estimated — net ~30 lines reduced (bulk savings come from Steps 6-7).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jshengdev) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
