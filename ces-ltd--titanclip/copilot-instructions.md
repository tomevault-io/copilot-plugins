## titanclip-monorepo

> TitanClip pnpm monorepo layout, scripts, and scoping


# Monorepo and tooling

- Use **pnpm** only (`packageManager` in root `package.json`). Node **>= 20**.
- Internal packages use **`workspace:*`** for `@titanclip/*` dependencies.
- Common commands from repo root: `pnpm install`, `pnpm run typecheck`, `pnpm run build:all`, `pnpm run dev` (Electron).
- Scope work with **`pnpm --filter <package-name> <script>`** (e.g. `@titanclip/server`, `@titanclip/ui`).
- **Own the package you touch**: `server/`, `ui/`, or a path under `packages/`. Do not edit `dist/`, `release/`, or other generated artifacts unless the task explicitly requires it.
- Vitest is configured via root `vitest.config.ts` as a multi-project workspace. If a listed project is missing locally, run tests from the package that defines them (`pnpm --filter <pkg> exec vitest run` or the package’s script).

---
> Source: [CES-Ltd/TitanClip](https://github.com/CES-Ltd/TitanClip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
