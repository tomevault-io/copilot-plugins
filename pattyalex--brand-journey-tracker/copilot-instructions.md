## brand-journey-tracker

> - All MegAI prompt logic lives in `src/lib/megai-prompt.ts`. Do not move it elsewhere.

# CLAUDE.md — HeyMeg Brand Journey Tracker

## MegAI Prompt Logic

- All MegAI prompt logic lives in `src/lib/megai-prompt.ts`. Do not move it elsewhere.
- Never rewrite or refactor `lib/megai-prompt.ts` unless explicitly asked to do so.

## Content Ideas

- Content ideas must always be theme-specific and tailored to the user's current brand journey stage. Never generate generic or one-size-fits-all suggestions.

## Preventing Repetition

- Always pass `previousIdeas` to API calls that generate content ideas. This ensures the AI does not repeat suggestions the user has already seen or dismissed.

---
> Source: [pattyalex/brand-journey-tracker](https://github.com/pattyalex/brand-journey-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-15 -->
