## demo-video-skill

> When the user asks for a demo video, feature walkthrough, product demo, or a screen recording of a

# Demo Video capability (Playwright + ffmpeg)

When the user asks for a demo video, feature walkthrough, product demo, or a screen recording of a
localhost web app — even when phrased as "make something that shows this feature running" — build
it by driving and recording the real running app headlessly with Playwright, baking overlays
(subtitle bar, feature label, smooth fake cursor, zoom) into code, and speed-compressing LLM/loading
dead-air with ffmpeg. No screen recorder (Cap/OBS) is used. This is for real screen-operation demos,
not motion-graphic teasers.

Key rules:
- Inject the overlay on `document.documentElement` (html), not `body` — body-level zoom must not
  scale the subtitle/cursor.
- Use a fake SVG cursor with a CSS transition (the real cursor is invisible in headless mode).
- Detect LLM/loading waits with `waitForFunction` (never a fixed sleep); record `waitMs` and speed
  up only that window 6–9x.
- `ctx.close()` MUST come before `browser.close()` or the webm is never flushed.
- `goto` with `waitUntil:'networkidle'`. Never kill a server the user started.

Workflow: storyboard + approval → env check → one PoC scene + tone approval → capture scenes
(`scripts/cap_scene.template.js`, replace only the SEQUENCE block) → post-process + montage QA
(`scripts/postprocess.sh`) → concat → confirm via user real playback.

See `SKILL.md` / `AGENTS.md` and `references/playbook.md` for full detail.

---
> Source: [Kminer2053/demo-video-skill](https://github.com/Kminer2053/demo-video-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
