## the-dime-podcast

> This repo has two parts:

# CLAUDE.md

This repo has two parts:

- `app/` — the Next.js site (dimepodcast.com): episode/video pages, SEO
  schema helpers (`app/lib/schema.ts`), transcript rendering (`app/lib/transcripts.ts`).
- `bot/` — Python automation, two separate pipelines:
  - Guest research bot (daily via GitHub Actions, Trello → Claude → Word doc → email) — see [README.md](README.md).
  - Transcript pipeline (YouTube captions → Claude → episode page content, written to `app/content/transcripts/*.json`) — runs via [.github/workflows/transcript-pipeline.yml](.github/workflows/transcript-pipeline.yml), matching logic in `bot/simplecast_feed.py`.

## SEO / LLM reach strategy

See [SEO_ROADMAP.md](SEO_ROADMAP.md) for what's shipped and what's planned
(topic hub pages, guest entity pages, llms.txt, etc.). Keep that file
updated as items ship rather than scattering roadmap notes in code comments.

---
> Source: [8thRev/The-Dime-Podcast-](https://github.com/8thRev/The-Dime-Podcast-) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
