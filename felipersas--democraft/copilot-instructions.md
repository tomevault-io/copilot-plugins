## democraft

> Workspace instructions for ZCode agents working in the **Democraft** monorepo.

# AGENTS.md

Workspace instructions for ZCode agents working in the **Democraft** monorepo.
Read this first, then `DEVELOPMENT.md` and `llms.txt` for depth.

## What this repo is

Democraft turns real browser workflows into deterministic Remotion videos. You
author one TypeScript demo (`demo.ts`); Playwright captures the target app;
the timeline resolver + Remotion renderer produce an MP4. **One authoring API
serves both humans and LLMs** — there is no separate agent format.

Pipeline: `demo.ts → compile → DemoIR → Playwright capture → manifest → resolveTimeline → RenderTimeline → Remotion → MP4`.

## Layout

- `packages/*` — published libraries (`@democraft/schema|core|compiler|playwright|timeline|preview|remotion|testing|cli|studio|authentication`). Each builds with **tsup** to `dist/`.
- `apps/studio` — Next.js 15 Studio runtime (preview/edit/render), installed internally by the CLI.
- `apps/docs` — Fumadocs documentation site (i18n: `en` + `pt-BR`).
- `apps/landing-page` — marketing site.
- `examples/*` — runnable demos (`demo-app` is the canonical full-pipeline example; serves on `:4173`).
- `docs/` — design bible: `spec/` (intended design), `architecture/` (how code works today, cites `file:line`), `adr/`, `guides/`, `process/`.
- `.democraft/` — **generated runtime artifacts** (captures, manifests, recordings, studio-data). Gitignored. Never hand-author.

## Commands

Canonical verification (run before submitting changes):

```bash
make check     # lint + typecheck --force + test --force + build --force + git diff --check
```

Day-to-day from repo root:

```bash
pnpm build         # turbo build (all packages)
pnpm typecheck     # turbo typecheck
pnpm test          # turbo test --concurrency=4 (vitest)
pnpm lint          # eslint .
pnpm format        # prettier --check . (use --write to fix)
```

Target a single package:

```bash
pnpm --filter @democraft/<name> typecheck
pnpm --filter @democraft/<name> test
pnpm --filter @democraft/<name> build
```

## Critical gotcha: workspace packages resolve from `dist/`

Workspace packages are consumed via their built `dist/` output (tsup, ESM + DTS).
**You must run `pnpm build` once after `pnpm install`**, and rebuild a package
after editing its source for downstream consumers to pick up the change.

`turbo` (used by `pnpm build` / `typecheck` / `test`) tracks the dependency
graph and rebuilds upstream packages automatically — so `pnpm typecheck` from
the root usually "just works". When debugging a single package in isolation,
rebuild its upstream deps manually.

## Architecture boundaries (load-bearing)

- **`schema` is the leaf** every other package depends on. JSON-serializable types + Zod schemas only.
- **`core`** is the thin authoring API (`defineDemo`, `defineTargets`, `byRole/byLabel/byTestId/byText`). No I/O.
- **`compiler` owns the IR.** Never hand-author `DemoIR` — author `demo.ts` and let `compileDemo` produce it.
- **`remotion` does NOT depend on `compiler` or `playwright`.** It consumes only manifest + timeline JSON. This keeps the renderer cacheable and decoupled — preserve it.
- **`cli` is the only package that depends on every other workspace package.**
- **`studio`** depends on compiler/core/playwright/remotion/schema/timeline/authentication.
- Capture (Playwright, needs the app running) is separated from preview (pure disk, no app). The Studio works fully from cached `.democraft/studio-data/`.

Full graph and per-package API surface: `docs/architecture/overview.md`.

## Authoring rules (for `demo.ts` edits)

1. Prefer **semantic target IDs** and resilient locators: `byRole` > `byLabel` > `byTestId` > `byText`.
2. Run `pnpm exec democraft validate demo.ts` before capture/render and repair every `DCxxxx` diagnostic.
3. Prefer high-level `democraft render` / `democraft studio` over manual artifact pipelines.
4. Use `--json` for machine-readable inspection, target lists, and diagnostics.
5. Treat `.democraft/` as generated. Presentation-only edits (captions, camera, pacing) reuse capture; changed browser actions or locators require a new capture.

## Conventions

- **Prettier**: `semi: true`, **double quotes**, `trailingComma: "all"`. (Root `.prettierrc.json`.)
- **ESLint**: flat config (`eslint.config.mjs`), `js.recommended` + `tseslint.recommended`. Ignores `dist/`, `.next/`, `.turbo/`, `next-env.d.ts`.
- **TypeScript**: `strict`, ES2022, ESNext modules, Bundler resolution. Path aliases (`@/components/*`, `@/lib/*`, etc.) are defined **only at root `tsconfig.json`** and point at `packages/remotion/src/*` — they exist for the Remotion package, not as a general convention.
- **Barrel exports**: each package has `src/index.ts` re-exporting focused modules. Keep the barrel complete when adding files.
- **Docs i18n**: keep English and pt-BR documentation trees in sync. Public API names (`defineDemo`, `byRole`, `RenderTimeline`, …) stay in English in both locales.

## Studio specifics

- Next.js 15 App Router, binds to **loopback only** (`127.0.0.1`).
- Mutation endpoints require an **ephemeral session token** minted by the CLI — don't call them directly without going through the CLI launch flow.
- Studio data dir overridable via `DEMOCRAFT_STUDIO_DATA`.
- Launch via the CLI, not `next dev`: `pnpm exec democraft studio examples/demo-app/src/demo.ts`.

## Release tooling

Releases use **one root `Makefile`**, not per-package scripts:

```bash
make pack                                       # build + pnpm pack all public packages → .artifacts/npm/
make publish-dry-run                            # check + dry-run every package
make publish CONFIRM=publish TAG=beta           # real publish — requires CONFIRM=publish AND a clean worktree
make release-beta-prepare SELECTED="studio cli" # bump prerelease versions for selected packages
```

Read `docs/releasing/npm.md` before publishing. `.artifacts/` is gitignored.

## Code intelligence

The repo is indexed with **codegraph** (`.codegraph/`, gitignored). For symbol
lookups and call-graph traversal prefer it over ad-hoc grep loops. See
`DEVELOPMENT.md` → "Code intelligence" for the CLI cheatsheet.

## Read before touching sensitive areas

- `docs/architecture/overview.md` — package graph and public API surfaces (cites `file:line`).
- `docs/architecture/pipeline.md` — the five on-disk caching boundaries.
- `docs/authentication.md` — auth profiles, Studio/CLI security model, agent-safe JSON contracts.
- `docs/releasing/npm.md` — before any release.
- `llms.txt` — compact machine-oriented reference.

---
> Source: [felipersas/democraft](https://github.com/felipersas/democraft) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-16 -->
