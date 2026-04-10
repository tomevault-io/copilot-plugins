## stickman-animation-app

> Keep authored code under `src/`, grouped by feature: shared easing functions in `src/lib/`, React (or vanilla JS) scenes in `src/scenes/`, and helpers in `src/utils/`. Visual assets (sprites, HDRIs, LUTs) live in `assets/`. Tests mirror the tree inside `tests/` so `tests/scenes/wave.spec.ts` verifies `src/scenes/wave.ts`. Treat `docs/` as the home for design references and timing charts.

# Repository Guidelines

## Project Structure & Module Organization
Keep authored code under `src/`, grouped by feature: shared easing functions in `src/lib/`, React (or vanilla JS) scenes in `src/scenes/`, and helpers in `src/utils/`. Visual assets (sprites, HDRIs, LUTs) live in `assets/`. Tests mirror the tree inside `tests/` so `tests/scenes/wave.spec.ts` verifies `src/scenes/wave.ts`. Treat `docs/` as the home for design references and timing charts.

## Build, Test, and Development Commands
Run `npm install` once to sync package versions. Use `npm run dev` for hot-reload authoring, `npm run build` for production bundles, and `npm run preview` to sanity-check the built artifacts. `npm run lint` enforces style, while `npm run test -- --watch` keeps the Jest/Vitest loop active during animation tweaks.

## Coding Style & Naming Conventions
Stick to TypeScript with strict mode enabled, 2-space indentation, and dangling commas in multiline literals. Components and systems use PascalCase (`RippleField`), hooks and utilities use camelCase (`useSpringChain`). Name animation clips and exported constants with suffixes like `*Motion` or `*Timeline` to clarify intent. Always run `npm run lint` or let the pre-commit hook autoformat via ESLint + Prettier.

## Testing Guidelines
Write unit tests with Vitest/Jest, one describe block per behavior and one expectation per scenario. Snapshot tests belong to `tests/scenes/__snapshots__` and must be regenerated via `npm run test -- --updateSnapshot`. Aim for >85% branch coverage on core physics/easing modules; integration tests for rendering artifacts can be lower but must assert key frame markers.

## Commit & Pull Request Guidelines
Follow Conventional Commits (`feat: add bezier stitching`, `fix: clamp delta time`). Keep commits focused and <300 lines where possible. PRs need a concise summary, linked issue or task ID, before/after GIF or FPS capture, and a checklist confirming `npm run lint` + `npm run test` passed. Request review from the module owner listed in `docs/ownership.md` once CI is green.

## Security & Configuration Tips
Keep API keys and service tokens in `.env.local`, never in source; use `env.example` to document required values. If you touch shader compilation or WASM modules, run `npm run verify:webgl` (provided script) to ensure browsers accept the updated binaries. Review Dependabot alerts weekly so animation playback stays fast and safe.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/meghamsh738)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/meghamsh738)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
