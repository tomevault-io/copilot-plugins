## circulari-back

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Spec

@docs/index.md

## Key Architectural Decisions

- **Single-tenant MVP**: no collaboration, each user has isolated data
- **Synchronous AI**: POST /items waits for OpenAI before responding — simpler flow, accepted latency tradeoff
- **Storage abstraction**: S3-compatible interface makes AWS S3 and Cloudflare R2 interchangeable
- **3-layer backend**: strict Controller → Service → Repository (NestJS)
- **Auth**: JWT + bcrypt, with Google/Apple social login

## Wiki Update Rule

After implementing any feature or completing a milestone task, update the docs:

1. **In the relevant doc file** (`docs/api.md`, `docs/ai.md`, etc.): change the section badge from `<Badge type="danger" text="Not Implemented" />` to `<Badge type="tip" text="Implemented" />`. If partially done: `<Badge type="warning" text="In Progress" />`.

2. **In `docs/roadmap.md`**: check off the completed tasks (`- [x]`) and update the milestone badge.

3. **In `docs/index.md`**: update the Feature Status Summary table badge for that feature.

Badge reference:
- `<Badge type="danger" text="Not Implemented" />` — not started
- `<Badge type="warning" text="In Progress" />` — partially done
- `<Badge type="tip" text="Implemented" />` — complete

---
> Source: [OtavioHo/circulari-back](https://github.com/OtavioHo/circulari-back) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
