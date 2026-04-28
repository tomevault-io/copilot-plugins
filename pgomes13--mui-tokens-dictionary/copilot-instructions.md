## mui-tokens-dictionary

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

### Token pipeline (repo root)
```bash
npm run build        # Run Style Dictionary → produces build/tokens.css, build/tokens.js, build/tokens.d.ts
make build           # Same via Makefile
make clean           # Remove build/
make rebuild         # clean + build
```

### Site (run inside `site/`)
```bash
cd site
npm run dev          # Vite dev server with HMR
npm run build        # TypeScript check + Vite production build → site/dist/
npm run preview      # Serve the production build locally
```

## Architecture

This repo has two independent sub-projects that share the `tokens/` directory as their source of truth.

### 1. Token pipeline (repo root)

`tokens/*.json` → `config.ts` + `scripts/build.ts` → `build/` artifacts

- Token files use the W3C design token format (`$value`, `$type` on every leaf).
- `config.ts` registers a custom Style Dictionary transform group `css/mui` and runs two passes: **light** (outputs `:root` CSS vars + JS/TS exports) and **dark** (outputs palette-only CSS vars under `[data-mui-color-scheme="dark"]`). `scripts/build.ts` merges the two CSS files into a single `build/tokens.css` and deletes the intermediate `build/tokens.dark.css`.
- `size/rem` transform is intentionally excluded — all sizes are already stored as CSS strings (e.g. `"8px"`).
- CSS variable names use the custom `name/mui/kebab` transform: kebab-case, `DEFAULT` path segments stripped, `--mui-` prefix. JS names use PascalCase with `Mui` prefix.
- Only `palette/light.json` and `palette/dark.json` differ between builds; all other token files are shared base tokens.

### 2. Site (`site/`)

React + Vite + TypeScript app deployed to GitHub Pages at `https://pgomes13.github.io/mui-tokens-dictionary/`.

