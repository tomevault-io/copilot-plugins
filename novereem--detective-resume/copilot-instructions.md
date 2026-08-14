## detective-resume

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal résumé website whose centerpiece is a fully in-browser 3D "detective office" built with **React Three Fiber / three.js**. The room is an interactive puzzle game: exploring and solving puzzles reveals the author's CV. There are effectively **two products in one repo**: the résumé site (`/[locale]/resume`, localized EN/NL) and the detective game (`/detective-room`, the bulk of the code).

The room is **100% procedural geometry** — there are no imported `.glb`/`.gltf` models. Every object is composed from primitives via a small custom "room engine". `README.md` is the developer-onboarding doc (how the engine works, tutorials for adding objects/puzzles); read it before doing engine work.

## Commands

```bash
npm run dev            # Next.js dev server (Turbopack) → http://localhost:3000
npm run build          # production build (NOTE: ESLint is skipped during build)
npm start              # run the production build

npm test               # Jest unit tests (jsdom). tests/Perf/ is excluded here.
npm test -- <pattern>  # single file, e.g. npm test -- state.data
npm test -- -t "name"  # single test by name
npx jest src/components/Game/__tests__/state.data.test.ts   # exact path

npm run lint           # `eslint .` (flat config). Clean: 0 errors; ~140 `no-explicit-any` warnings are expected signal, not failures.
npm run test:perf      # Playwright FPS/perf spec (needs the app running)
npm run ci:playwright  # start prod server + run perf spec
npm run lhci           # Lighthouse CI
```

CI (`.github/workflows/ci-cd.yml`, on push/PR to `master`): `npm ci` → `npm test -- --runInBand` → `npm run build` → Playwright perf → Lighthouse. **There is no lint step in CI**, and lint is not enforced anywhere.

## Architecture — the big picture

**Routing / entry.** `src/app/detective-room/page.tsx` → `DetectiveRoomClient.tsx` (tutorial overlay, boot/preload gate, all HUD providers) → dynamically imports `DetectiveRoom.tsx` with **`ssr: false`** (the whole game is client-only; anything touching `window`/WebGL must stay out of SSR). `DetectiveRoom.tsx` owns the `<Canvas>` and wires the scene, camera/input, post-processing, and the inspect overlay. The résumé site lives under `src/app/[locale]/` and is a separate, simpler surface.

**Model-building stack (read these together).** `@/*` maps to `src/*` (tsconfig).
- `Models/Generic/Outlined/Outlined.tsx` — one mesh + a scaled back-face outline mesh + hover/click/inspect + optional texture and magnifier-reveal material.
- `Models/Generic/Outlined/FramedPlane.tsx` — framed poster/screen primitive (used for puzzle "clue" images), with a magnifier-only visibility mode.
- `Models/Generic/ModelGroup.tsx` — the central abstraction: takes an array of `PartSpec`, builds many `Outlined` parts, resolves per-part vs group vs `materialsById` overrides, manages auto/manual interaction hitboxes, and emits the inspect payload. Every concrete model (Bookshelf, Desk, Clock, …) is a thin wrapper that computes `parts` and hands them to `ModelGroup`.

**Scene composition.** `DetectiveRoom.tsx` renders **cluster** components (`DetectiveRoom/Clusters/*`: Walls, BigFurniture, BindersAndBooks, Lights, AnimatedDecoration, FlatDecoration, Decoration) plus **functional-object** groups (`FunctionalObjects/*`: MovingObjects, PuzzleObjects, UsableItemObjects, Effects). Add new objects to the appropriate cluster, never directly to the scene root.

**Anchors are the single source of truth for placement.** `Game/anchors.ts` holds position/rotation (and optional `eye` for camera focus) for every object and camera target. Clusters read `ANCHOR.*`; right-click focus and post-solve camera zooms are computed from anchor `eye`/`position`.

**Game state is a hand-rolled external store — NOT Redux/Zustand/Context.**
- `Game/state.logic.ts` — a `GameState` class with an immutable `GameSnapshot` (files, drawer_files, poofs, drawers, puzzlesConfig, puzzleStatus, cardboardBoxes). Every mutation creates a new snapshot and calls `emit()`.
- `Game/state.ts` — a **module-level singleton** `gameState`, plus `useGameState()` (subscribe + force re-render) and `useGameActions()` (returns the singleton). The singleton persists across mounts/HMR and has no reset method; tests import the class directly.
- `Game/state.data.ts` — `initialSnapshot` + all puzzle definitions. **This is where the résumé content lives** (see below). Branded IDs via `asPuzzleId`/`asFileId`; `state.data.test.ts` enforces cross-references between `puzzlesConfig`/`puzzleStatus`/anchors stay in sync.

