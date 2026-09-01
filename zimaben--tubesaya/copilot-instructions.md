## tubesaya

> TubeSaya — Remotion-based video generation app. React/Vite frontend, Remotion Studio for composition/render, json-server (`db.json`) as a lightweight local state store standing in for a real backend.

# CLAUDE.md

## Project
TubeSaya — Remotion-based video generation app. React/Vite frontend, Remotion Studio for composition/render, json-server (`db.json`) as a lightweight local state store standing in for a real backend.

## Environment
- Runs inside WSL2 (Ubuntu) on a Windows host — native Windows is not used for this project.
- Sandbox is enabled for this project. See `.claude/settings.json` for filesystem/network boundaries — respect them; don't suggest commands that write outside `allowWrite` paths.
- Git identity for this repo is scoped locally (personal account/SSH key), separate from other repos on this machine. See `.claude/tasks.md` "Done" for context if auth ever looks wrong again.

## Current tasks
See `.claude/tasks.md` for active work, backlog, and recently completed items. Check it at the start of a session and update it as work completes.

## Architecture
- `remotion/Root.jsx` — composition entry point. Reads `db.json` at `app.installedMacros` to resolve which component + props render for `app.lastOpenMacro`. Has a code-only `DEV_OVERRIDE_MACRO_ID` for local tuning that never touches `db.json`.
- `remotion/compositions/<MacroName>/` — one folder per macro (e.g. `AnimateImage`, `AnimateText`). Each exports a component matching its `COMPONENT_MAP` key in `Root.jsx`.
- `db.json` — source of truth for app settings (video/audio/render config) and installed macro props. Treated as generated/machine state, not hand-authored prose — edit carefully, valid JSON only.
- `src/` — app-side (non-Remotion) React/Vite code. Indexed via `index.<folder>.json` files per folder (see conventions below).

## Macros
Status as of last db.json review — update when a stub gets implemented.

| Macro | Component | Status |
|---|---|---|
| AnimateText | `remotion/compositions/AnimateText/` | Built — supports typewriter animation, single text or `sequence[]` of timed text blocks |
| AnimateImage | `remotion/compositions/AnimateImage/` | Built — `bounceIn` animation implemented; guards against empty `src` (see Gotchas log) |
| MarkerText | `remotion/compositions/MarkerText/` | Built — hand-drawn draw-on circle (`MarkerCircle.jsx`) + marker-styled typewriter reveal (`MarkerTypewriter.jsx`), composed by `MarkerText.jsx`; circle draws first, text starts once it finishes |
| DashboardScreen | — | Stub — `macro: {}` in db.json, no component yet |
| AnimateMap | — | Stub — `macro: {}` in db.json, no component yet |
| SyncText | — | Stub — `macro: {}` in db.json, no component yet |
| VFX | — | Stub — `macro: {}` in db.json, no component yet |

When building a stubbed macro: add its schema to `db.json` under `installedMacros.<Name>.macro`, register the component in `COMPONENT_MAP` in `remotion/Root.jsx`, and add an entry to this table.

## Conventions
- **Per-folder index files**: `src/` folders are documented via `index.<folder>.json` alongside the code. Each entry covers purpose, propsIn, propsOut/emits, dependencies, usedBy (reverse dependency, requires full-tree scan — don't infer per-file), sideEffects, gotchas, and relatedData (link to `db.json` schema where applicable). Check the relevant `index.<folder>.json` before modifying a file in `src/`; update it after.
- **Macro props with a `src` field** (images, etc.) can be empty strings before upload — components consuming them must guard against falsy `src` before passing to Remotion's `<Img>`, which throws on empty/missing src. Don't assume `props.src` is always populated.
- **`staticFile()` vs base64**: `Root.jsx` only resolves `src` through `staticFile()` for relative paths; base64 `data:` URLs are passed through as-is. Preserve this branch if touching prop resolution.
- Line endings: repo uses LF (`core.autocrlf=input`). Don't introduce CRLF.
- `node_modules` is never copied/committed across environments — always fresh `npm install` per environment.

## Gotchas log
- Empty-`src` crash on `<Img>` in animation components that use Remotion's `<Img>` instead of a plain `<img>` fallback — see `.claude/tasks.md` "Done" for the fix pattern (guard clause before animation dispatch, in `AnimateImage.jsx`).

---
> Source: [zimaben/TubeSaya](https://github.com/zimaben/TubeSaya) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
