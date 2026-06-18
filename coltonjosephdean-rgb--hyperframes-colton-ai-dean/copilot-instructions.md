## hyperframes-colton-ai-dean

> This workspace uses AI agent skills for framework-specific patterns. Install them if not already present:

# HyperFrames Composition Project

## Skills

This workspace uses AI agent skills for framework-specific patterns. Install them if not already present:

```bash
npx skills add heygen-com/hyperframes
```

Skills encode patterns like `window.__timelines` registration, `data-*` attribute semantics, and shader-compatible CSS rules that are not in generic web docs. Using them produces correct compositions from the start.

| Skill                    | Command                    | Purpose                                                                  |
| ------------------------ | -------------------------- | ------------------------------------------------------------------------ |
| `hyperframes`            | `/hyperframes`             | Authoring compositions, captions, TTS, audio-reactive animation           |
| `hyperframes-cli`        | `/hyperframes-cli`         | CLI commands (init, lint, preview, render, transcribe, tts, doctor)       |
| `gsap`                   | `/gsap`                    | GSAP animation reference                                                  |
| `hyperframes-registry`   | `/hyperframes-registry`    | Installing catalog blocks/components                                       |
| `website-to-hyperframes` | `/website-to-hyperframes`  | URL → video (7-step capture-to-video pipeline)                             |
| `make-a-video`           | `/make-a-video`            | Beginner-friendly end-to-end video creator                                 |
| `short-form-video`       | `/short-form-video`        | 9:16 vertical talking-head + motion-graphic playbook                       |
| `study-creator`          | `/study-creator <url>`     | Study a reference creator's video; rolls up a style fingerprint            |
| `feedback`               | `/feedback`                | Capture what worked/didn't on a finished video; updates `PREFERENCES.md`   |
| `setup`                  | `/setup`                   | First-run onboarding — creator profile, reference reels, brand colors      |

## Commands

```bash
npx hyperframes preview      # preview in browser (studio editor)
npx hyperframes render       # render to MP4
npx hyperframes lint         # validate compositions (errors + warnings)
npx hyperframes lint --json  # machine-readable output for CI
npx hyperframes docs <topic> # reference docs in terminal
```

## Project Structure

- `index.html` — main composition (root timeline)
- `compositions/` — sub-compositions referenced via `data-composition-src`
- `assets/` — media files (video, audio, images)
- `meta.json` — project metadata (id, name)
- `transcript.json` — whisper word-level transcript (if generated)

## Linting — Always Run After Changes

After creating or editing any `.html` composition, run the linter before considering the task complete:

```bash
npx hyperframes lint
```

Fix all errors before presenting the result.

## Key Rules

1. Every timed element needs `data-start`, `data-duration`, and `data-track-index`
2. Visible timed elements **must** have `class="clip"` — the framework uses this for visibility control
3. GSAP timelines must be paused and registered on `window.__timelines`:
   ```js
   window.__timelines = window.__timelines || {};
   window.__timelines["composition-id"] = gsap.timeline({ paused: true });
   ```
4. Videos use `muted` with a separate `<audio>` element for the audio track
5. Sub-compositions use `data-composition-src="compositions/file.html"`
6. Only deterministic logic — no `Date.now()`, no `Math.random()`, no network fetches

## Documentation

Full docs: https://hyperframes.heygen.com/introduction

Machine-readable index for AI tools: https://hyperframes.heygen.com/llms.txt

---
> Source: [coltonjosephdean-rgb/Hyperframes-colton.ai.dean](https://github.com/coltonjosephdean-rgb/Hyperframes-colton.ai.dean) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
