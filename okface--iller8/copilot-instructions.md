## iller8

> A personal Serbian language learning SPA. Teaches practical, modern Serbian through interactive exercises with spaced repetition. Built for deployment on GitHub Pages. The content is tailored for an English speaker learning Serbian to talk to their half-Serbian girlfriend and Serbian/Bosnian best friend (a backend developer).

# iller8 — Serbian Language Learning App

## Project Overview
A personal Serbian language learning SPA. Teaches practical, modern Serbian through interactive exercises with spaced repetition. Built for deployment on GitHub Pages. The content is tailored for an English speaker learning Serbian to talk to their half-Serbian girlfriend and Serbian/Bosnian best friend (a backend developer).

## Commands
- `npm run dev` — Start dev server
- `npm run build` — Type-check + production build
- `npx tsc --noEmit` — Type-check only
- Build output goes to `dist/`

## Tech Stack
- Vite + React 18 + TypeScript
- Tailwind CSS v4 (via `@tailwindcss/vite` plugin)
- React Router v6 (HashRouter for GitHub Pages)
- localStorage for all state persistence
- No backend

## Architecture
- `src/data/lessons/*.json` — 10 lesson files with ~360 total phrases
- `src/engine/srs.ts` — SRS algorithm: 8 buckets (0-7), intervals from 1h to 30d
- `src/engine/exercise-generator.ts` — Generates exercises, handles direction locking by bucket
- `src/engine/script-converter.ts` — Cyrillic ↔ Latin conversion
- `src/engine/scoring.ts` — Answer checking with fuzzy matching for diacritics
- `src/store/types.ts` — All TypeScript types
- `src/store/progress.ts` — localStorage persistence for UserProgress
- `src/components/exercises/` — One component per exercise type
- `src/pages/` — Route-level page components
- `src/lib/claude.ts` — Claude API client for custom content extraction

## Exercise Types (9 total)
1. `MultipleChoice` — Pick the correct translation from 4 options
2. `TypeTranslation` — Type the translation (fuzzy matching for diacritics)
3. `FillInBlank` — Complete a sentence (targets content words, not function words)
4. `MatchPairs` — Connect Serbian phrases to English meanings
5. `WordTiles` — Arrange word tiles into a sentence
6. `ScriptConvert` — Convert between Cyrillic and Latin script
7. `ContextPick` — Pick the right phrase for a situation
8. `SentenceBuilder` — Build a Serbian response from a situational prompt with distractor tiles
9. `Comprehension` — Read a Serbian-only dialogue and answer comprehension questions

## Key Patterns
- All lesson content is in JSON data files, separate from code
- Script toggle (Latin/Cyrillic) is a global setting passed via props
- Progress state lives in a single `UserProgress` object persisted to localStorage
- Exercise components share a common `onAnswer(correct: boolean)` callback pattern
- Exercise components MUST use `key={currentIndex}` to force remount between questions
- SRS uses bucket-based progression (0-7) with time decay scaled by bucket level
- Direction locking: bucket 0-1 = SR→EN only; bucket 2 = 70/30; bucket 3+ = balanced
- Failed items get retried within the same session (retry queue, max 2 retries)
- New phrases go through a PhraseIntro phase before exercises begin (5 at a time)
- Notes are shown AFTER answering, never before (to avoid spoiling answers)

## Lesson Flow
1. PhraseIntro phase — 5 new phrases: present → breakdown → context → echo (type it) → quick recall check
2. Exercise phase — 15 mixed exercises with progressive difficulty
3. Retry queue — Failed items reappear 4 exercises later
4. Completion screen — Accuracy stats, phrase review summary

## Adding New Lessons
1. Create a JSON file in `src/data/lessons/` following the existing format
2. Import it in `src/data/lessons/index.ts` and add to the `lessons` array
3. Each phrase needs `id`, `sr_latin`, `sr_cyrillic`, `en`, and `context`
4. Add `variations` for phrases with gender/formality variants
5. Add `notes` for grammar explanations (shown post-answer)

## Design
See `DESIGN.md` for visual design instructions for Claude Design.

---
> Source: [okface/iller8](https://github.com/okface/iller8) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
