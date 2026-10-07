## promo-video

> Rules for making promo videos with this repo

This repo renders promo videos from HTML scenes. Before doing anything video-related, read `AGENTS.md` (workflow, hard rules, gotchas) and `docs/API.md`.
Scenes must be pure functions of time (`window.seek(t)`), timings live in `cues.json`, fonts are bundled, outputs go to `out/`, and nothing is committed unless the user asks.

---
> Source: [visser23/promo-video-generator](https://github.com/visser23/promo-video-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-07 -->
