## compagnon-eveil

> Yorimi is an adaptive AI character presence demo for study, work, emotional check-ins, routines, object-based companions, and creator IP interaction.

# AGENTS.md

## Project Purpose

Yorimi is an adaptive AI character presence demo for study, work, emotional check-ins, routines, object-based companions, and creator IP interaction.

The companion must reply naturally from:
- the user's latest message
- selected tone
- selected role
- selected use case
- current mode
- recent chat history
- companion profile and memory

The model is the brain. Local code provides context, routing hints, schema validation, session memory, rendering, assets, voice fallback, and emergency fallback behavior.

## Repository Organization

1. `docs/` contains plans, pitch material, team deliverables, and test matrices.
2. `frontend/static/` contains the served standalone HTML demo and companion data.
3. `frontend/companion-experience/` contains the companion interaction-system source files.
4. `server/engines/openai/` contains the OpenAI adapter.
5. `server/engines/mock/` contains emergency fallback logic only.
6. `server/engines/runtime/` contains orchestration and runtime guards.
7. `server/tests/` contains executable contract and smoke tests.
8. `assets/` contains committed visual/video companion assets.
9. `api/` contains Vercel Serverless Function wrappers only. Keep these wrappers as `.mjs` while the package remains `type: commonjs`.
10. Do not add new docs or deliverables to the repo root unless a tool convention requires it.

## Non-Negotiable Runtime Rules

1. Normal chat uses the OpenAI runtime when `OPENAI_API_KEY` is configured.
2. `runMockEngine()` is fallback only.
3. Mock fallback must never pretend to be GPT tutoring.
4. Do not route normal study chat directly to mock.
5. Do not generate the main chat answer from `micro_task_plan`.
6. The visible assistant bubble must use `answer` first, then `reply`.
7. The current user message wins over stale mode, stale memory, or previous fallback state.
8. `Study Sprint Mode` is a behavior layer, not a fixed answer template.
9. Never reintroduce generic all-purpose study templates such as "open the most relevant material", "name the first concrete part", "do one small example", or "mark the first blocker" as the main answer.
10. For broad or unknown subjects, let the model infer a useful beginner path.
11. Keep request-level logic in `server/http/runtimeHandlers.js`; `server/index.js` is local-only and Vercel functions must not import it.

## Frontend Rendering Rules

1. Chat bubble rendering must prefer `result.answer || result.reply`.
2. Never build the visible assistant message from `micro_task_plan.map(...)`.
3. `micro_task_plan` may be shown only as optional task cards or buttons.
4. Fallback status must be visible when `fallback_used === true`.
5. Old `localStorage` or `sessionStorage` messages must not mask new runtime responses during testing.
6. Keep a visible or console build marker for runtime debugging.
7. Runtime logs should include request URL, `runtime_source`, `fallback_used`, `intent`, `mode`, `answer`, `reply`, and final rendered assistant text.

## Voice Rules

1. Voice is progressive enhancement. Typing and reading must always work.
2. The Chat Mode Play button must work with browser `speechSynthesis` when Fish Audio is unavailable.
3. `/api/tts` may return `503` when Fish Audio env vars are missing; the frontend must fall back to browser speech or show a clear manual voice state.
4. Video Mode may use `/api/tts`, but `/api/tts` failure must not block normal chat.
5. Browser autoplay restrictions must be handled through user-triggered Play buttons.
6. Never hide the Play button just because Fish Audio is not configured.
7. Do not commit real Fish/OpenAI keys.

## Assets Rules

1. Default companion assets live under `assets/companions/`.
2. Required default assets:
   - Cappu / Coffee Cup: `assets/companions/coffee.jpg`
   - Folio / Open Book: `assets/companions/folio-open-book.png`
   - Luma / Desk Lamp: `assets/companions/luma-desk-lamp.png`
3. Default object companions must use `assetUrl`, not emoji only.
4. Missing image assets must degrade to emoji without breaking setup or chat.
5. Do not reference local assets that are not committed or served by `server/index.js`.

## Env And Secrets Rules

1. `.env` must never be committed.
2. `.env.example` must contain realistic defaults and no real secrets.
3. `OPENAI_API_KEY` is required for live AI.
4. `FISH_AUDIO_API_KEY` and `FISH_AUDIO_REFERENCE_ID` are optional.
5. Missing OpenAI key should trigger schema-compatible mock fallback.
6. Missing Fish Audio keys should not break chat or browser voice fallback.
7. Tests must never print or snapshot secrets.

## Required Tests Before Commit

Run:

```bash
npm run test:agents
npm run test:vercel
npm run test:mart
npm run test:api
npm run test:d
```

Run when `OPENAI_API_KEY` is configured:

```bash
npm run test:openai
```

Required acceptance:
- live chat returns `fallback_used: false` when OpenAI is configured
- mock only appears when OpenAI is missing/failing or explicitly forced
- answer-first rendering is preserved
- voice fallback contract is preserved
- required companion asset paths are present
- `.env` is untracked

---
> Source: [Iamsohungrynow/compagnon-eveil](https://github.com/Iamsohungrynow/compagnon-eveil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-10 -->
