## void-saber2

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

Goal: Quickly finish end-to-end prototype.

See docs/BUILD-GUIDE.md for build steps and decisions. See docs/Board.md for current tasks.

# Principles

- Follow TDA, PLOP, and Encapsulation
- Strict: cleanup, no memory leaks
- Strict: no double computations — if something can be pre-calculated, it must not be repeated
- Strict: never swallow any error — either fail hard or log, based on whether it makes sense to continue or it completely breaks app
- Strict: no spooky action at a distance

# Scripts

- `pnpm dev` — Vite dev server (HTTPS via basicSsl, required for WebXR on Quest 2)
- `pnpm build` — vite build
- `pnpm preview` — preview production build
- `pnpm typecheck` — tsc --noEmit

No test runner or linter is configured.

# Architecture

See `docs/architecture.md` for app model, routing, directory structure, and module map.

## Module Pattern

All modules use `createXxx(deps...) → handle` with systems, methods, and dispose. No classes. Pages implement `{ systems: System[], dispose: Teardown }`.

## Command Queue

Cross-page communication goes through `CommandQueue` — pages enqueue domain events, router drains once per frame. No callbacks between pages and router. Commands: `songSelected` (seed), `arenaSessionCompleted`.

## XR Session

Persistent across page transitions. `createXRSession` resolves only after user enters VR — non-null guarantee. Arena uses controllers for sabers, lobby will use them for laser pointers.

## Conventions

- **Babylon.js imports**: Use deep imports (`@babylonjs/core/Meshes/meshBuilder`) not barrel imports. Side-effect imports (e.g. `import '@babylonjs/loaders/glTF'`) go in main.ts only.
- **Babylon.js scene**: Always pass `scene` explicitly to constructors (`new StandardMaterial(name, scene)`, `MeshBuilder.Create*(name, opts, scene)`). Never rely on Babylon's implicit "last created scene" fallback.
- **No cross-page imports**: `lobby-page/`, `arena-page/`, and `music/` must not import from each other. Shared types go in `src/types.ts` or `music/music-types.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jigargosar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
