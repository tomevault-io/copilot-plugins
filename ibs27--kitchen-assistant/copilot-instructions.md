## kitchen-assistant

> Project rules for coding agents working in this repository.

# AGENTS.md

## Purpose
Project rules for coding agents working in this repository.

## Roadmap Source of Truth
- Use `/Users/srinivasib/Developer/kitchen-assistant/plan.md` for product scope, phase alignment, and feature priorities.
- Consult `plan.md` whenever a task touches roadmap decisions, MVP boundaries, or sequencing.
- If a user request conflicts with `plan.md`, follow the user request and call out the deviation briefly.

## Scope Guardrails
- iOS-first development. Do not spend time on Android unless explicitly requested.
- Keep MVP scope tight. Do not add new product flows, backend services, or mock systems unless explicitly requested.
- Respect `plan.md` out-of-scope items unless the user asks to include them.
- Temporary scaffolding must be clearly marked as dev-only.

## Build Defaults
- App direction: Expo + React Native (iOS-first), consistent with `plan.md`.
- For local iOS simulator networking, use `http://127.0.0.1:<port>` by default.
- For new `EXPO_PUBLIC_*` vars:
  - update `.env.example`
  - do not commit `.env`
  - remind to restart Metro with `--clear`

## Verification Rules
- After code changes, run `npm run typecheck`.
- For networking changes, provide a reproducible verification command (for example, `curl /health`).
- If sandbox limits prevent runtime verification, explicitly say so and provide exact local verification steps.

## Response Style
- In responses: root cause first, minimal fix second.
- Keep recommendations pragmatic and aligned to the current roadmap phase in `plan.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IBS27)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/IBS27)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
