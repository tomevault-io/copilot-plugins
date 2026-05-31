## pi-desktop-old

> This file governs how AI agents work on the `pi-desktop` codebase.

# Pi Desktop — Agent Rules

This file governs how AI agents work on the `pi-desktop` codebase.

## Project Context

- **Renderer**: React 19 + Vite + TypeScript + shadcn/ui (base-nova preset)
- **Shell**: Electron 41 with `electron-vite` build system
- **Styling**: Tailwind CSS v4 + `tw-animate-css`
- **Icons**: `@tabler/icons-react`
- **Package manager**: pnpm
- **Main/preload**: TypeScript, compiled to `out/` by `electron-vite`

## Absolute Rules

### 1. Never auto-run the project
- **DO NOT** run `pnpm run dev`, `pnpm run build`, or any Electron launch command unless the user explicitly asks.
- **DO NOT** restart the Electron app after every file change.
- Only build or launch when the user says "start", "run", "launch", or "build and test".

### 2. Never auto-git-push
- **DO NOT** `git add`, `git commit`, or `git push` unless the user explicitly requests it.
- It is fine to check `git status` or suggest a commit message, but never execute the push.

### 3. Never auto-install dependencies
- **DO NOT** run `pnpm install` or `pnpm add` unless:
  - The user asks for a specific package, OR
  - A `package.json` change was made and the user says "install".
- Never install packages speculatively.

### 4. Prefer editing over rewriting
- Use `edit` for precise changes. Only use `write` for new files or complete rewrites.
- Keep `oldText` as small as possible while still unique.

### 5. Respect the icon library
- This project uses **Tabler Icons** (`@tabler/icons-react`).
- **Never** import from `lucide-react`. Convert any `lucide` icons to their Tabler equivalents.

### 6. Respect the framework
- This is a **Vite SPA**, not Next.js.
- **No** `"use client"` directives.
- **No** `next/head`, `next/image`, or App Router conventions.

### 7. Security boundaries
- Renderer code **never** imports Node/Electron APIs directly.
- All main-process access goes through the typed preload bridge (`window.electron`).
- Preload uses `contextBridge.exposeInMainWorld` — never expose raw `ipcRenderer`.

### 8. Type safety
- Run `pnpm run typecheck` after non-trivial changes.
- All IPC payloads must have TypeScript types in `types/electron-api.ts`.

## Workflow

1. Read the relevant files first.
2. Make focused edits.
3. Typecheck if types changed.
4. Wait for user confirmation before running or pushing.

---
> Source: [abboskhonov/pi-desktop-old](https://github.com/abboskhonov/pi-desktop-old) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-31 -->
