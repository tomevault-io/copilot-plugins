## slate-project-manager

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Dev (hot-reload: Go restarts on backend change, Vite HMR on frontend change)
wails dev

# Dev (Linux)
wails dev --tags webkit2_41

# Production build (Windows)
wails build

# Production build (Linux)
wails build --tags webkit2_41

# Frontend only (from frontend/)
pnpm dev          # Vite dev server standalone
pnpm build        # vue-tsc typecheck + Vite build
```

No test suite exists yet.

## Architecture

**Wails v2** desktop app: Go backend exposes methods to a Vue 3 frontend via generated bindings.

```
main.go                      # Wails app bootstrap, window config
app.go                       # All Go→frontend bindings (the entire API surface)
internal/
  models/project.go          # Domain structs: Project, Category, Store, Action, Milestone, Comment
  scanner/scanner.go         # Tech auto-detection (walks dirs 3 levels, checks config files + extensions)
  storage/storage.go         # JSON persistence → OS config dir / Slate / projects.json
frontend/src/
  App.vue                    # Root: all reactive state + all Wails API calls live here
  types.ts                   # TypeScript mirrors of Go structs + TECH_COLORS map
  components/
    atoms/                   # BaseBadge, BaseIcon, IconButton
    molecules/               # ActionCard, EmptyState, InfoBar, InlineTagInput, PinnedActions
    organisms/               # ActionsPanel, FileTree, QuickIdeaWidget
    *.vue                    # Feature-level components (modals, sidebar, header, cards)
```

### Data flow

- All state owned by `App.vue`; child components receive props and emit events upward.
- Every mutation calls a Wails binding (e.g. `window.go.main.App.UpdateProject(…)`) which writes through to `projects.json` atomically.
- Wails generates `frontend/wailsjs/go/main/App.js` (and `.d.ts`) from `app.go` at build time — never edit these generated files.

### Key design facts

- **Blank/idea projects** (`isBlank: true`) have no `path`. Guards exist in `app.go` and the frontend to skip scanning them.
- **Editor resolution** (`app.go:editorForProject`): per-category `EditorCommand` overrides global `Settings.EditorCommand`, which defaults to `code`. Template token `{path}` is substituted; otherwise path is appended.
- **Storage path**: `os.UserConfigDir()/Slate/projects.json` — Linux: `~/.config/Slate/projects.json`, Windows: `%APPDATA%\Slate\projects.json`.
- **Tech detection** (`scanner/scanner.go`): walks up to 3 directory levels, skips `node_modules`, `vendor`, `.git`, `dist`, `target`; reads `package.json` deps to distinguish React/Vue/Angular/etc.
- **Accent color** auto-assigned from a 12-color palette cycling on `len(projects) % 12`.
- **Tailwind v4** is used via `@tailwindcss/vite` plugin (no `tailwind.config.js`); utility classes work directly in `.vue` files.
- **Icon keys** for `Action.icon`: `terminal | git | rocket | bolt | code | cog | database | cloud` — defined in `frontend/src/utils/actionIcons.ts`.

---
> Source: [kraibse/slate-project-manager](https://github.com/kraibse/slate-project-manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
