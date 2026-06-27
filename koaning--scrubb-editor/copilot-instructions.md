## scrubb-editor

> A desktop code editor with drag-to-scrub numeric literals.

# scrubb — orientation for Claude

A desktop code editor with drag-to-scrub numeric literals.

## Stack

- Tauri v2 (desktop shell, Rust backend)
- Vite + TypeScript (frontend build & dev server)
- Monaco Editor (the editor surface)
- Vitest (unit tests)

## Where things live

- `src/main.ts` — app orchestrator: wires Monaco, the file tree, tabs, the scrub controller, and the status bar; owns keyboard shortcuts.
- `src/scrub/` — the drag-to-edit-numbers feature.
  - `numberScanner.ts` — finds numeric literals in source text.
  - `decorations.ts` — applies the `scrubbable` Monaco decoration, filtering out tokens that lie inside strings/comments.
  - `scrubController.ts` — mouse handling for the drag interaction.
  - `numberScanner.test.ts` — vitest unit tests for the scanner.
- `src/sidebar/fileTree.ts` — folder browser panel.
- `src/tabs/tabBar.ts` — tab bar for open files; manages Monaco model lifecycle.
- `src-tauri/src/lib.rs` — Rust commands exposed to the frontend (`read_dir`, `read_file`, `write_file`, `get_initial_folder`).
- `src-tauri/tauri.conf.json` — window/title/identifier/bundle settings.

## Tests

- Run with `npm test`.
- Tests live next to source as `*.test.ts`.

## Startup folder

- `get_initial_folder` (Rust) returns the first non-flag argv, falling back to `SCRUBB_INITIAL_FOLDER`, validated as an existing directory.
- `main.ts` calls it once at startup and opens the folder via `FileTree.openDirectory`.
- `npm run tauri:dev:here` sets `SCRUBB_INITIAL_FOLDER=$PWD` so dev launches with the project root pre-loaded.

---
> Source: [koaning/scrubb-editor](https://github.com/koaning/scrubb-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
