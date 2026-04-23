## tadaima

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

tadaima is a [greetd](https://sr.ht/~kennylevinsen/greetd/) greeter library for AGS/GJS. It lets users build custom Linux login screens with GTK4 and TypeScript. The library handles greetd authentication (via Unix socket IPC), session discovery (.desktop files), and state caching. Users provide the UI.

**Runtime**: GJS (GNOME JavaScript), not Node.js. The library has zero npm dependencies — it uses native GJS/GIO APIs exclusively.

## Development Setup

Requires Nix. Enter the dev environment:
```sh
direnv allow   # or: nix develop
```

Install dependencies and generate type definitions:
```sh
pnpm run setup   # runs pnpm install + scripts/types.sh
```

## Common Commands

### Build example greeters (Nix)
```sh
nix build .#simple
nix build .#image
nix build .#movie
```

### Documentation
```sh
pnpm run typedoc:build      # Generate API docs (TypeDoc)
pnpm run typedoc:preview    # Preview API docs locally
pnpm run wiki:dev           # Astro dev server for wiki
pnpm run wiki:build         # Build wiki for deployment
```

### Package publishing
```sh
pnpm run package:pub        # Publish to npm
```

## Architecture

### Runtime chain
```
greetd → dbus-run-session → cage (Wayland kiosk) → greeter binary (AGS)
```

cage does not support `wlr-layer-shell`, so greeters must use `Gtk.ApplicationWindow` (not AGS's `<window>` which depends on `gtk4-layer-shell`). cage automatically fullscreens the window.

### Monorepo structure (pnpm workspaces)

- **`package/`** — The npm library (`tadaima`). Source is in `package/src/`.
- **`examples/`** — Three example greeters (simple, image, movie). Each depends on `tadaima: workspace:*`.
- **`docs/wiki/`** — Astro + Starlight documentation site, deployed to Cloudflare Workers.
- **`nix/module.nix`** — NixOS service module (`services.tadaima`).
- **`@girs/`** — GObject Introspection type definitions (generated, not hand-written).

### Library modules (`package/src/libs/`)

- **`client.ts`** — `createGreeter()` factory and `createLoginHandler()` with concurrency guard.
- **`greetd.ts`** — Low-level greetd-ipc(7) protocol: JSON over Unix socket, async GIO streams.
- **`sessions.ts`** — `SessionManager`: discovers and parses `.desktop` session files.
- **`cache.ts`** — `CacheManager`: JSON state persistence with atomic writes (temp file + rename).

Public API is re-exported from `package/src/index.ts`.

## TypeScript Configuration

- JSX: `react-jsx` with `jsxImportSource: "ags/gtk4"` (Gnim JSX for GTK4)
- Module: ES2022, moduleResolution: Bundler
- Strict mode enabled
- Target: ES2020

## Formatting

Prettier with: printWidth 120, tabWidth 2, single quotes, trailing commas (es5), semicolons.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Myxogastria0808) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
