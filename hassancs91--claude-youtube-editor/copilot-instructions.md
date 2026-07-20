## claude-youtube-editor

> **You record the talking head. Claude Code does the rest.** Every screen moment — UI walkthroughs,

# CLAUDE.md — claude-youtube-editor

**You record the talking head. Claude Code does the rest.** Every screen moment — UI walkthroughs,
full-screen statements, diagrams, terminal mockups — is built as Remotion TSX and composited over
your cut. No screen recording, no video editor. The right skill is picked from the request:

| The user asks to… | Skill | Output |
|---|---|---|
| set up / change their brand, colors, fonts, wordmark | `/brand-setup` | `brand.md` + `remotion/src/{brand.ts,fonts.ts}` + a proof render |
| cut the raw footage / tighten pacing / remove fillers | `/clean-cut` | master cut + `videos/<project>/work/{analysis/cuts.json, edited-transcript.json}` |
| build the visual beats / add overlays | `/make-tsx` (+ `/fake-screencast`) | shots in `remotion/src/shots/<project>/` + a baked preview |
| write a crash-free TSX shot | `/vidtsx-2d-generator` | the low-level Remotion authoring rules |
| remove background noise / isolate voice | `/clean-audio` | a cleaned master (levels preserved) |
| add SFX / sound-design a beat | `/suggest-sfx` | `videos/<project>/work/sfx-plan.json` + an audition mix |
| package a video / titles + thumbnails | `/packaging` | `videos/<project>/packaging/` (1 title × 3 thumbnail bets + rendered thumbs) |
| upload it | `tools/yt_upload.py` | a private draft on YouTube |

The pipeline order is: **cut → visuals → voice → SFX → packaging → upload.**

`brand.md` is the style contract every step-2+ skill reads (palette, motion, delivery specs, SFX
taste). `remotion/src/brand.ts` + `fonts.ts` are the same contract as code — **`/brand-setup` owns all
three together; they drift silently if you hand-edit one.**

## Layout

```
tools/            Python tools (see requirements.txt); the cut-editor UI in tools/editor/;
                  RNNoise models in tools/models/rnnoise/
remotion/         the Remotion project — src/lib/ (kit, browser, screencast, vscode),
                  src/shots/<project>/; registry is GENERATED (npm run gen)
media/            Remotion's public root: library/ (reusable: sfx, music, logos, faces)
                  + projects/<project>/ (media for ONE video — via staticFile('projects/<p>/x'))
videos/           per-video project data — EMPTY until you make one. See videos/README.md
brand.md          the style contract
```

## The example

`remotion/src/shots/example/` is **37 real shots** from a published video — the worked example of the
kit. They render standalone with no footage: `cd remotion && npm run studio`. Read a few before
authoring new ones; that's the fastest way to learn `remotion/src/lib/`.

`remotion/src/shots/brand/BrandProof.tsx` is not a video beat — it's a utility shot that renders the
current brand (wordmark, palette, type) so you can see it. `/brand-setup` uses it.

## Conventions (hard rules)

- **Run everything from the repo root.** Tools resolve *engine* paths (media/library, catalogs,
  remotion/out) against their own location, but *project* paths against the CWD — so pass the project
  as `videos/video-1` and run from the repo root. Example: `python tools/render_cuts.py videos/video-1
  --style natural --mode preview`.

- **Python: use a venv at the repo root.** These tools have real dependencies. Create it once:
  ```
  python -m venv venv
  venv/Scripts/python -m pip install -r requirements.txt     # Windows
  ./venv/bin/pip install -r requirements.txt                 # macOS/Linux
  ```
  Then either activate the venv or call `venv/Scripts/python tools/<tool>.py`. A bare `python` that
  resolves to a system interpreter will hit `ModuleNotFoundError` (requests, Pillow, google-*) — that
  error means you're not on the venv. `ffmpeg`/`ffprobe` and `node`/`npx` must be on PATH (not pip).

- **API keys** live in `.env` at the repo root (copy `.env.example`). Never commit `.env`.
  `ASSEMBLYAI_API_KEY` = transcription · `ELEVENLABS_API_KEY` = voice-isolate + SFX + music ·
  `GEMINI_API_KEY` = thumbnails. **YouTube uses OAuth, not a key**: `tools/yt_upload.py` (upload) and
  `tools/yt_stats.py` (stats) authorize via a browser and cache tokens under `.youtube/`
  (git-ignored). One-time setup: see `tools/yt_upload_SETUP.md`, then `python tools/yt_upload.py auth`.

- **The Remotion registry is generated:** after adding/renaming a shot, `cd remotion && npm run gen`
  (the render/frames scripts do NOT run it themselves). `npm run gen` discovers `src/shots/**` and
  writes `src/registry.gen.tsx` + `src/shots.manifest.json` (both git-ignored).

- **Media rules:** `media/library/` is for CROSS-VIDEO reusable assets only (each with a catalog).
  Anything generated FOR ONE video (screen-caps, example art) goes in `media/projects/<project>/`,
  referenced as `staticFile('projects/<project>/x')`. Reuse before you generate — check the catalogs.
  The face-reference kit for thumbnails is `media/library/faces/` — **you supply your own** images
  (git-ignored; see its README). No face kit, no face thumbnails.

- **Committed vs git-ignored:** the repo carries the *reproducible pipeline* — `cuts.json`, edit
  plans, `timeline.json`, transcripts, TSX shots, SFX/music plans, catalogs. **Raw footage and master
  cuts NEVER go to git** (`videos/*/*.MP4`, `videos/*/reference/`), and neither do rendered
  previews/outputs, audio work files, or rendered face thumbnails (`.gitignore` covers all of it). The
  SFX/music library `clips/*.mp3` ARE committed (there is deliberately no blanket `*.mp3` ignore).

- **QA is not optional:** render frames and READ them before declaring a shot done; run
  `verify_cut.py` on every cut render (it catches ghost speech + A/V drift); audit the SFX cue sheet
  before mixing. Scratch renders/frames go in a scratch dir, not the project.

- **The brand contract is three files.** `brand.md`, `remotion/src/brand.ts`, `remotion/src/fonts.ts`.
  Nothing errors when they disagree — the docs just stop describing the videos. Change them together,
  or run `/brand-setup`.

---
> Source: [hassancs91/claude-youtube-editor](https://github.com/hassancs91/claude-youtube-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-20 -->
