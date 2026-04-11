## base-plugin-coordinate-converter

> - Vite + React + TypeScript app; entry at `src/index.tsx` mounts the UI.

# Repository Guidelines

## Project Structure & Module Organization
- Vite + React + TypeScript app; entry at `src/index.tsx` mounts the UI.
- Shared hooks in `src/hooks.ts`; localization setup in `src/i18n/` (`i18n.ts` bootstraps, `resources.ts` holds strings).
- Static assets live in `public/`; build output is emitted to `dist/` after `npm run build`.
- Keep new feature modules co-located with related hooks/components to reduce cross-file churn.

## Build, Test, and Development Commands
- `npm run dev` (alias `npm start`): start Vite dev server with HMR.
- `npm run build`: type-check via `tsc`, then produce a production bundle with Vite into `dist/`.
- `npm run preview`: serve the built bundle locally to verify production output.

## Coding Style & Naming Conventions
- TypeScript throughout; prefer explicit prop and state types to avoid `any`.
- Use 2-space indentation, single quotes, and trailing semicolons for consistency with existing files.
- Components and hooks are PascalCase (e.g., `CoordinateForm`), utility functions camelCase.
- Keep side effects isolated inside `useEffect`; derive view state from props/data instead of mutable globals.
- Internationalization: add new copy into `src/i18n/resources.ts` and reference keys via `useTranslation`.

## Testing Guidelines
- No automated tests are present yet; when adding, prefer Vitest + React Testing Library to match Vite/React stack.
- Co-locate test files as `<name>.test.tsx` near the code under test; favor behavior-focused tests over implementation details.
- Ensure any new feature includes at least smoke coverage for rendering and critical conversions.

## Commit & Pull Request Guidelines
- Follow concise, descriptive commit subjects in imperative mood (e.g., `Add coordinate format selector`, `Fix WGS84 conversion`).
- For pull requests, include: what changed, why, test notes or manual steps (commands run, browsers checked), and any screenshots for UI tweaks.
- Link related issues or tickets when available; flag breaking changes or new configuration requirements in the description.

## Security & Configuration Tips
- Avoid committing secrets; keep API keys or tokens in environment variables and document any required env keys in the PR.
- Validate user-provided coordinates before conversion to prevent runtime errors; prefer graceful error messages over silent failures.
- Before deploying, run `npm run build` and a local preview to ensure translations and theming render correctly in production mode.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phobal)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/phobal)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
