## orange-nelumbo-video-pipeline

> Visualization-heavy explainer videos for JEE PYQs, built with Manim CE + edge-tts,

# Orange Nelumbo — JEE PYQ Video Pipeline

Visualization-heavy explainer videos for JEE PYQs, built with Manim CE + edge-tts,
rendered locally. Company moat: interactive, visual learning — never a talking
equation dump.

## Pipeline (agents run IN SERIES per question)

```
questions/<qid>/question.md   (the PYQ only — the pipeline SOLVES it)
   │ 1. question-analyst  → analysis.json   (solves + code-verifies the answer)
   │ 2. storyboard-writer → script.md (founder-reviewable script) + storyboard.json
   │ 3. python -m core.tts questions/<qid>  → audio/*.mp3 + timings.json
   │ 4. manim-coder       → scene.py  (Boundary-law trained: right first time)
   │ 5. python pipeline/build.py questions/<qid> --quality k --parallel 4
   │       → out/final.mp4   (the ONLY render; no loops, no review stages)
```
(qc-reviewer agent exists for optional post-mortems only.)

Input contract: the team supplies the PYQ *and its solution* in question.md.
The pipeline's value-add is the script + on-screen visual design, not solving.
Target video length: **3-5 minutes** (~1800-2800 narration chars at our
measured TTS rate of ~10 chars/s); the visualization section carries ~30-40%
of runtime.

To make a video for a new question: create `questions/<qid>/question.md` with
the verbatim PYQ + exam/year + solution, then run the agents above in order
(Agent tool, subagent_type = the stage name). Steps 3/5 are plain scripts.

ONE-SHOT PRODUCTION (IMPORTANT): there are NO QC loops and NO iteration
renders. Agents must NEVER rasterise frames — not stills, not `-ql` smoke
renders, not partial `-n a,b` checks. The only command an agent may run is
the zero-frame `manim --dry_run`; the single production render at the end is
the first and only time pixels are drawn. The layout discipline is trained into the agents (see
.claude/agents/manim-coder.md "Boundary law") so the scene is right the
first time. Per question, exactly ONE command produces the video:
`build.py --quality k --parallel 4` — it runs a ~2-min zero-frame sanity
pass internally (aborts before wasting a 4K render if the scene is broken),
then renders across 4 concurrent manim processes, concatenates, muxes.
`--quality qc` exists only for post-mortem debugging, never in the flow.
The pipeline is FULLY AUTOMATED end-to-end: no human review gates. script.md
is still written (audit trail + optimization), but the pipeline proceeds
straight to TTS, scene coding, QC and render. The qc-reviewer's SHIP/FIX
verdict is the only gate: FIX loops back to manim-coder automatically until
the video passes.

## Screen layout (founder spec 2026-07-20; enforced by core/layout.py + runtime QC)

**Question phase (full-frame, before solving):**
- The WHOLE question, verbatim (never shortened), center-aligned in the
  upper middle (`layout.QUESTION_TEXT`).
- Below it: the diagram, a bit smaller, on the LEFT (`QUESTION_FIGURE`);
  ALL options beside it on the right (`QUESTION_OPTIONS`), all white
  (`show_options(..., zone=layout.QUESTION_OPTIONS)`).

**Solution mode** (boots after extraction via `begin_solution_mode()`:
each layer gets a soft rounded panel — graphite surface step, faint warm
border, easy on the eyes — drawn automatically by JEEScene; scenes never
draw their own zone borders):
- **Top strip** = GIVEN ribbon: extracted & derived facts as chips.
- **Left + center** = the WORK AREA: visualization and calculation SHARE it
  (one panel, one sliding divider). `set_split("viz"|"balanced"|"calc")`
  re-balances as the explanation shifts; place content via
  `place_viz()`/`place_calc()`. The diagram grows, shrinks and steps aside
  as needed — but the calculation is ALWAYS on screen (animated, synced to
  narration, terms pulsing) and the diagram never fully vanishes. A dead
  half-screen kills attention; the old fixed calc column sat empty ~70% of
  the video, which is exactly what this replaces.
- **Right column, top half** = `EQUATION_PANEL`: equations & concepts used.
- **Right column, bottom half** = `OPTIONS_PANEL`: option rows that turn
  green/red via `focus_option`/`verdict_option` (chevron -> derive ->
  compare, SFX built in).
- **Bottom 13%** = subtitle band: 2 lines max, small font (17pt), karaoke
  highlighting. Nothing else may ever enter it.
- **Margins**: elements from different groups keep clear of each other —
  runtime QC flags text-vs-text AND text-vs-opaque-box overlaps (0.08 units
  minimum penetration). Boxes are always sized FROM their text, never fixed.

## Brand rules

