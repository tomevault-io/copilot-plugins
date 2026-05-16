## hyperframes-ai-video-generation

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

**One Archon workflow** (`create-archon-short`) ported from `diy-yt-creator-hyperframes`. It turns a topic prompt into a vertical YouTube Short rendered with **HyperFrames** (HTML/GSAP-based; not Remotion despite the repo name). Each invocation spawns `videos/<slug>/` from `templates/shorts/archon/`, runs research + script + TTS + composition + lint, and opens a browser preview. Render is always manual.

This repo holds the workflow + template + skills only — there's no global package.json or build at the root. Each `videos/<slug>/` is a self-contained HyperFrames project with its own `index.html`, `audio/`, `assets/`, etc.

## Commands

```bash
# Spawn a new video — default 30s; topic is the entire $ARGUMENTS string
archon workflow run create-archon-short --no-worktree "<topic>"

# Override duration in the topic
archon workflow run create-archon-short --no-worktree "duration 45s, GPT-5 vs Claude coding"

# Per-video CLI ops (after a video exists)
npx hyperframes preview videos/<slug>                    # browser studio
npx hyperframes lint    videos/<slug>                    # validate; ALWAYS run after edits
npx hyperframes inspect videos/<slug>                    # rendered-layout overflow check
npx hyperframes render  videos/<slug> -o videos/<slug>/out/<slug>.mp4

# Workflow validation (after editing the YAML or command files)
archon validate workflows create-archon-short
archon validate commands create-archon-short

# Inspect / clean up Archon state
archon workflow status                # list active runs + working_path
archon isolation list                 # list worktrees
```

Required env in `.archon/.env` (or `~/.archon/.env`): `ANTHROPIC_API_KEY` or `CLAUDE_CODE_OAUTH_TOKEN`. ElevenLabs is optional — default voice is edge-tts `en-US-AndrewNeural`.

Runtime deps: Node ≥18, pnpm, Python ≥3.10 with `pip install edge-tts`, ffmpeg, jq, bun.

## Architecture

### The DAG (3 nodes)

`.archon/workflows/create-archon-short.yaml`:
1. **`parse-input`** (bash) — derives `{topic, slug, duration, title}` JSON. Note: bash node, not script-node — see Gotchas.
2. **`precheck`** (bash) — verifies `templates/shorts/archon/` exists, `videos/<slug>/` doesn't, `npx` is on PATH.
3. **`create-short`** (Claude command) — runs `.archon/commands/create-archon-short.md`, which delegates to `.claude/skills/diy-yt-creator/new-archon-short.md`. This is the ~25min playbook: research → script → TTS → transcribe → compose → lint → preview.

### Skills (loaded by the AI in `create-short`)

| Skill | Purpose |
|---|---|
| `.claude/skills/diy-yt-creator/` | Authoritative playbook (`new-archon-short.md`). Also `new-anthropic-short.md` as shape reference, `capture-asset.md` for screenshot grounding, `qa-composition.md` for visual QA. |
| `.claude/skills/hyperframes/` | Framework patterns — palettes, scripts, references, visual styles. **Always invoked before editing any composition.** |
| `.claude/skills/hyperframes-cli/` | CLI reference (init, lint, preview, render, transcribe, tts). |

### Project rules (`.claude/rules/`)

Loaded into every session. The playbook follows them strictly when editing `index.html`. Critical ones:
- `shorts-typography.md` — min font sizes for 1080×1920
- `visual-pacing-5s.md` — never static more than 5s
- `step-by-step-reveal.md` — enumerated lists reveal one beat at a time; use `tl.set()` + `tl.to()`, never `tl.from()` (visibility leak)
- `tts-pronunciation.md` — heteronym audit before generating narration
- `shorts-thumbnail-final-frame.md` — final ≥1.5s held still that works as YouTube thumbnail
- `audio-design.md` — SFX volumes, alignment audit
- `sub-composition-wiring.md` — strict `data-composition-id` matching; mismatches fail silently in studio

