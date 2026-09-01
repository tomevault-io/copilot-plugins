## demo-video

> Auto-produce a demo/feature-walkthrough video of a local web app with Playwright + ffmpeg (no screen recorder). Use when the user asks for a demo video, product walkthrough, screen recording of a localhost app, or "show this feature working".


# Demo Video (Playwright + ffmpeg)

When the user wants a **demo video / feature walkthrough / product demo / screen recording of a
localhost app** — even phrased as "make something that shows this feature running" — drive and
record the real running app headlessly with Playwright and bake in overlays in code. This is for
real screen-operation demos, NOT motion-graphic teasers.

Core ideas:
1. Inject the overlay (subtitle bar, feature label, fake cursor) on `document.documentElement`
   (html), NOT `body` — zoom transforms `body`, so an html-level overlay is immune to zoom.
2. A fake cursor (SVG + CSS transition) stages the mouse; the real one is invisible headless.
3. LLM/loading waits use `waitForFunction` (never a fixed sleep); record `waitMs` and speed up
   only that window 6–9x with ffmpeg.

Workflow (fixed order): script the scene table & get approval → environment check (never kill the
user's server) → build ONE PoC scene & get tone approved → capture scenes with
`scripts/cap_scene.template.js` (replace only the SEQUENCE block) → post-process & montage-QA per
scene with `scripts/postprocess.sh` → concat into the montage → confirm with user real playback.

Non-negotiables: `ctx.close()` before `browser.close()` (flushes the webm); `goto` with
`waitUntil:'networkidle'`; speed-up on wait windows is mandatory.

Full detail: read `references/playbook.md`. Full skill: `SKILL.md` / `AGENTS.md`.

---
> Source: [Kminer2053/demo-video-skill](https://github.com/Kminer2053/demo-video-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