**Puzzle / inspect flow.** Clicking an object calls `openInspect(state)`; `Puzzles/ObjectInspectOverlay.tsx` renders it **in its own separate `<Canvas>`** (on-demand `invalidate()` rendering, unlike the always-rendering main scene). Solving reports back up through `DetectiveRoom`'s `onSolved`/`onAction` callbacks, which drive game-state actions (`pinPuzzle`, `handleSecretOpen`, `requestOpenCardboardBox`) and a choreographed set of timers for view-delay → camera zoom. Solved puzzles get pinned to the corkboard (`Puzzles/PuzzleNode.tsx`, `RedStringsEffect.tsx`).

**Textures & materials.** `Textures/TextureManager.ts` is a standalone (non-React) module: a concurrency-limited (`MAX=2`) load queue with promise dedup, ref-counting, global loading state, and `preloadTextures`. `useManagedTexture` is the React wrapper (releases on unmount/URL change). `DETECTIVE_ROOM_TEXTURES` lists every texture for preload — keep it updated when adding assets. Big-object materials are centralized in `Materials/detectiveRoomMats.ts` and passed to models as `materialsById`.

**Settings, quality, camera, magnifier.**
- `Settings/SettingsProvider.tsx` — all player prefs, persisted to `localStorage` (writes gated behind a `loadedRef` so nothing persists before the initial read). `useSettings()` throws if used outside the provider. `QualityContext` exposes just `modelQuality` so heavy models can vary LOD.
- `PlayerControls/*` — `CameraControls` (free-look slerp + `PlayerMover` + pose bridge), `InputControls` (wheel zoom + right-click focus), `GameplayControls` (magnifier pickup), `DevControls` (fly + object-move gizmo). Damping is frame-rate-independent throughout.
- Magnifier: `CameraEffects/Magnifier/MagnifierStateContext.tsx` shares a per-frame lens mask via a **ref** (avoids per-frame React renders); `MagnifierRevealMaterial.tsx` patches a shader to discard fragments outside the lens, revealing magnifier-only secrets.

**Dev modes** are URL-gated: `?fly` (WASD/Space/Shift fly camera) and `?move-objects` (drag gizmo that logs updated anchor coords to the console for copy-paste into `anchors.ts`). They can be combined.

## Where the résumé content lives (important for content edits)

Updating the CV is **not** a single-file text edit. Puzzle prompts, accepted answers, and the author's facts are in `Game/state.data.ts` (`puzzlesConfig[...].view.inspect`), **and** each clue is a hand-authored baked JPG in `public/textures/` (e.g. `puzzle_hboictpropedeuse.jpg`, `puzzle_semesterplanning.jpg`, `puzzle_blurredcompanies.jpg`). Changing a fact usually means editing the answer/prompt **and** re-authoring the matching image. The game HUD/puzzle strings are hardcoded English (no `next-intl`); only the `/[locale]/resume` page is bilingual (real EN/NL translations in `messages/`).

## Conventions

- **Extend, don't inline:** new object → model file under `Models/` (a `ModelGroup` wrapper) + anchor in `anchors.ts` + render in a cluster. New puzzle → entry in `PZ` + `files`/`drawer_files` + `puzzlesConfig` + `puzzleStatus` (kept in sync by `state.data.test.ts`).
- Use `materialsById` for shared materials rather than duplicating material config; use `useQuality()` to gate detail on heavy models.
- Models are wrapped in `memo()` and memoize their `parts` arrays; keep that pattern.
- Tests mock R3F and drive `useFrame` manually — logic/behavior is tested, real three.js math is not.

## Known gotchas

- **ESLint uses a single flat config** (`eslint.config.mjs`); the old `.eslintrc.cjs` was removed. Lint runs via `eslint .` and is clean (0 errors); `no-explicit-any` is intentionally `warn`, and `react/no-unknown-property` is disabled for the r3f directories. `next build` still skips lint (`next.config.ts` `ignoreDuringBuilds: true`), and there is no lint step in CI.
- **`next build` can fail with `Cannot find module '.../[turbopack]_runtime.js'`** when the `.next` dir holds stale artifacts from `npm run dev` (the dev script uses `--turbopack`). Fix: `rm -rf .next` and rebuild. CI is unaffected (clean checkout).
- **The main scene runs an always-on render loop** (`frameloop="demand"` is commented out in `DetectiveRoom.tsx`). `TriangleLogger` and dev controls now only mount when `process.env.NODE_ENV === 'development'` / when their `?fly`/`?move-objects` URL params are set.
- **`disposeAllManagedTextures()` is never called by app code** (only in tests) — textures persist in GPU memory after leaving `/detective-room`.
- `Puzzles/ObjectInspectOverlay.tsx` is the largest, least type-safe (`as any`-heavy) file and has **no tests** despite being central to the puzzle flow — be careful editing it.
- Cross-component signalling uses `window` custom events (e.g. `tt:moveBackToDesk`) cast with `as any`.
- The startup-tutorial GIFs (`public/tutorial/*.gif`) are not committed yet; `TutorialMedia` degrades to a text placeholder when an asset is missing, so add the GIFs to light it up.

---
> Source: [Novereem/detective-resume](https://github.com/Novereem/detective-resume) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
