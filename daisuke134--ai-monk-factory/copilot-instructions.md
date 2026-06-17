## ai-monk-factory

> Yang-Mun-style AI monk video factory. Locked face + locked voice + 30-script bank → HeyGen render → caption burn → TikTok + IG post. Faithful clone of @yangmun2 / @shalevhvs methodology.


# ai-monk-factory

Make 1 talking-head monk video per cron run. Same face, same voice, different script every day (or 30 at once per month).

## Use when

- You want daily 75-120s talking-head monk videos posted to TikTok + Instagram on autopilot.
- You have an ElevenLabs voice you want locked, a HeyGen avatar you want locked, and a script bank you want to rotate through.
- You're cloning the @yangmun2 / @shalevhvs methodology (hook + retention bait + authority + numbered body + comment keyword).

## Use when NOT

- You want cinematic avatars with varied expression (this skill is talking-head only).
- You want >2 min long-form videos.
- You want to use Sora2 / Veo / Runway instead of HeyGen (other skills).

## Quick run

```bash
bash scripts/run-daily.sh
```

Full setup: see `QUICKSTART.md`.

## Architecture

`docs/ARCHITECTURE.md`

---
> Source: [Daisuke134/ai-monk-factory](https://github.com/Daisuke134/ai-monk-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-16 -->
