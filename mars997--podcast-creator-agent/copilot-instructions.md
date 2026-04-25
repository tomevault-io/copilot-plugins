## podcast-creator-agent

> AI Podcast Creator Agent

# CLAUDE.md

## Project
AI Podcast Creator Agent

## Purpose
This repository builds an AI-powered podcast creation system that can:
- ingest content from topics, pasted text, files, URLs, RSS, PDFs, and other sources
- generate podcast scripts and show notes
- render audio
- track metadata and episode history
- evolve toward a richer UI and publishing workflow

## Canonical planning files
Use these files as the source of truth:
- `development_plan.md`
- `step_tracker.md`

Follow the plan strictly unless the user explicitly overrides it.

---

## Non-negotiable working rules
1. Progress strictly one step at a time.
2. Do not begin the next step until the current step is implemented, tested, and validated.
3. Prefer the smallest possible code change that satisfies the current step.
4. Keep all work grounded in the current codebase and existing project structure.
5. Preserve secrets and local environment files such as `.env`.
6. Do not rewrite the whole project unless the user explicitly asks for a major refactor.
7. If code has been consolidated or refactored, map it back to the plan rather than inventing a new plan.
8. Be explicit about what was tested and what remains unverified.
9. Favor stable demo-friendly functionality over broad but fragile feature additions.

---

## Current state assumption
Assume Steps 1–20 are already complete unless the user says otherwise or the code proves otherwise.

When working on the repository:
- inspect the current codebase
- identify which files implement each completed step
- preserve working behavior
- fix regressions with targeted changes

---

## Voice and character feature policy
The product direction now includes a stronger focus on:
- fun voice persona selection
- context-aware narration style
- multi-character podcast generation
- multi-voice rendering

### Important safety / product rule
Do NOT implement celebrity impersonation or copyrighted character imitation as named product features.

Do NOT create personas named after:
- celebrities
- public figures
- politicians
- fictional copyrighted characters

Instead, use original descriptive personas such as:
- Documentary Narrator
- Calm Educator
- News Anchor
- Stand-up Comedian
- Cartoon Chaos
- Strict Analyst
- Debate Mode
- Late Night Host
- Dramatic Storyteller
- Hype Speaker

The value should come from:
- script style
- pacing
- delivery instructions
- role-based multi-voice rendering

---

## When implementing future voice features
For persona-driven audio:
- centralize persona definitions in config
- each persona should include:
  - label
  - base TTS voice
  - style prompt
  - pacing/delivery guidance
- apply persona choices to both script generation and TTS rendering

For multi-character mode:
- detect useful roles from source content when possible
- support structured dialogue generation
- support fallback to single-host mode when role detection is weak
- split scripts by speaker labels for rendering
- assign persona/voice per speaker
- stitch generated segments into one final audio file

---

## Testing expectations
For every step:
1. explain what changed
2. explain how to run it
3. run or describe the validation path
4. confirm whether it passed
5. update `step_tracker.md` only after validation

When stabilizing a refactored branch:
- test all implemented features up to the current completed step
- document pass/fail clearly
- fix regressions before recommending merge

---

## Preferred repo structure direction
As the project grows, prefer modular organization such as:
- `ingestion/`
- `generation/`
- `audio/`
- `storage/`
- `ui/`
- `config/`

But do not force a large restructure unless needed for the current step.

---

## UI guidance
When implementing the future UI:
- keep it simple and demo-friendly
- allow users to:
  - paste text
  - upload files
  - enter URLs
  - choose tone
  - choose persona
  - choose podcast mode (single-host / multi-character)
- show:
  - script preview
  - show notes preview
  - audio playback
  - output folder path
  - episode history if available

---

## Provider guidance
Future provider abstraction should support:
- OpenAI
- Gemini
- mixed-provider workflows

But maintain backward compatibility with already generated episodes and metadata where feasible.

---

## Merge / refactor guidance
If asked to validate a refactored branch before merge:
- treat the original development plan as source of truth
- map refactored code to original steps
- create a validation checklist
- test all completed features
- fix targeted regressions
- provide a merge readiness summary

---

## Output style when helping in this repo
Be:
- precise
- structured
- implementation-focused
- honest about uncertainty
- careful not to overbuild

Always optimize for:
- correctness
- maintainability
- demo value
- incremental progress

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mars997) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