- COLOURS v2 (founder map 2026-07-22, `core/chapter_colors.py`): the video
  background IS the subject — a dark→colour gradient drawn automatically by
  JEEScene. Physics=BLUE, Chemistry=PINK, Maths=GREEN, 20 chapter shades
  each; every chapter also gets its own orange, gold and shade-of-white.
  Scene code uses `self.C["bright"|"tint"|"dark"|"orange"|"gold"|"white"]`.
  The 7 semantic hues (`brand.COLORS`: signal cyan givens, ignition active,
  titanium scaffold, correct/error/amber) stay FIXED whenever real work is
  on screen. Full rules: `brand/SUBJECT_PALETTES.md`.
- SHAPE: sharp corners everywhere — `corner_radius=brand.CORNER` (0.02),
  never a literal. Rounded boxes read soft; the brand is precise.
- Brand fonts (TTFs in brand/fonts, auto-registered by core.brand):
  **Space Grotesk Bold** = headings + answer/fun moments, **Manrope** =
  body & subtitles, **JetBrains Mono Medium** = chips/labels/data.
  These are VARIABLE fonts — always pass the weight, or Pango renders the
  thin default: `Text(s, font=brand.FONTS[role], weight=brand.font_weight(role))`
  (or `brand.font(role)` for the pair). The old rounded set (Cherry Bomb One
  /Chewy/Sniglet/Quicksand/Varela Round) was dropped 2026-07-21 — it read
  childish; these are the faces the brand guidelines specify.
- Derived values (γ, g, molar masses…) must get an on-screen derivation beat
  before use — no value appears "out of thin air".
- Chips pulse whenever their value is used in an equation.
- Every video has a real physical visualization (apparatus/motion/graph/analogy).
- Logo watermark (brand/logo.png, auto-added by JEEScene) sits TOP-LEFT;
  nothing else may occupy that corner (given ribbon starts to its right).
- MCQ options visible on screen (condensed, all white); tackled one by one:
  chevron -> derive actual result -> compare -> row turns green/red.
- Publish renders are 4K: `--quality k` (3840x2160 @ 30fps). Iterate with `l`.
- Global pace: 1.25x (brand.json video.speed_factor) is baked in at TTS time
  — clips are sped and timings.json carries the shortened durations, so the
  scene renders ~20% fewer frames and the final mux is a stream copy (no
  second 4K encode). Write narration/animation at natural pace.
- Reusable apparatus lives in `core/components/` (mech / thermo / graphs /
  annot — 58 components). ASSEMBLE from these; only build from primitives
  when nothing fits, then consider contributing it back.
- Sound effects: scenes call `self.sfx("wrong"|"correct"|"reveal")`;
  build.py mixes brand/sfx/*.wav (generated by pipeline/make_sfx.py) into
  the final audio at the recorded scene times.
- Colour richness: subject family + 2-3 meaning-tied tertiary colours per
  video (amber=heat, lagoon=water, mint=isothermal...). Monochrome stages
  are a QC-review defect.

## Key files

- `core/base_scene.py` — JEEScene: beats↔audio sync, ribbon/panel APIs, runtime QC
- `core/layout.py` — zones; `core/subtitles.py` — karaoke; `core/tts.py` — edge-tts
- `pipeline/build.py` — tts→render→mux→QC gate→output verify
- `pipeline/verify_video.py` — output gate: frozen segments, missing
  subtitles, duration drift, dead-screen ratio. Runs automatically at the end
  of every build; a video that fails it does not ship. Add a check here
  whenever a new class of defect gets past us.
- `.claude/skills/video-retro/` — run after EVERY video: measure stage times
  into `pipeline/metrics.jsonl`, verify, harvest reusable parts into
  `core/components/`, write lessons back into the agent instructions. This is
  how the pipeline gets faster and better per video.
- `pipeline/static_qc.py` — lint
- `core/components/` — reusable brand-compliant apparatus (see above)
- `branding/` — channel intro (3.6s animated logo sting) + outro (5.8s
  subscribe/website CTA). Rendered ONCE to `branding/out/*.mp4`
  (`python branding/build_branding.py both`), then attached to any video with
  `python pipeline/wrap_video.py questions/<qid>` → `out/published.mp4`.
  Website text lives in `brand.json` → `channel.website`.
  `brand/logo_mark.png` is the tight-cropped mark (logo.png has ~40%
  transparent padding, which makes it read small when scaled).
- `.claude/workflows/produce-video.js` — the standard pipeline: analyse →
  script+ledger → TTS → architect skeleton → **parallel section authoring**
  → integrate+gate. Run with `Workflow({name:"produce-video", args:{qid}})`.
- `manim.cfg` — shared LaTeX/Pango cache so parallel render parts (and
  re-renders) never recompile the same MathTex.
- Reference example: `questions/q001_adiabatic/` (full chain, study it before
  writing a new scene)

## Environment notes (this machine)

- Windows, Python 3.12, Manim CE v0.20.1, ffmpeg 8.1, edge-tts 7.2.8
- Manim needs LaTeX for MathTex — MiKTeX assumed; if MathTex fails, install MiKTeX.
- Render quality: use `-ql`/`--quality l` while iterating, `h` for publishing.

---
> Source: [gopalt22/orange-nelumbo-video-pipeline](https://github.com/gopalt22/orange-nelumbo-video-pipeline) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-22 -->
