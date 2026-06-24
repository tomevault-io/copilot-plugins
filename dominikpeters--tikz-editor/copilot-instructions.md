## tikz-editor

> A WYSIWYG editor foundation for TikZ with a layered parser -> semantic IR -> SVG pipeline.

# tikz-editor
A WYSIWYG editor foundation for TikZ with a layered parser -> semantic IR -> SVG pipeline.

## Architecture of parsing and rendering
1. Parser (`/Users/dominik/GitHub/tikz-editor/packages/core/src/parser`) parses TikZ into a lossless AST with diagnostics.
2. Semantic evaluator (`/Users/dominik/GitHub/tikz-editor/packages/core/src/semantic`) resolves styles, transforms, coordinates, and path semantics into a scene graph.
3. SVG backend (`/Users/dominik/GitHub/tikz-editor/packages/core/src/svg`) emits pure SVG from scene elements.
4. Render convenience API (`/Users/dominik/GitHub/tikz-editor/packages/core/src/render`) provides end-to-end source -> SVG orchestration.

## Apps

1. Web app (`/Users/dominik/GitHub/tikz-editor/apps/web`)
2. Desktop app (`/Users/dominik/GitHub/tikz-editor/apps/desktop`) built with Tauri

## Capability Matrix
Capabilities are tracked explicitly in:
1. `/Users/dominik/GitHub/tikz-editor/packages/core/src/capabilities/feature-ids.ts`
2. `/Users/dominik/GitHub/tikz-editor/packages/core/src/capabilities/matrix.ts`
3. `/Users/dominik/GitHub/tikz-editor/packages/core/src/capabilities/registries.ts`

Capability drift is CI-gated by `/Users/dominik/GitHub/tikz-editor/test/capabilities.spec.ts`.

## Scripts
1. `npm run typecheck` runs root TypeScript checks (`tsc --noEmit`).
2. `npm run lint:prod` runs ESLint over production app/core code and fails on warnings. It intentionally excludes tests, e2e/profiling harnesses, scripts, config files, and temp repro files.
3. `npm run lint:ci` currently aliases `npm run lint:prod`; use it as the CI lint gate.
4. `npm run lint` runs full-repo noisy ESLint, including the remaining test/tooling/harness warning debt. Use it for audits, not as the production gate.
5. `npm test` runs all vitest suites (`generate:grammar` + `vitest run`).
6. `npm run test:capabilities` runs capability matrix guards only.
7. `npm run test:corpus` runs PGF corpus regression only.
8. `npm run test:e2e` runs web Playwright suites.
9. `npm run test:e2e:ci` runs web Playwright suites with line reporter.
10. `npm run test:desktop:e2e` runs desktop e2e (may skip on unsupported platforms).
11. `npm run build` builds the core parser package.
12. `cd /Users/dominik/GitHub/tikz-editor/apps/web && npm run build` builds the web app.
13. `npm run compare:renderers -- --input path/to/snippet.tex` runs our renderer and a TeX reference render, then writes a comparison manifest.
14. `npm run compare:pgf-docs -- --source-file pgfmanual-en-tikz-paths.tex` renders snippets from one PGF doc source file and writes an `index.html` side-by-side gallery. It generates side-by-side.png files that can be visually inspected for render accuracy.
15. `npm run version:bump -- v0.2.0` updates first-party npm/Tauri/Cargo version metadata to `0.2.0`, updates internal workspace pins, regenerates the npm lockfile, verifies consistency, stages the expected release files, and commits with message `v0.2.0`. It does not create a git tag; tagging is handled by the release GitHub Action. Use `--no-commit` to skip the commit, and `--allow-dirty` only when intentionally working around unrelated local changes.

## Linting policy
Production app/core code should stay warning-free under `npm run lint:prod`. Do not add new warnings there. If you touch test, e2e, profiling, script, config, or temp repro files, prefer cleaning local warnings opportunistically, but those areas are still tracked as full-audit debt under `npm run lint`.

## Corpus Source
The repository includes `pgf-docs/`, a copy of the PGF manual source files. It also includes `pgf-src/`, a copy of the PGF source files. Both should be used to check the intended rendering of TikZ features against the reference implementation. The `pgf-docs/` snippets are also used for testing and capability tracking.

## Testing note
vitest doesn’t support --runInBand in this environment.

## Profiling Scripts
Performance profiling scripts live in `apps/web/profiling/`. They use Playwright + CDP to capture CPU profiles of the running web app. They are manual dev tools, not part of CI.

Run from `apps/web/`:
```
npx playwright test --config profiling/playwright.config.ts profiling/profile-paper-drag.spec.ts
npx playwright test --config profiling/playwright.config.ts profiling/profile-paper-selection.spec.ts
etc.
```

The config builds the app in production mode and serves it on port 4174. Set `TIKZ_PROFILE_VERBOSE=1` for verbose logging. Output `.cpuprofile` and `-report.json` files go to `apps/web/profiling/traces/`.

CPU profiles JSON reports can be inspected directly. To analyze a `.cpuprofile` programmatically:
```
node scripts/analyze-cpuprofile.mjs apps/web/profiling/traces/paper-drag-visible.cpuprofile
```

## Tauri

Tauri docs for the desktop app are available in `tauri-docs/` for local search.

# My approach to git

I sometimes keep "dirty" files in the work tree, especially when they are draft changes, before committing them. Typically the request I make will not touch the same files. So you don't have to worry about changes already in the repo, and certainly there is no need to stop your work once you see such files, because the presence of these files is expected.

---
> Source: [DominikPeters/tikz-editor](https://github.com/DominikPeters/tikz-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-24 -->
