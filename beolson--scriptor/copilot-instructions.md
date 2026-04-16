## scriptor

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Scriptor** is a platform for browsing and running host-specific setup scripts. Scripts are organized by platform (Linux, Mac, Windows) and served via a static documentation site deployed to GitHub Pages.

Monorepo workspaces live under `20_Applications/`:
- `scriptor-web` — static documentation/marketing site (Next.js 16 App Router, deployed to GitHub Pages)
- `scriptor-web-test` — Playwright E2E tests for the web app

Scripts and their spec files live in `scripts/`. Platform metadata lives in `10_Specs/platforms.json`.

## Package Manager

**ALWAYS use Bun.** Never use `node`, `npm`, `npx`, or any other package manager. Every install, script run, and execution must use `bun` or `bunx`.

## Running Scripts

Always run scripts through the root `package.json`. Never `cd` into a sub-project and run scripts from there. Use `bun run <script>` from the repository root.

Root scripts:
```
bun run dev           # turbo dev (all workspaces)
bun run build         # turbo build
bun run lint          # biome check . (all workspaces)
bun run format        # biome format --write . (all workspaces)
bun run typecheck     # tsc --noEmit (all workspaces)
bun run test:unit     # vitest (all workspaces)
bun run test:e2e      # playwright (web only; requires build first)
```

Turbo enforces task dependencies: `typecheck` depends on `build`; `test:e2e` depends on `build`. Running via Turbo handles ordering automatically.

## Pre-Commit Checklist

Before committing any changes, run all of the following and fix any issues:

```
bun run lint
bun run format
bun run typecheck
bun run test:unit
bun run test:e2e
```

Do not commit if any of these fail.

## Linting & Formatting

**Biome only** — no ESLint, no Prettier. Root `biome.json` is extended by both workspaces.

- Indentation: tabs
- Quotes: double quotes in JS/TS
- `bun run lint` → `biome check .`
- `bun run format` → `biome format --write .`
- Biome also enforces import organization (`organizeImports: on`).

## TypeScript

All workspaces use strict mode. `scriptor-web` config: target ES2017, module ESNext, path alias `@/*` → `./` (relative to `20_Applications/scriptor-web/`).

## Web Workspace (`20_Applications/scriptor-web/`)

**Purpose**: Static documentation site deployed to GitHub Pages via `output: "export"` in `next.config.ts`.

**Stack**: Next.js 16, App Router only (no `pages/` directory), React 19, TypeScript.

**Data flow**: `scripts/` at repo root → read at build time by `lib/loadScripts.ts` → passed as props to page components. Platform metadata from `10_Specs/platforms.json` via `lib/loadPlatforms.ts`. Key types: `lib/types.ts`.

**Route structure**:
- `/` → `app/page.tsx` (hero + platform cards)
- `/scripts/[platform]` → `app/scripts/[platform]/page.tsx`
- `/scripts/[...slug]` → `app/scripts/[...slug]/page.tsx` (individual script pages)

**Testing**:
- Unit: co-located `.test.tsx`/`.test.ts` files, run via `bun run test:unit`
- E2E: Playwright in `20_Applications/scriptor-web-test/` (Chromium only). Requires `bun run build` first.

## Web Styling Rules

- **CSS Modules only** — every component has a co-located `ComponentName.module.css`
- **No Tailwind utility classes in JSX** — Tailwind v4 is present via PostCSS but only for its plugin infrastructure. Never use `className="flex gap-4"` or similar.
- **Design tokens** live in `20_Applications/scriptor-web/app/globals.css` as CSS custom properties. Always use `var(--color-accent)`, `var(--gap-lg)`, etc. in CSS modules — never hardcode colors, spacing, or font sizes.
- **Fonts**: IBM Plex Mono (`--font-ibmplex`, body/UI) and JetBrains Mono (`--font-jetbrains`, code). Loaded via `next/font/google` in `app/layout.tsx`.
- **Theming**: Dark/light via `data-theme` attribute on `<html>`. Tokens defined for both themes in `globals.css`. Inline script in `layout.tsx` sets theme before first paint (reads `localStorage`). `<html>` has `suppressHydrationWarning` for this reason.

## Adding a New Script

1. Add the script to `scripts/<platform>/<category>/script-name.sh` (or `.ps1` for Windows).
2. Optionally add a corresponding `.spec.md` file alongside it.
3. The web site picks up the new script automatically at the next build — no code changes needed.

## Release Process

Uses [Changesets](https://github.com/changesets/changesets) for version management.

1. `bun run changeset` — create a changeset file (commit alongside the code)
2. Open a PR and merge to `main`
3. CI auto-creates a "chore: version packages" PR
4. Merging the version PR triggers release:
   - **Web**: Static site built, E2E tested, and deployed to GitHub Pages

## WSL Detection & Headed Browser

### Detecting WSL

Before launching a browser, check if you're running in WSL:

```bash
cat /proc/version        # contains "microsoft-standard-WSL2" if in WSL
echo $WSL_DISTRO_NAME    # non-empty if in WSL (e.g. "Debian13Dev")
```

### Opening a Headed Chrome Browser in WSL

When running in WSL, use the Windows Chrome executable via `PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH` with the `playwright-cli` skill and the `--headed` flag:

```bash
PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH="/mnt/c/Program Files/Google/Chrome/Application/chrome.exe" \
  playwright-cli open --headed https://example.com
```

This launches Chrome as a headed Windows process (visible on the Windows desktop) while being controlled from WSL via Playwright.

## CI/CD

GitHub Actions only. Two workflows:
- `.github/workflows/ci.yml` — on push to `main` and all PRs: build → lint + typecheck + unit tests → E2E
- `.github/workflows/release.yml` — on push to `main`: Changesets → GitHub Pages deploy

CI uses `bun install --frozen-lockfile`. Playwright installed with `bunx playwright install --with-deps chromium`.

<!-- BEGIN:nextjs-agent-rules -->

# Next.js: ALWAYS read docs before coding

Before any Next.js work, find and read the relevant doc in `20_Applications/scriptor-web/node_modules/next/dist/docs/`. Your training data is outdated — the docs are the source of truth.

<!-- END:nextjs-agent-rules -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/beolson) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
