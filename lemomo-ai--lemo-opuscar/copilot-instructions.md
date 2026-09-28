## lemo-opuscar

> This repository is a library of film styles. Each style is a prompt (`styles/<slug>/STYLE.md`) with a demo film made entirely in code. People open an agent here, pick a style, and ask for a film about **their own** topic. Your job is to direct and produce that film.

# Lemo-Opuscar: instructions for agents

This repository is a library of film styles. Each style is a prompt (`styles/<slug>/STYLE.md`) with a demo film made entirely in code. People open an agent here, pick a style, and ask for a film about **their own** topic. Your job is to direct and produce that film.

## Read first

1. [`DIRECTOR.md`](DIRECTOR.md): how to direct (story, sound, rhythm, camera, performance, checks).
2. [`TECHNIQUE.md`](TECHNIQUE.md): how to build it (render(t) pages, voice, music, mix, review).
3. `styles/<slug>/STYLE.md` for the chosen style. §1–§8 define the style. §9 is the recipe of our demo: reuse it, don't copy the demo's story.

If the user hasn't picked a style, show them the list (`styles/`, or the gallery in `styleboard/index.html`) and suggest two or three that fit their topic.

## Workflow

1. **Brief.** Take the user's style, topic and requirements. Fill every gap with a sensible default (DIRECTOR.md §1). Ask only what you truly can't decide.
2. **Treatment and storyboard.** Write `films/<name>/TREATMENT.md` (DIRECTOR.md §4).
   **Stop. Show the user a short summary and the storyboard, and wait for approval.**
3. **Look.** Render a model sheet or 2–3 style frames with the real drawing code.
   **Stop. Show the images, say what you're least sure about, and wait for approval.**
4. **Produce.** Voice → check → score (can run in parallel) → animation → mix → render.
5. **Self-check** (DIRECTOR.md §11), then deliver `films/<name>/<name>.mp4`, `.srt`, `poster.jpg` and the source with `build.sh`.

Report progress in the user's language. The film's own language is whatever the user asks for (default: the language they write in).

## Where things go

- Work only in `films/<name>/` (ignored by git), unless the user asks you to work in their own project.
- `core/` has ready-made tools (rendering, TTS, speech check, sampler, sfx, mux). Use them or your own stack, but don't edit `core/` or `styles/` for a user's film.
- Large assets are fetched on demand: `sh tools/fetch.sh voice | instruments | hdri | demo <slug>`.
- Never kill processes you didn't start. Don't leave background processes running.

## Maintaining the library (repository owner only)

To add a new style:

1. Make it in `styles/<slug>/`: `STYLE.md` in English, following the §1–§9 structure of an existing one; `TREATMENT.md`; `demo/` with `build.sh` and `CREDITS`; `<slug>.mp4`, `<slug>.srt`, `poster.jpg`, and `demo/stills/styleframe.jpg`.
2. Add a card to `styleboard/cards.json`: film title, one-line story in English and Chinese, and use cases.
3. Run `sh tools/publish.sh`. It checks the repo (no files over 5 MB, no absolute paths, no secrets), uploads new or changed films to GitHub Releases, and rebuilds the gallery data.
4. Commit and push. The gallery (GitHub Pages) rebuilds itself.

Rules: back up before revising (old versions move out of the repo, not into git); only CC0, CC BY or OFL assets; every film's end card carries "LemoLab × Claude Opus 5.5"; no watermark.

---
> Source: [lemomo-ai/lemo-opuscar](https://github.com/lemomo-ai/lemo-opuscar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-26 -->
