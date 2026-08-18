## mdflow

> A fast, lightweight markdown editor built with Tauri 2 + Rust. **Clean-room rewrite**

# CLAUDE.md

## What is mdFlow

A fast, lightweight markdown editor built with Tauri 2 + Rust. **Clean-room rewrite**
— independent, MIT-licensed. Same eventual feature set as the Kaelio editor, but
written from scratch with a modular architecture and refined UI. License: MIT.
Identifier: `com.kael.mdflow`. Current status: M1 + shell Phase 5 + AI/render/export
Phases 6–7 implemented.

**Always read `docs/spec.md` and `docs/tasks.md` before starting work.**

## Clean-room rule (do not violate — this is the legal basis of the project)

mdFlow must stay legally independent of Kaelio / mx / Vibery Studio (those are
GPL-3.0). Therefore:

- **Never copy, paste, or port code or CSS from Kaelio.** Not a single line.
- Kaelio (`/Users/cong.bui/Kael/20-Projects/kaelio`) may be **read only as a behavior
  reference** — to learn *what* a feature does and *how it behaves* — then a fresh
  implementation is written from understanding, not from the source text.
- No names "mx", "Vibery", "Kaelio", or attribution to them anywhere in source, UI,
  or docs.
- This rule applies to every contributor, human or AI (including Codex).

### How to reference Kaelio correctly

When you want a feature to behave like Kaelio's: open the relevant Kaelio file by
absolute path, study the behavior, describe it in your own words / as a spec, then
implement fresh in mdFlow. Do not keep Kaelio source open while typing mdFlow code,
and never paste from it. Do not add Kaelio as a submodule or copy its files in.

## Working style (from global rules)

- Keep it simple. No premature abstraction. Small, focused files (one responsibility).
- Spec-driven: spec → plan → build → test → document, in order.
- Solo dev — favor low-risk, incremental, always-shippable steps.
- Any bug fix must update `CHANGELOG.md` in the same change, unless the user
  explicitly says not to.

## Commands

- `npm install` — install JS deps (run `npm approve-scripts esbuild fsevents`
  once after install; their native postinstalls are gated by the script allowlist)
- Build/test/check commands are the standard ones — see `package.json` scripts and
  `src-tauri/` for cargo. `npm run test` covers pure functions only.

## Architecture

Tauri 2 + Rust native shell; plain-TypeScript frontend (no framework) wired by a
thin `main.ts`. One responsibility per file.

Data flow: edit → 300ms debounce → `renderMarkdown` → preview pane + word count.
`Cmd+S` → `saveFile` → IPC `save_file`. `Cmd+O` → dialog → IPC `read_file` → editor.
View mode + zoom persist to `localStorage` (`mdflow.ui`).

Native menu contents live in `menu.rs`. Two non-obvious conventions: the View menu's
Soft Wrap and Font/Size/Theme submenus drive `settings.json` (and `sync_view_menu`
reflects settings back into the menu checks), and the command/file palette is **`⌘K`
only** — `⌘P` is the preview toggle.

mdFlow supports multiple independent native Tauri windows. View ▸ New Window and the
macOS Dock context menu create fresh workspaces; menu and Finder-open events route to
the focused native window. Only the original `main` native window writes the persisted
restore-session snapshot. Each native window can still use the in-window Main/Sub split.

The macOS bundle declares Markdown, plain-text, and PDF document types. Finder opens
arrive through Tauri's opened event, are queued until the frontend listener is ready,
then use the same document-opening workflow as File ▸ Open. The application menu can
register mdFlow as the Markdown/text editor or PDF viewer through LaunchServices.

Special-pane document types: `.excalidraw` files open as an editable Excalidraw board
(single pane, React-isolated); `.mind` files open as an editable jsMind mindmap board
(single pane, drag-drop node editing, similar layout shape). Mindmap nodes support
per-node shape (rect/rounded/pill/circle), fill/text color, font size, and bold, stored
in node `data` and persisted in the `.mind` file (no format change). Dedicated activity-bar
buttons create untitled boards of either type. jsMind nodes and connector lines derive
their colors from the active mdFlow theme and update when the theme changes. Mindmaps
support transient multi-selection via shift/⌘-click or marquee drag; Delete/Backspace
and the toolbar Delete action remove the selected subtrees without changing the
`.mind` format.

Editor settings live at `<app config dir>/settings.json`; AI providers and terminals
live at `<app config dir>/agent.json` (legacy `ai.json` is auto-migrated on first
run), while API keys live in the OS keychain and never in that file. The Gear button opens either file as a normal tab, and saving applies
the relevant configuration. Agent settings are split into CLI Agents and Models
(local and hosted) tabs; CLI Agent rows can be edited inline.

Keyboard shortcuts are fully customizable. `keymap.ts` is the command registry
(ids match the native menu item ids in `menu.rs`); overrides persist in
`settings.json` under `keymap`. The settings panel's **Keys** tab (also View ▸
Keyboard Shortcuts) lists every command grouped by category with record / reset /
Restore Defaults. Menu-scoped accelerators are pushed to the native menu at
runtime via the `set_accelerators` command (`item.set_accelerator`); app-scoped
shortcuts (palette, AI send) and the JS fallback handlers match against the
resolved keymap.

The AI Chat tab accepts file references via a 📎 picker, drag-drop onto the panel,
or `@`-mention (fuzzy picker over the open folder). CLI agents receive the file
**paths** (and run with cwd = the open folder, so they can read attachments and
write output files); HTTP models get text-file contents inlined and skip
binary/image files. The Terminal tab has a live terminal picker; terminals (and
their font/size) are edited in the settings Agent ▸ Terminals section. `ai_run`
takes an optional `cwd`.

The AI panel also offers inline **quick actions** (Proofread / Rewrite / Summarize
/ Generate outline) from the command palette (`⌘K`) and the editor right-click
menu. Edit-kind actions (Proofread, Rewrite) replace the selection — or the whole
document when nothing is selected — through the source-bound diff review
(`src/ai/edit-review.ts`); chat-kind actions (Summarize, Generate outline) stream
into the panel. `src/ai/quick-actions.ts` holds the catalog + a DOM-free runner.
Before an AI edit lands on a saved document, a labeled snapshot is written via the
Recovery store (revert from Version History). After a CLI agent run in the open
folder, a changed-files summary (`src/ai/run-summary.ts`) lists added/modified/
deleted files. The model used for quick actions is `quickActionProvider`
(Agent ▸ Models; defaults to the default provider).

Document PDF/DOCX export requires Pandoc and Typst:

```bash
brew install pandoc typst
```

Export is document-aware: Markdown/plain documents offer rendered PDF/DOCX plus
PNG/SVG; HTML and Excalidraw offer PNG/SVG; mindmaps offer PNG; PDF is read-only.
Document export renders KaTeX and Mermaid before conversion. DOCX rasterizes SVG
diagrams as a best-effort compatibility step while retaining MathML for equations.

Heavy render/terminal engines are loaded on demand so the startup bundle remains
below the release chunk-warning threshold.

PDF OCR runs fully offline and Excalidraw is pinned as a vendored board-only module;
both live under `public/vendor/` and React stays isolated from the plain-TypeScript
shell. Rebuild steps: see the `vendored-assets` skill.

Updater runtime and UI are active. Gear ▸ General offers Manual (no background
checks) and Automatic (at most once per 24 hours); both modes always prompt before
installing, and manual checks remain available. Production release checks still
require the signed release feed (`latest.json`) and public key in `tauri.conf.json`;
keep the matching private signing key only in CI secrets.

---
> Source: [kael-wanderer/mdFlow](https://github.com/kael-wanderer/mdFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
