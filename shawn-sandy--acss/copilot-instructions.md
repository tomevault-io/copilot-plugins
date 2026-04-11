## acss

> <!-- OPENSPEC:START -->

<!-- OPENSPEC:START -->
# OpenSpec Instructions

These instructions are for AI assistants working in this project.

Always open `@/openspec/AGENTS.md` when the request:
- Mentions planning or proposals (words like proposal, spec, change, plan)
- Introduces new capabilities, breaking changes, architecture shifts, or big performance/security work
- Sounds ambiguous and you need the authoritative spec before coding

Use `@/openspec/AGENTS.md` to learn:
- How to create and apply change proposals
- Spec format and conventions
- Project structure and guidelines

Keep this managed block so 'openspec update' can refresh the instructions.

<!-- OPENSPEC:END -->

# Repository Guidelines
This monorepo powers the ACSS component system across Vite, Storybook, and Astro surfaces; follow these guidelines when contributing.

## Project Structure & Module Organization
`src/` hosts the Vite demo and shared stories in `src/stories/`. Place reusable library work in `packages/fpkit/`, authoring TypeScript in `src/` and letting builds emit to `libs/`. Astro marketing experiments live in `apps/astro-builds/`, using Astro's `src/{components,layouts,pages}` layout. Static assets belong in `public/`, while repo-level configs (Vite, ESLint, Lerna) sit at the root.

## Build, Test, and Development Commands
- `npm run dev` — Vite playground at http://localhost:5173.
- `npm run storybook` — Storybook on port 6006 for component review.
- `npm run build` — TypeScript project references plus production bundle.
- `npm run lint` — Shared ESLint rules; fix issues with `npm run lint -- --fix`.
- `npm --prefix packages/fpkit run build` — Bundle the library and compress SCSS output.

## Coding Style & Naming Conventions
Write TypeScript with 2-space indentation and single quotes for imports. Components use PascalCase directories and default functional exports; hooks follow `useCamelCase`; colocated tests mirror filenames with `.test.tsx`. SCSS in `packages/fpkit/src/components/` compiles during the build, so keep selectors scoped. `eslint.config.js` enforces hook rules and blocks `console` statements—run the linter early in development.

## Testing Guidelines
Run `npm --prefix packages/fpkit run test` for Vitest + Testing Library. Use `npm --prefix packages/fpkit run test:coverage` before reviews and `npm --prefix packages/fpkit run test:snapshot` after intentional UI updates. Keep assertions user-focused, store snapshots under `__snapshots__/`, and call out meaningful coverage gaps in the PR body.

## Commit & Pull Request Guidelines
Follow the Conventional Commit format from `CONVENTIAL_COMMITS.md`, e.g. `feat(button): add neutral variant`. Keep commits focused with subjects under 50 characters. Pull requests must describe the change, link related tickets, list verification commands, and attach Storybook evidence for UI shifts; wait for green CI before merging.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/shawn-sandy)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/shawn-sandy)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