**Data flow:**
- `site/src/tokens/index.ts` imports all token JSON files as static compile-time imports (via Vite's `resolveJsonModule` + `@tokens` alias pointing to `../tokens/`). No runtime fetching for reads.
- `site/src/lib/tokenUtils.ts` provides `flattenTokens` (recursive W3C leaf walker), `toCssVar` / `toJsName` (replicates the pipeline's naming logic), and `groupByCategory`.
- `site/src/hooks/useTokenData.ts` memoizes the flattened/grouped result once on mount.

**Two audience views** toggled in `App.tsx`:
- **Designer view** (`DesignerPage`) — color swatches, typography specimens, spacing bars, simple token tables. All token values and CSS variable names are displayed as `CopyChip` buttons — clicking copies to clipboard, hovering shows "Copy to clipboard". Includes a "Download Tokens Studio JSON" button (`site/src/lib/w3cExport.ts`) that merges all token files into the Tokens Studio envelope format and triggers a browser download.
- **Developer view** (`DeveloperPage`) — filterable table of all tokens with CSS variable name, JS export name, value, and an "Explore" button. Clicking Explore opens `UsageSnippet` as a modal dialog with copy-ready CSS and JS code blocks.

**CopyChip component** (`site/src/components/common/CopyChip.tsx`):
- Shared clickable chip that displays the value as its label (or an optional `label` prop override).
- Copies to clipboard on click, shows "✓ Copied" feedback for 1.5 s, and `stopPropagation` to avoid triggering parent click handlers.
- Props: `value` (string to copy), `label?` (display text override), `mono?` (monospace font).
- CSS class `.copy-chip` uses `display: inline-block; width: auto` so chips are content-width, not column-width.

**Theme toggle:** sets `data-mui-color-scheme="dark"` on `<html>`, persisted in `localStorage`. This is the same attribute MUI's `CssVarsProvider` uses.

**Version footer** (`site/src/components/layout/Footer.tsx`):
- Reads `__APP_VERSION__` (injected at build time via Vite's `define` from the root `package.json` version).
- Renders a centered link to the corresponding GitHub release: `https://github.com/pgomes13/mui-tokens-dictionary/releases/tag/v{version}`.
- Version is injected in `site/vite.config.ts` using `createRequire` to read `../package.json`.
- `site/src/vite-env.d.ts` declares `declare const __APP_VERSION__: string`.

### CI/CD

**Release flow** (push to `main` → semantic-release → site deploy):

1. `.github/workflows/release.yml` — triggers on every push to `main`. Runs `npm ci`, `npm run build` (validates tokens), then `npx semantic-release`. semantic-release analyzes conventional commits, bumps `package.json`, updates `CHANGELOG.md`, creates a git tag, and publishes a GitHub Release with `tokens.css`, `tokens.js`, and `tokens.d.ts` attached. Requires `fetch-depth: 0` for full git history.

2. `.github/workflows/deploy-site.yml` — triggers via `workflow_run` after the "Release" workflow completes successfully (or manually via `workflow_dispatch`). Checks out `main` (with `ref: main` to get the version-bumped `package.json` committed by semantic-release), runs root token build, builds the site, and pushes `site/dist/` to the `gh-pages` branch via `peaceiris/actions-gh-pages`.

3. `.github/workflows/publish-npm.yml` — **manual only** (`workflow_dispatch`). Accepts an optional tag input (defaults to the latest release tag). Checks out that tag, builds tokens, and runs `npm publish`. Requires `NPM_TOKEN` secret in GitHub repo settings.

**Why `workflow_run` (not `push: tags`)?** Workflows triggered by `GITHUB_TOKEN` (e.g., semantic-release's tag push and chore commit) do not cascade to other workflows. `workflow_run` fires after the named workflow finishes, regardless of how it was triggered.

semantic-release is configured in `.releaserc.json` with `npmPublish: false` — publishing to npm is always manual.

After the first deploy, enable Pages in repo Settings → Pages → Source: `gh-pages` branch.

### Commit message convention

| Prefix | Version bump |
|--------|-------------|
| `fix:` | patch — `1.0.0` → `1.0.1` |
| `feat:` | minor — `1.0.0` → `1.1.0` |
| `feat!:` or `BREAKING CHANGE:` | major — `1.0.0` → `2.0.0` |

### npm package

The package is published to npm as `mui-tokens-dictionary`. Consumers install with `npm install mui-tokens-dictionary` and use:
- `import 'mui-tokens-dictionary/tokens.css'` — CSS custom properties
- `import { MuiPalettePrimaryMain } from 'mui-tokens-dictionary'` — JS/TS named exports

The `package.json` `exports` field maps `.` → `build/tokens.js` and `./tokens.css` → `build/tokens.css`. Only the `build/` artifacts are included in the published package (`files` field).

## Skills

Three slash commands are available in `skills/`:

| Skill | Purpose |
|-------|---------|
| `/add-token` | Add a new token to a JSON source file; wires up new category files if needed |
| `/edit-token` | Update `$value` / `$type` on an existing token |
| `/remove-token` | Remove a token; cleans up empty parent objects; greps site for hardcoded references first |

All three run `npm run build` + `cd site && npm run build` and remind the user to commit and push.

## Token file conventions

- Sizes are CSS strings (`"8px"`, `"1rem"`) — do not use raw numbers.
- Only palette tokens belong in `light.json` / `dark.json`. Everything else goes in a base file.
- To add a new base token category, create `tokens/<name>.json` and add its path to `baseTokenSources` in `config.ts` and to `rawTokenFiles` in `site/src/tokens/index.ts`.

---
> Source: [pgomes13/mui-tokens-dictionary](https://github.com/pgomes13/mui-tokens-dictionary) — distributed by [TomeVault](https://tomevault.io/claim/pgomes13).
<!-- tomevault:4.0:copilot_instructions:2026-04-17 -->
