## oasis-write

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Commands

```bash
# Development (runs Tauri dev server with hot reload)
npm run tauri dev

# Production build (creates platform-specific installers)
npm run tauri build

# Universal macOS build (Intel + Apple Silicon)
npm run tauri build -- --target universal-apple-darwin

# Frontend only (no Tauri, for quick UI iteration)
npm run dev
```

There are no tests in this project.

## Architecture

Oasis Write is a **Tauri 2 desktop app**: a React frontend communicating with a Rust backend via Tauri `invoke` calls. The frontend has no direct filesystem access — all file I/O goes through Tauri commands.

### Data flow

1. `App.tsx` is the root orchestrator: it holds all application state (selected file, content, dirty state, sidebar visibility, clipboard) and passes handlers down as props.
2. The `Sidebar` switches between two views — `FileTree` (file browser) and `Outline` (heading navigator).
3. `Editor` wraps `TipTapEditor`, which uses `tiptap-markdown` to serialize/deserialize markdown. The editor content is kept as markdown strings in state; React re-renders drive TipTap updates.
4. Auto-save: a 2-second debounced `useEffect` in `App.tsx` calls `invoke("write_file")` after each content change.
5. External file change detection: `useFileWatcher` polls via `FileWatcher.ts` (which wraps a Tauri file-watch event) and compares content hashes (`useFileHashes`) to distinguish self-saves from external edits.

### Key services and utilities

- `src/services/FileWatcher.ts` — wraps the Tauri file-watch plugin into a simple class
- `src/services/ImageResolver.ts` — resolves relative image paths to absolute paths for display inside TipTap
- `src/services/MarkdownLinkResolver.ts` — resolves `[text](link.md#anchor)` links: determines if external URL, same-file anchor, or cross-file markdown link
- `src/utils/hash.ts` — content hashing used by the file-watch dedup logic
- `src/utils/editorHelpers.ts` — TipTap utilities: scroll to heading by slug, extract heading slugs, validate link existence
- `src/extensions/CollapsibleHeading.ts` — custom TipTap ProseMirror extension replacing the default heading node to support collapse/expand toggling

### Theme system

`ThemeContext.tsx` manages themes and fonts. Themes are defined as color objects in `src/themes.ts` and applied as CSS variables on `:root`. The context blends heading colors across h1–h6 using `mixHeadingColor`. Theme and font preferences persist to `localStorage`.

CSS variable names: `--bg-primary`, `--bg-secondary`, `--text-primary`, `--text-secondary`, `--border-color`, `--heading-color-1` through `--heading-color-6`, `--accent-color`, `--code-bg`, `--link-color`, `--app-font-family`, `--code-font-family`.

### File tree filtering

Only `.md`, `.markdown`, and image files (`.png`, `.jpg`, `.jpeg`, `.gif`, `.svg`, `.webp`) are shown in the file tree. The constants and helper functions are in `src/constants.ts`.

### Keyboard shortcuts (defined in App.tsx)

| Shortcut | Action |
|---|---|
| Cmd/Ctrl+S | Save current file |
| Cmd/Ctrl+1 | Toggle file tree (press again to enter zen mode) |
| Cmd/Ctrl+2 | Toggle outline (press again to enter zen mode) |
| Cmd/Ctrl+= / + | Increase font size |
| Cmd/Ctrl+- | Decrease font size |
| Cmd/Ctrl+, | Open settings |
| Cmd/Ctrl+N | New window |

### Releasing

1. Update version in `package.json` and `src-tauri/tauri.conf.json`
2. Tag and push: `git tag -a vX.Y.Z -m "Release vX.Y.Z" && git push origin vX.Y.Z`
3. GitHub Actions builds for all platforms and attaches installers to the release

---
> Source: [Varshneyabhushan/oasis-write](https://github.com/Varshneyabhushan/oasis-write) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-17 -->
