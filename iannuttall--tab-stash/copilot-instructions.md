## tab-stash

> - App code lives in [src/](src/):

# Repository Guidelines

## Project Structure & Module Organization
- App code lives in [src/](src/):
  - [src/background/](src/background/) — extension service worker (message routing, DB ops).
  - [src/dashboard/](src/dashboard/), [src/sidepanel/](src/sidepanel/), [src/options/](src/options/), [src/popup/](src/popup/) — UI entry points (Vite HTML + TSX).
  - [src/components/ui/](src/components/ui/) — shadcn/ui primitives (do not modify Radix APIs).
  - [src/shared/](src/shared/) — shared utilities: [db.ts](src/shared/db.ts) (Dexie/IndexedDB), [messaging.ts](src/shared/messaging.ts), [types.ts](src/shared/types.ts), [settings.ts](src/shared/settings.ts).
  - [src/styles/](src/styles/) — Tailwind base styles.
- Build output is written to `dist/` (generated). Do not edit files in `dist/`.
- Static extension manifest: [public/manifest.json](public/manifest.json).

## Build, Test, and Development Commands
Run the following as needed.

Dev server:
```
npm run dev
```

Build:
```
npm run build
```

Preview build:
```
npm run preview
```

Typecheck:
```
npm run typecheck
```

Overlay QA (always do this): after any UI change, open menus/tooltips/dialogs and confirm they have a solid background and render above nearby text. We standardize on `z-[2000]` for dropdowns and `z-[1000]` for tooltips.

Resume/Focus QA: after laptop sleep or when returning to a page, confirm Dashboard and Side Panel automatically refresh (items visible, counts correct). We attach focus/visibility listeners to call `GET_ITEMS`/`GET_TABS_STATUS`; do not remove these hooks.

## Coding Style & Naming Conventions
- TypeScript + React (functional components). Two‑space indentation.
- Components: `PascalCase.tsx` (e.g., [src/sidepanel/ui/SidePanelApp.tsx](src/sidepanel/ui/SidePanelApp.tsx)). Utilities: `camelCase.ts`.
- Use official shadcn/ui components only. Install or update via:
  - `npx shadcn@latest add <component>` (e.g., `npx shadcn@latest add tooltip`)
  - Never hand‑roll replacements under [src/components/ui/](src/components/ui/) unless explicitly requested. Do not fork Radix primitives.
  - If a component needs tweaks, extend via props/classes in feature code; do not edit the generated base files.
- Favor Tailwind utility classes; keep variants compact.
- No direct edits to compiled artifacts or vendor code. Avoid new deps unless necessary.
 - Tooltips: always use shadcn `Tooltip` (no native `title` attributes). Wrap triggers with `asChild` and keep content `bg-popover` with high z-index.

## Testing Guidelines
- No formal test suite yet. Validate via:
  - Type checks: `npm run typecheck`.
  - Manual flows: load `dist/` as an unpacked extension, exercise Dashboard, Side Panel, and background messaging.
- Keep changes small and easy to verify. Prefer pure functions in [src/shared/](src/shared/).

## Overlay QA (Must‑Do)
- After any change to dropdowns, dialogs, selects, or tooltips, verify overlays have a solid background and correct stacking:
  - Content has `bg-background` (or explicit theme color) and `z-[2000]` for menus/dialogs; tooltips `z-[1000]`.
  - No bleed‑through behind overlays in both light/dark themes.
  - Confirm closure on Esc and outside click where appropriate.

## Commit & Pull Request Guidelines
- Commit messages must follow the project format (see `docs/commit-style.md`).
- Use imperative, present tense. Group related changes.
- PRs should include:
  - Summary of changes and rationale.
  - Screenshots/GIFs for UI changes (Dashboard/Side Panel menus, dialogs).
  - Notes on migration or manifest permission changes (if any).
  - Verification steps: commands run and how to load/test the extension.

## Security & Configuration Tips
- Manifest permissions are minimal; justify any additions in PRs.
- Do not store secrets. Persist settings via `chrome.storage.local` only.
- IndexedDB is the primary data store ([src/shared/db.ts](src/shared/db.ts)). Never block the UI with long operations.
- MV3 CSP: Never add inline scripts to HTML entry files. Put logic in TS modules instead. For theme detection, use `initSystemTheme()` from [src/shared/theme.ts](src/shared/theme.ts) and call it in each entry (e.g., [src/sidepanel/main.tsx](src/sidepanel/main.tsx), [src/dashboard/main.tsx](src/dashboard/main.tsx), [src/options/main.tsx](src/options/main.tsx), [src/popup/main.tsx](src/popup/main.tsx)).

## Agent-Specific Notes
- Refresh behavior: UI should re-fetch data on focus/visibility change. Side Panel also polls every ~4s while visible to catch missed tab events (post-sleep). Avoid removing these listeners.

### Icons
Generate extension icons from a 512×512 PNG using ImageMagick:

```
[bash scripts/generate-icons.sh](scripts/generate-icons.sh) /path/to/icon-512.png
```
- Outputs to `public/icons/icon-{16,32,48,128,256,512}.png`. Manifest already references 16/32/48/128.
- After updating icons, rebuild and reload the extension.

---
> Source: [iannuttall/tab-stash](https://github.com/iannuttall/tab-stash) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-03 -->
