## frameplanner2

> - `src/`: Svelte 4 + TypeScript app code. Key areas: `lib/` (shared logic), `stores/`, `utils/`, feature folders (e.g., `bookeditor/`, `filemanager/`, `notebook/`). Entry points: `main.ts`, `App.svelte`.

# Repository Guidelines

## Project Structure & Module Organization
- `src/`: Svelte 4 + TypeScript app code. Key areas: `lib/` (shared logic), `stores/`, `utils/`, feature folders (e.g., `bookeditor/`, `filemanager/`, `notebook/`). Entry points: `main.ts`, `App.svelte`.
- `public/`: Static assets bundled by Vite.
- `test/` and `testdata/`: Vitest helpers and fixtures (`test/setup.ts`, `test/helpers.ts`).
- `dist/`: Build output. Do not edit.
- Config: `vite.config.ts`, `tsconfig*.json`, `tailwind.config.cjs`, `postcss.config.cjs`, `.env*`, `wrangler.toml`, `firebase.json`.

## Build, Test, and Development Commands
- `npm run dev`: Start Vite dev server.
- `npm run dev-https`: Local HTTPS dev using `vite.https.config.ts` and bundled certs.
- `npm run build`: Typecheck and bundle to `dist/`.
- `npm run preview`: Serve the production build locally.
- `npm run check`: Run `svelte-check` type diagnostics.
- `npm run vitest` (or `npx vitest --run`): Run unit tests.
- テスト実行時は `npx tsc --noEmit && npm run vitest -- --run` のように型チェックも併せて行うこと。
- Deploy: `npm run deploy` (publishes via Cloudflare Pages only on `sns_main`).

## Coding Style & Naming Conventions
- TypeScript, 2-space indentation, semicolons optional but be consistent.
- Svelte components: PascalCase (`FramePanel.svelte`). Utilities/stores: camelCase files (`fileSystem.ts`, `uiStore.ts`).
- Path aliases: `$bookTypes/*` → `src/lib/book/types/*`, `$protocolTypes/*` → `src/utils/edgeFunctions/types/*`.
- Keep UI in `.svelte`, business logic in `src/lib/**`, and side effects in `src/utils/**`.
- Styling via Tailwind/PostCSS; prefer utility classes over inline styles.

## Testing Guidelines
- Framework: Vitest with jsdom and polyfills in `test/setup.ts` (import in specs as needed).
- Location: place specs under `test/` or alongside code as `*.test.ts`.
- Write deterministic tests; mock browser APIs and storage via helpers in `test/helpers.ts`.
- Example: `npx vitest --run --reporter=verbose`.

## Commit & Pull Request Guidelines
- Commit style: imperative mood; optional Conventional Commits (`feat:`, `fix:`, `chore:`). Example: `feat(book): add multi-image reference support`.
- Before PR: run `npm run check`, `npm run build`, and tests.
- PRs: include clear description, linked issues (`Closes #123`), and screenshots/GIFs for UI changes.
- Branching: target `sns_main` when preparing a release. Deploys run from `sns_main` via `npm run deploy`.

## Security & Configuration Tips
- Copy `.env.example` → `.env`, fill keys (Firebase, Supabase, Sentry). Never commit secrets.
- HTTPS dev: use `npm run dev-https` with the provided local certs.
- Cloudflare config in `wrangler.toml`; avoid editing prod bindings in PRs without review.

# Notice

- 応答は日本語で
- gitコミットはユーザーが指示するまでするな。指示されてコミットするときはコミットメッセージは英語で
- ビルド・実行は不要
- npm run checkは必要ならしてください
- A11y警告はsvelte-ignoreでOK

---
> Source: [jonigata/FramePlanner2](https://github.com/jonigata/FramePlanner2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
