## memora

> - Workspace managed with `pnpm` (see `pnpm-workspace.yaml`).

# AGENTS.md

## Repository overview

- Workspace managed with `pnpm` (see `pnpm-workspace.yaml`).
- Primary package: `@memora/web` in `packages/web`.
- Frontend stack: React 19, Vite (rolldown-vite), Tailwind CSS v4.
- Routing uses `react-router` plus `vite-plugin-route-builder`.

## Language and interface copy

- Use direct, natural language. Avoid inflated wording and development slang in user-facing communication.
- Do not write redundant eyebrow labels. Remove them instead of changing them to sentence case.
- For necessary interface labels, use sentence case. Do not use all-uppercase section labels, metric labels, status chips, tabs, buttons, or navigation items, and do not use letter-spaced uppercase text to create hierarchy.
- Preserve canonical capitalization for technical acronyms such as OCR, ASR, LLM, OPFS, and API.

## Commands (build/lint/test)

### Workspace basics

- Install deps: `pnpm install`
- Run commands for a package: `vp run @memora/web#<command>`
- Run the web app with workspace dependencies: `vp run -t @memora/web#dev`
- Run the marketing site (`@memora/site`, port `9004`): `vp run @memora/site#dev`
  - It renders real `@memora/web` components from source on mock data; `@/livestore/store` is aliased to a stub, so only embed components that don't need the store.
- Build the web app with workspace dependencies: `vp run -t @memora/web#build`

### Development

- Dev server with workspace dependencies: `vp run -t @memora/web#dev`
  - Vite runs on port `9003` (see `packages/web/vite.config.ts`).
- Preview build: `vp --filter @memora/web preview`

### Build

- Build web app and declared workspace dependencies: `vp run -t @memora/web#build`
- Vite Task follows `workspace:*` dependencies before running the web build.

### Lint

- Lint web app: `pnpm --filter @memora/web lint`
  - Oxlint configured for TypeScript and React rules via `packages/web/.oxlintrc.json`.

### Format

- Format workspace: `vp fmt .`
- Check formatting: `vp fmt . --check`
  - Oxfmt reads config from the `fmt` block in the root `vite.config.ts`.

### Tests

- Web app tests live under `packages/web/test/<module>/...`.
- Run all web tests: `pnpm --filter @memora/web test`
- Run a single web test file: `pnpm --filter @memora/web test -- test/<module>/<file>.test.ts`

## Code style and conventions

### Formatting

- Use double quotes for strings.
- Use semicolons.
- Use trailing commas where Oxfmt would allow.
- Keep import blocks separated from other code with a blank line.
- Keep JSX wrapped with parentheses and multiline formatting.

### Imports

- Prefer ES module imports.
- Group imports by source:
  1. React/third-party
  2. Internal modules (`./`, `../`)
- Use named imports when possible; default imports for components.

### TypeScript

- `strict: true` and unused checks enabled.
- Avoid `any`; use concrete types or generics.
- Use `type` for unions/intersections, `interface` for object shapes.
- Favor explicit return types for exported functions.
- Use `as const` when creating literal unions.
- Avoid non-null assertions unless a runtime guarantee exists.

### React

- Function components only.
- Keep hooks at the top-level of components.
- Prefer memoized callbacks with `useCallback` when passing down.
- Prefer `useMemo` only for expensive computations.
- Use `StrictMode` (already set in `packages/web/src/main.tsx`).

### Naming

- `camelCase` for variables/functions.
- `PascalCase` for React components/types/interfaces.
- Constants in `SCREAMING_SNAKE_CASE` when truly constant.
- Files: `camelCase.ts` for utilities, `PascalCase.tsx` for components/pages.

### Error handling

- Handle async errors explicitly; fail fast where appropriate.
- Prefer early returns over deep nesting.
- For user-facing errors, return safe defaults or fallbacks.
- Validate external inputs before use (e.g., `Number.isFinite`).

### State and side effects

- Keep derived state out of `useState` when possible.
- Clean up side effects in `useEffect` return functions.
- Avoid mutations of React state arrays/objects.

### CSS/Tailwind

- Use Tailwind utility classes and `tailwind-merge` helpers where needed.
- Reuse shared class strings via helpers in `src/lib` if repeated.

