## saturno-site

> - Parent context exists at `../AGENTS.md`.

# Saturno project agent context

## Start here
- Parent context exists at `../AGENTS.md`.
- Parent local Hermes context: `../.hermes/context.md`.
- Parent local memory: `../.hermes/memory/project.md`, `../.hermes/memory/decisions.md`, `../.hermes/memory/lessons.md`.
- Parent active Plannotator/tasks: `../.hermes/plans_tasks/active.md`.
- Local skill: `saturno-enneagram-web` from `../.hermes/skills/`.
- OpenCode project skill: `saturno-project-integrity` from `~/.opencode/skills/` (load when doing architectural or integrity work).

## Operating mode
- Caveman mode: brief, direct, no narrative.
- Context mode: read local memory/plans before asking or guessing.
- Prefer tool-backed facts over guesses.
- Keep context lean. Load only skills relevant to current task.
- Default useful tools: file, terminal, code_execution, web, skills, todo, delegation.
- Avoid browser/vision/image/tts/messaging/cron unless explicitly needed.
- Check `git status --short` before editing.
- Do not overwrite dirty/untracked files blindly.

## Project goal
Saturno is a modern Enneagram personality-test website.
Goal: fun, animated, trustworthy self-reflection experience.
Current planned direction: “Saturno Type Orbit” with scenario questions, animated enneagram constellation, transparent scoring, core type + wing + center + growth/stress result reveal.

## Product rules
- Enneagram is self-reflection, not clinical diagnosis.
- Use “likely pattern”, “reflection tool”, “growth path”; avoid deterministic claims.
- Optimize for playful premium UX, not a boring form.
- Respect accessibility and `prefers-reduced-motion`.

## Tech rules
- Next.js 16 + React 19 + TypeScript + Tailwind CSS 4 + Vitest.
- This is NOT old Next.js. Read `node_modules/next/dist/docs/` before using uncertain Next APIs.
- Before finalizing code changes, run relevant checks:
  - `npm run test` (includes integrity tests at `tests/integrity/*.test.ts`)
  - `npm run typecheck`
  - `npm run lint`
  - `npm run build` when launch-impacting.

## Important files
- Plan: `docs/plans/2026-05-30-modern-enneagram-experience.md`
- Data model: `data/enneagram-system.ts`
- Scoring: `lib/scoring-engine.ts`
- Tests: `tests/enneagram.test.ts` + `tests/integrity/*.test.ts`
- Landing: `app/page.tsx`
- Quiz orchestrator: `components/quiz/QuizApp.tsx`
- Profile: `app/profile/page.tsx` + `components/profile/`
- Characters: `app/characters/page.tsx` + `components/characters/`
- Integrity skill: `~/.opencode/skills/saturno-project-integrity/SKILL.md`

## Implementation priority
1. ✅ Research doc.
2. ✅ Expand Enneagram data contract → `data/enneagram-system.ts`.
3. ✅ Upgrade scoring → `lib/scoring-engine.ts` + integrity tests.
4. ✅ Split quiz components → `components/quiz/`.
5. ✅ Build result reveal → `app/profile/` + `components/profile/`.
6. ✅ Characters/type gallery → `app/characters/` + `components/characters/`.
7. Ongoing: Verify with integrity tests + typecheck/lint/build before merging.

---
> Source: [SaturnoEnneagram/saturno_site](https://github.com/SaturnoEnneagram/saturno_site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-12 -->