### Per-video output structure

```
videos/<slug>/
├── index.html              ← root composition (root timeline)
├── meta.json               ← { id, name }
├── hyperframes.json        ← schema/registry/paths
├── DESIGN.md, README.md    ← per-video design + spawn notes
├── script.txt              ← narration source
├── audio/
│   ├── narration.wav       ← edge-tts output
│   ├── narration.mp3       ← compressed copy
│   └── narration-chunks/   ← per-sentence intermediates (gitignored)
├── transcript.json         ← word-level timestamps from edge-tts (may be empty if service degrades)
├── compositions/           ← sub-compositions (rare for shorts; phase mutex lives in index.html)
├── assets/
│   ├── archon-logo.png
│   └── sfx/                ← per-video SFX subset, synced from shared/audio/sfx/ via scripts/sync-video-sfx.sh
└── out/                    ← rendered MP4 (gitignored)
```

## Gotchas

- **Bun on Windows truncates `bun -e <script>` at the first newline** when spawned via Node's `execFile` (Archon's mechanism — only line 1 runs, every later statement is silently dropped, exit code is still 0). The `parse-input` node uses a `bash:` wrapper that `mktemp`s a `.js` file and runs `bun --no-env-file run "$TMP" "$ARGUMENTS"` instead. Don't switch back to `script: runtime: bun` for multi-line bodies on Windows.
- **`edge-tts` WordBoundary events sometimes return 0** (service-side, all voices). The playbook falls back to character-proportion estimates, which can be ±1s off. Re-time the composition when service recovers, or switch to ElevenLabs (`scripts/elevenlabs-tts.py` + `tts_lib.py` are available in the source repo if you need to port).
- **Multilingual edge-tts voices are forbidden** — `en-US-AndrewMultilingualNeural` and `en-US-BrianMultilingualNeural` emit empty WordBoundary arrays unconditionally. Stick to `AndrewNeural`, `BrianNeural`, `GuyNeural`.
- **`-no-worktree` is required.** The workflow YAML pins `worktree.enabled: false` because video artifacts must land on the working branch — running in a worktree would dump them into a checkout that has to be merged back.
- **Auto-resume across runs.** Re-invoking `archon workflow run create-archon-short` with the same topic resumes the prior failed run (skipping nodes via `prior_success`). To force a fresh run after editing the workflow, delete the prior run from `~/.archon/archon.db` (`remote_agent_workflow_runs` + `remote_agent_workflow_events` rows) — see Gotcha #2 in the prior `feat/archon-video-generic` branch's CLAUDE.md for the SQL.
- **Windows bash startup overhead.** Anything tighter than ~30s for a bash node is fragile here — bash startup + jq spawn × 3 routinely takes >5s. The `parse-input` and `precheck` nodes are at 30s.
- **CLAUDECODE=1 hangs `archon chat`** silently. Run `archon serve` from a regular shell if you need the orchestrator UI.
- **Shorts MUST end on a thumbnail-grade final frame** — the final ≥1.5s held still is the YouTube auto-pick + loop-pause thumbnail. Never end on a fade-to-black or a CTA-pill-on-empty. See `.claude/rules/shorts-thumbnail-final-frame.md`.
- **Sub-composition wiring fails silently.** When a parent's `data-composition-id` doesn't match the child file's internal `data-composition-id`, the studio shows "Drop media here…" with `0:00/0:00` — and lint passes. Always preview-check duration before declaring done.

## Scope discipline (project rule)

When fixing a bug or shipping a feature, only modify files **directly related** to that task. Refactoring is fine on files we already need to touch; surfacing improvements in unrelated files goes in the PR description as a follow-up suggestion, not in the diff. This applies especially to Archon's simplify / self-fix passes — constrain them to the touch-set or drop unrelated changes before pushing.

---
> Source: [coleam00/hyperframes-ai-video-generation](https://github.com/coleam00/hyperframes-ai-video-generation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-14 -->