## Releases and versions

- The app version is the build's git short hash (`git describe --always --dirty`), shown in Settings > About.
- For a change people will notice, add a bullet under a new top `## <release date>` section in `packages/web/RELEASE_NOTES.md`, in plain user-facing language. The update dialog shows the sections a user has not seen yet.
- The service worker waits for the user to accept an update from that dialog; it does not activate new builds on its own.
- Third-party runtime files copied into the build (ONNX Runtime, VAD, sqlite-vec) are served from `/vendor/<package>@<version>/`. Reference them through `__VENDOR_ASSETS__`, not fixed paths.

## Generated files

- `packages/web/src/generated-routes.ts` is generated by the route builder.
- Do not edit generated files directly.

## Vite and build notes

- Vite uses React Compiler via `babel-plugin-react-compiler`.
- Static assets (VAD worklets, ONNX, WASM) are copied via `vite-plugin-static-copy`.
- Build targets `esnext` with sourcemaps enabled.

## Workspace instructions (from .github/instructions)

### Project goals

- Multi-modal learning content: documents, audio, images, video.
- Emphasis on local-first privacy and AI processing.
- React + Tailwind + Vite frontend.

### Dev tips

- Use `vp run @memora/<project_name>#<command>` for package-specific runs.
- Run `vp install` to ensure tools can see packages.
- Confirm package names in each `package.json` (top-level name is not used).

### Base UI

- Base UI docs live at `https://base-ui.com/react/...` and cover:
  - Accessibility, styling, animation, composition, TypeScript usage.
  - Use these docs when working with `@base-ui/react` components.

## Agent-specific guidance

- Keep changes small and localized.
- Match existing formatting and component patterns.
- Avoid introducing new dependencies without discussion.
- Update this file if new lint/test/build commands are added.

## Agent skills

### Issue tracker

Issues and specs are tracked in GitHub Issues for `MaxtuneLee/memora`. See `docs/agents/issue-tracker.md`.

### Triage labels

GitHub issues use the canonical `needs-triage`, `needs-info`, `ready-for-agent`, `ready-for-human`, and `wontfix` labels. See `docs/agents/triage-labels.md`.

### Domain docs

This repository uses a single-context domain documentation layout. See `docs/agents/domain.md`.

<!--VITE PLUS START-->

# Using Vite+, the Unified Toolchain for the Web

This project is using Vite+, a unified toolchain built on top of Vite, Rolldown, Vitest, tsdown, Oxlint, Oxfmt, and Vite Task. Vite+ wraps runtime management, package management, and frontend tooling in a single global CLI called `vp`. Vite+ is distinct from Vite, and it invokes Vite through `vp dev` and `vp build`. Run `vp help` to print a list of commands and `vp <command> --help` for information about a specific command.

Docs are local at `node_modules/vite-plus/docs` or online at https://viteplus.dev/guide/.

## Built-in Commands vs Scripts

`vp <name>` runs a built-in command. `vp run <name>` runs a `package.json` script or a `vite.config.ts` task. Scripts cannot overwrite built-ins, so `vp dev` and `vp run dev` may do different things. Check `package.json` and `vite.config.ts` first, and run `vp run <name>` when the project defines a script or task with that name.

## Tool Versions

Run `vp toolchain` to show versions and relationships in the active Vite+
release. Add a tool name to select part of the graph. For example, run
`vp toolchain vite`. Use `--global` to ignore the local `vite-plus` package. Use
`vp why <package>` to show the package-manager dependency graph.

## Review Checklist

- [ ] Run `vp install` after pulling remote changes and before getting started.
- [ ] Run `vp check` and `vp test` to format, lint, type check and test changes. (notice: you don't need to run full test suite after changing anything, just run related test)
- [ ] Check if there are `vite.config.ts` tasks or `package.json` scripts necessary for validation, run via `vp run <script>`.
- [ ] If setup, runtime, or package-manager behavior looks wrong, run `vp env doctor` and include its output when asking for help.

<!--VITE PLUS END-->

---
> Source: [MaxtuneLee/memora](https://github.com/MaxtuneLee/memora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-06 -->
