## dsh-whale-pet

> - `src/index.ts` is the intentionally minimal Harness host entry point.

# Repository Guidelines

## Project Structure & Module Organization

- `src/index.ts` is the intentionally minimal Harness host entry point.
- `src/client/` contains the React UI, state mapping, activity summaries, drag behavior, preferences, localization, styles, and assets.
- `src/client/idle-animation.json` is the single source for idle asset names, frame order, and interaction timing; `useIdleBehavior.ts` owns the idle reducer and transitions.
- `tests/*.spec.ts(x)` holds Vitest unit/component tests; `tests/visual/` contains Playwright fixtures and visual flows.
- `character/` stores immutable source PNGs. Generated WebP inputs live under ignored `character/generated/`; tracked previews are written to `docs/`.
- `scripts/` contains asset, license, bundle, package, and Harness smoke checks. `lib/` is generated.

## Build, Test, and Development Commands

Use Node 22.19+ (or 24+) and pnpm 11.19.0.

```bash
pnpm install --frozen-lockfile  # install the locked dependency graph
pnpm assets:build               # verify and process artwork
pnpm build                      # bundle host/client code and emit declarations
pnpm test                       # run Vitest once
pnpm test:watch                 # run Vitest interactively
pnpm test:visual                # run Playwright browser tests
pnpm check                      # run the complete required pre-PR gate
```

Install Chromium once with `pnpm exec playwright install chromium`. For bundle or compatibility changes, pack with `npm pack`, then run `node scripts/smoke-harness.mjs --package ./dsh-whale-pet-*.tgz`.

## Coding Style & Naming Conventions

TypeScript is strict and uses ES modules. Follow the existing style: two-space indentation, single quotes, no semicolons, trailing commas, and explicit return types for production functions. ESLint enforces type-aware rules, React Hooks, type-only imports, and handled promises; run `pnpm lint`. Use PascalCase for components and types, camelCase for functions and values, and kebab-case for scripts and configuration. Keep `.ts`/`.tsx` import extensions.

## Testing Guidelines

Name unit tests `*.spec.ts` or `*.spec.tsx`; place browser scenarios in `tests/visual/*.visual.spec.ts`. Vitest uses jsdom and Testing Library. Playwright retains failure traces/screenshots. No numeric coverage threshold is configured; add focused regression tests for changed state transitions, preferences, UI behavior, or compatibility contracts.

## Commit & Pull Request Guidelines

Recent history uses concise Conventional Commits-style subjects such as `feat: add ...`, `ci: update ...`, and `release: prepare ...`. Keep each commit scoped and imperative. PRs should complete the repository template: summarize behavior, list verification, and note compatibility impact. Run `pnpm check`; add `pnpm test:visual` for UI/artwork changes and the isolated Harness smoke test for bundle changes. Artwork PRs must update `assets.config.json` and `ASSET_ATTRIBUTION.md` and include reviewed preview changes.

Keep `README.md` and `README.zh-CN.md` concise and equivalent. Durable behavior belongs in README, CONTRIBUTING, SECURITY, or AGENTS documentation; do not commit implementation plans, temporary design notes, or completed task checklists.

## Release Guidelines

- Stable releases use matching `package.json` and `v<version>` Git tag values and publish to npm with the `latest` dist-tag.
- Run `pnpm check`, `pnpm test:visual`, `pnpm audit --prod --audit-level high`, `npm pack`, and the isolated Harness smoke test before tagging.
- Do not create the tag or GitHub Release until npm publishing authority and package-version availability have been verified.
- After release, install the published package into the local `web` profile, inspect `dsh --profile web --dump-config`, and smoke-test the real Web UI.

## Compatibility & Assets

Do not add DOM-selector fallbacks, patch Harness source, or register into the root slot. Preserve the profile-bundle and `shell.overlay` contracts. Never overwrite an existing source PNG; add a new attributed asset with its SHA-256 and compatible redistribution rights.

Keep idle animation selectors inactive while `data-collapsed="true"`. Runtime WebPs have a 700,000-byte hard limit and currently total 679,044 bytes, so run `pnpm assets:check` and review headroom whenever artwork changes.

---
> Source: [Er1c0v0/dsh-whale-pet](https://github.com/Er1c0v0/dsh-whale-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
