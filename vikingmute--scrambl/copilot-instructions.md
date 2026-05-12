## scrambl

> This file gives AI coding agents the project context needed to work safely in this repository.

# Agent Guide

This file gives AI coding agents the project context needed to work safely in this repository.

## Project Overview

Scrambl is a pnpm workspace for a zero-dependency text scramble animation library.

Published packages:

- `@scrambl/core` - core TypeScript animation engine
- `@scrambl/react` - React hook and component adapter
- `@scrambl/vue` - Vue composable and component adapter

Docs site:

- Package: `@scrambl/docs`
- Framework: Astro + Starlight
- Production URL: `https://scrambl.vikingz.me`
- Cloudflare Pages output directory: `docs/dist`

## Repository Layout

```txt
packages/core/   Core engine, charsets, easing, DOM/headless APIs
packages/react/  React hook and ScrambleText component
packages/vue/    Vue composable and ScrambleText component
docs/            Astro/Starlight documentation site
RELEASE.md       Human release checklist
```

## Common Commands

Install:

```bash
pnpm install --frozen-lockfile
```

Verify:

```bash
pnpm run lint
pnpm run build
pnpm run test
```

Build docs for Cloudflare:

```bash
pnpm --filter @scrambl/core build
pnpm --filter @scrambl/docs build
```

Build docs in one line:

```bash
pnpm --filter @scrambl/core build && pnpm --filter @scrambl/docs build
```

## Important Build Detail

`@scrambl/docs` imports the workspace package `@scrambl/core`.

The `@scrambl/core` package entry points resolve to `packages/core/dist`, so a clean environment must build core before building docs. Cloudflare Pages should use:

```txt
Root directory: /
Build command: pnpm --filter @scrambl/core build && pnpm --filter @scrambl/docs build
Build output directory: docs/dist
Production branch: main
Node version: 20 or 22
```

If Cloudflare reports:

```txt
Failed to resolve entry for package "@scrambl/core"
```

the docs build ran before `@scrambl/core` was built.

## Publishing

Use Changesets. Package versions are not force-linked, so release only the packages that actually need a new npm version.

Release flow:

```bash
pnpm changeset
pnpm version-packages
pnpm run lint
pnpm run build
pnpm run test
pnpm release
```

See `RELEASE.md` for the full checklist, including npm verification, GitHub tags, and Cloudflare deployment.

If only `@scrambl/core` changes, it is valid to release `@scrambl/core` alone. React and Vue should only be republished when their own code changes or when you intentionally want their published dependency graph to move to a newer `@scrambl/core`.

## Package Metadata

Keep package metadata aligned:

- Homepage: `https://scrambl.vikingz.me`
- Repository: `https://github.com/vikingmute/scrambl`
- License: MIT
- Public npm access through `publishConfig.access = "public"`

Each npm package has its own README. Those READMEs are the npm package pages, so update them when public APIs or examples change.

## Code Style

- TypeScript first.
- Keep `@scrambl/core` framework-agnostic.
- React-specific code belongs in `packages/react`.
- Vue-specific code belongs in `packages/vue`.
- Docs examples should mirror the real package APIs.
- Prefer small, direct APIs over new abstractions.
- `renderMode: 'cells'` is the most stable visual mode but is best suited to monospace or monospace-like presentation.
- For proportional fonts, prefer `renderMode: 'text'` unless the design explicitly wants a gridded decode look.
- `renderMode: 'auto'` is a safety-first heuristic based on the scramble character set, not a font-aware typography optimizer.

Formatting/linting is managed by Biome:

```bash
pnpm run lint
pnpm run lint:fix
```

## Git Hygiene

Do not commit generated or local state:

- `node_modules/`
- `dist/`
- `.astro/`
- `.turbo/`
- `.wrangler/`
- `*.tsbuildinfo`

Before committing:

```bash
git status --short
pnpm run lint
pnpm run build
```

## Documentation Notes

The docs homepage is custom:

```txt
docs/src/pages/index.astro
```

The Starlight splash page at:

```txt
docs/src/content/docs/index.mdx
```

is intentionally overridden by the custom homepage.

The public logo used by READMEs is:

```txt
https://scrambl.vikingz.me/logo-mark.webp
```

Keep this URL stable because npm and GitHub README rendering depend on it.

---
> Source: [vikingmute/scrambl](https://github.com/vikingmute/scrambl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-11 -->
