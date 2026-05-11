## endfield-pomodoro

> - Name: endfield-pomodoro

# AGENTS.md

## Project
- Name: endfield-pomodoro
- Stack: React 19 + TypeScript + Vite + TailwindCSS v4
- Goal: Pomodoro timer with theme system, i18n, and music playback.

## Runbook
- Install: `pnpm install`
- Dev: `pnpm dev`
- Build: `pnpm build`
- Lint: `pnpm lint`
- Format: `pnpm format`
- Check: `pnpm check`

## Source Map
- App entry: `src/main.tsx`, `src/App.tsx`
- Core components: `src/components/`
- Theme config: `src/config/themes.ts`
- i18n: `src/utils/i18n.ts`
- Music config: `src/config/musicConfig.ts`
- Types/constants: `src/types.ts`, `src/constants.ts`

## Change Rules
- Keep TypeScript types strict; update `src/types.ts` when contracts change.
- Reuse existing component layering: `components/ui`, `components/themes`, business components.
- For UI text changes, update both zh/en entries in `src/utils/i18n.ts`.
- For theme changes, prefer CSS variables and centralized config in `src/config/themes.ts`.
- Do not introduce unrelated refactors in feature/fix tasks.

## Done Criteria
- Relevant commands pass: `pnpm lint` & `pnpm check`.
- If behavior changed, include/update minimal tests or verification notes.
- Keep diffs focused and document key decisions in PR/task notes.

---
> Source: [ChuwuYo/Endfield-Pomodoro](https://github.com/ChuwuYo/Endfield-Pomodoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
