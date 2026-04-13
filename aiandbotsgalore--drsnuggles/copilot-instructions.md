## drsnuggles

> - Read `WORKING-NOTES.md` first for the practical start command, test command, important folders, safe no-touch areas, and the definition of done for this repo.

# Repository Guidelines

## First Read
- Read `WORKING-NOTES.md` first for the practical start command, test command, important folders, safe no-touch areas, and the definition of done for this repo.

## Project Structure & Module Organization
- `src/main`: Electron main process (audio capture/resample, LLM clients, knowledge base, memory).
- `src/renderer`: React UI (components, panels, services, analytics).
- `src/shared`: Types shared between main and renderer.
- `src/lib`: Shared utilities (e.g., `audioResampler.ts`).
- `dist`: Build output for main and renderer.
- `src/config`: Runtime tuning and performance settings.

## Build, Test, and Development Commands
- `npm run dev`: Runs Electron main + Vite renderer in watch mode.
- `npm run dev:main`: Type-checks main process and launches Electron.
- `npm run dev:renderer`: Starts the Vite dev server for the UI.
- `npm run build`: Builds main and renderer for production.
- `npm run start`: Runs Electron against the built output.
- `npm run verify-api`: Builds main and runs API verification script.

## Coding Style & Naming Conventions
- Use TypeScript and React conventions already in the codebase.
- Indentation is 2 spaces (see `src/renderer/App.tsx`).
- Component files are `PascalCase.tsx` (e.g., `DrSnugglesControlCenter.tsx`).
- Hooks are `useX.ts` (e.g., `useAudioServices.ts`).
- Services use `*Service.ts` and are grouped under `src/renderer/services`.

## Testing Guidelines
- No formal test runner is configured.
- Ad-hoc scripts exist: `test-electron.js`, `test-key.ts`, `src/brain/test-brain.ts`.
- Run with `node test-electron.js` or `npx tsx src/brain/test-brain.ts` as needed.

## Commit & Pull Request Guidelines
- Follow conventional commit style seen in history: `feat(scope): ...`, `chore: ...`.
- Keep messages concise and task-focused (e.g., "feat(ux): replace native confirm dialogs").
- PRs should include a short summary, linked issue (if any), and UI screenshots for renderer changes.

## Configuration & Secrets
- Set `GEMINI_API_KEY` via environment or `.env.local`.
- Do not commit secrets or local machine paths.
- Performance tuning lives in `src/config/performance.config.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aiandbotsgalore)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/aiandbotsgalore)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
