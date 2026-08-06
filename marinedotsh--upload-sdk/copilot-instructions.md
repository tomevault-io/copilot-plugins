## upload-sdk

> This repository is in active development and is not publicly published.

# Repository Guidelines

## Project Structure & Module Organization

This repository is in active development and is not publicly published.
`packages/upload-sdk` has no external users yet, so prioritize clean iteration
over backward compatibility for existing consumers.

This is a pnpm workspace managed with Turbo. The SDK lives in
`packages/upload-sdk`, with TypeScript source under `packages/upload-sdk/src`.
Provider implementations are in `src/providers`, validation helpers in
`src/validation`, and public exports are collected through `src/index.ts`.

The demo app lives in `apps/demo`. Demo API routes are under
`apps/demo/src/app/api`, server-side upload setup is in
`apps/demo/src/server`, and UI files are in `apps/demo/src/app`.
Repository-level docs are in `docs`, including the Git convention reference.

## Build, Test, and Development Commands

- `pnpm install`: install workspace dependencies using `pnpm-lock.yaml`.
- `pnpm dev`: run all package/app development tasks through Turbo.
- `pnpm build`: build the SDK and demo, producing package `dist` output.
- `pnpm test`: run workspace test tasks. The SDK currently maps tests to TypeScript checking.
- `pnpm typecheck`: run TypeScript checks across the workspace.
- `pnpm check`: run Biome checks at the repository root.
- `pnpm check:fix` or `pnpm format`: apply Biome fixes/formatting.
- `pnpm --filter @marinedotsh/upload-sdk publish:check`: verify package build, types, exports, and pack output.

## Coding Style & Naming Conventions

Use TypeScript and ESM modules. Biome is the source of truth for formatting:
2-space indentation, 100-character line width, double quotes, and semicolons as
needed. Prefer small, typed modules with explicit exports. Use descriptive
kebab-case filenames such as `create-uploader.ts` or `upload-input.ts`; reserve
PascalCase for React components in the demo.

## Testing Guidelines

There is no separate unit test suite yet. For SDK changes, run
`pnpm --filter @marinedotsh/upload-sdk typecheck` and `pnpm --filter @marinedotsh/upload-sdk build`.
Add focused tests when introducing behavior that can regress, and place them near
the code they cover using a clear `*.test.ts` naming pattern if a test runner is
introduced.

## Commit & Pull Request Guidelines

Follow `docs/GIT_CONVENTION.md`: use `<type>(<scope>): <summary>` with a required
scope, for example `feat(upload-sdk): add typed upload providers`. Keep commits
to one logical change.

PRs should include a short summary, verification commands run, linked issues when
relevant, and screenshots for demo UI changes. Note public API changes,
configuration requirements, or publishing risks explicitly.

## Security & Configuration Tips

Do not commit provider credentials, private bucket names, or `.env*` secrets.
Keep upload validation strict and mirror realistic configuration only through
local demo environment variables.

---
> Source: [marinedotsh/upload-sdk](https://github.com/marinedotsh/upload-sdk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-28 -->
