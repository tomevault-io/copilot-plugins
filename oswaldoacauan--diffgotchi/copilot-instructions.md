## diffgotchi

> Opinionated TUI diff reviewer built with opentui + React.

# Diffgotchi

Opinionated TUI diff reviewer built with opentui + React.

## Quick Reference

```bash
bun install              # install deps
bun run dev              # run from root (unstaged changes)
bun run typecheck        # tsc --noEmit
bun run lint             # oxlint --fix
bun run fmt              # oxfmt --write .
bun changeset            # create a changeset for your PR
bun run version          # apply changesets (CI does this)
```

## Architecture

Monorepo with bun workspaces:

- `packages/cli` — Main TUI app (`@diffgotchi/cli`)
- `packages/web` — Astro/Starlight docs site
- `schemas/` — JSON Schema for config and themes

## Tech Stack

- **Runtime**: Bun
- **TUI**: @opentui/core + @opentui/react (NOT @opentuah — fork, don't use)
- **UI**: React 19 with `jsxImportSource: "@opentui/react"` (box/text/diff/scrollbox = JSX intrinsics)
- **CLI parser**: goke
- **Syntax**: Tree-sitter via opentui (built-in: typescript/markdown, 25 WASM parsers at startup)
- **Linting**: oxlint + oxfmt (NOT eslint/prettier)

## Key Conventions

- `.tsx` = renders UI (has JSX). `.ts` = pure logic, no React
- No `.js` import extensions. No `/index` in import paths — use `@/store` not `@/store/index`
- Versioning via changesets — run `bun changeset` for user-facing changes
- `bun run typecheck` must pass after every change

## React + Zustand Rules

Don't repeat these mistakes:

- **No useEffect for derived state.** Compute during render. `useMemo` for expensive. Never `useEffect` + `setState` to transform data.
- **No useEffect for user events.** Put logic in event handlers, not effects triggered by state changes.
- **Every useEffect dep array must be complete.** Context values (`toast`, `dialog`) captured in closures = must be in deps. `store` (Zustand module ref) is stable, context hooks are not.
- **Clean up timers.** Every `setTimeout`/`setInterval` needs a ref + cleanup on unmount. No fire-and-forget timers that call `setState`.
- **Read store lazily in callbacks.** Command handlers, event handlers = call `store.getState()` at execution time. Don't close over reactive values that cause 20-dep useEffects.
- **Zustand selectors are already atomic.** `useAppStore(s => s.field)` only re-renders when that field changes. Don't "optimize" by grouping into object selectors — that's worse without `useShallow`.
- **TextAttributes are bitfield flags.** Combine with `|`: `TextAttributes.UNDERLINE | TextAttributes.ITALIC`. Parent attributes inherit to children — don't repeat on every child span.
- **No `any` types.** Define interfaces for opentui nodes, event handlers, refs. Even a 3-field inline type is better than `any`.

## opentui Gotchas

Real bugs/API mismatches that burned us:

- `TextAttributes.BOLD` = numeric constant (1), not constructor — `new TextAttributes({bold: true})` crashes
- Box props: `position="absolute"`, `left`, `top`, `zIndex` direct props, NOT inside `style`
- No `onClick` — use `onMouseDown`/`onMouseUp`
- KeyEvent has `.name`, `.sequence`, `.ctrl`, `.shift`, `.option` — no `.input` property
- `RGBA.fromInts()`, `RGBA.fromHex()`, `parseColor()` for colors — no `RGBA.brighten()`
- DiffRenderableOptions: `addedBg`, `removedBg`, `contextBg`, `addedContentBg`, `removedContentBg` — no `addedWordBg`/`removedWordBg`
- `renderer.currentFocusedRenderable` — no `renderer.getFocused()`
- JS/JSX/TS/TSX all map to `"typescript"` for tree-sitter (no separate jsx/tsx grammars)

## Project Structure (cli package)

Concern-based: directory = what file IS. Extension = has UI or not.

```
src/
  main.tsx                # CLI entrypoint, provider tree, git orchestration
  app.tsx                 # Root App component, render-only
  themes/*.json           # 30 built-in theme files

  lib/                    # Pure logic, no React (.ts files)
    git/
      index.ts            # buildGitCommand, execGit, ensureGitRepo, startWatcher
      parse.ts            # Diff parsing, filetype detection, ignored files
    config.ts             # DiffgotchiConfig schema + load/save
    review.ts             # Done tracking + session persistence (.git/diffgotchi-*.json)
    update.ts             # BUILD_META, version check, upgrade
    themes.ts             # Theme resolution, syntax theme generation
    parsers.ts            # WASM tree-sitter parser registration

  store/                  # Zustand state (.ts files)
    index.ts              # App state + actions
    selectors.ts          # Derived hooks (useCurrentFileIndex, useTheme, etc.)

  hooks/                  # Standalone React hooks
    use-commands.tsx       # Command palette registration (23 commands)
    use-keyboard.tsx       # Keyboard handler + navigateHunk
    use-measured-width.ts  # Width measurement with resize

  providers/              # Context providers (state + hook export, minimal UI)
    keybind.tsx            # KeybindProvider + useKeybind
    dialog.tsx             # DialogProvider + useDialog
    command.tsx            # CommandProvider + useCommand
    toast.tsx              # ToastProvider + useToast

  components/
    header.tsx             # File name, status badge, +/- counts
    status-line.tsx        # Keybind hints (responsive), stats, branch
    ui/                   # Reusable UI primitives
      dialog-list.tsx      # Searchable list dialog
      dialog-overlay.tsx   # Modal backdrop + panel
      diff-view.tsx        # Wraps opentui <diff> with theme colors
      highlighted-text.tsx # Fuzzy match highlight rendering
      error-boundary.tsx   # Global error boundary
      toast.tsx            # Toast stack rendering
    dialogs/              # Modal dialogs
      file-picker.tsx      # Fuzzy search file list
      theme-picker.tsx     # Live preview theme switching
      about.tsx            # Version/platform info
      commands.tsx         # Command palette UI
    done.tsx              # All-done celebration view

  util/                   # Pure helper functions (.ts files)
    clipboard.ts           # OSC52 + pbcopy/xclip
    editor.ts              # Open file in $EDITOR
    selection.ts           # Copy selected text from renderer
    fuzzy-match.ts         # fuzzyMatch + fuzzyMatchSingle
    truncated-path.ts      # Path truncation with ellipsis
    status-badge.ts        # File status → badge label + color
    scroll-acceleration.ts # MacOS scroll acceleration wrapper
```

## Config

File: `~/.config/diffgotchi/config.json`

Key fields: `theme`, `keybinds`, `context_lines`, `mouse`, `filetypes` (custom ext→language map), `ignored_files` (regex patterns)

Theme selection persists to config on confirm (not during preview).

## Release Flow

- **Edge**: every push to main → binaries under rolling `edge` GitHub Release tag (`edge.yml`)
- **Stable**: `workflow_dispatch` on `release.yml` → auto-generates changesets from commits → version bump → tag → binaries + GitHub Release + homebrew formula update
- **Web**: `deploy-web.yml` deploys GitHub Pages on push to main

## Git Commands

- Default mode: `git add -N . && git diff` (unstaged, intent-to-add for new files)
- Staged: `git diff --cached`
- Ref comparison: `git diff base...head`
- Uses `spawn` (streaming, no buffer limit) with 50MB soft cap + truncation flag
- Errors show error screen, watcher retries automatically

---
> Source: [oswaldoacauan/diffgotchi](https://github.com/oswaldoacauan/diffgotchi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-20 -->
