## transmission-control-app

> Context and conventions for AI agents working on this repository.

# AGENTS.md

Context and conventions for AI agents working on this repository.

## What this app is

Electron desktop app that acts as a side-docked control panel for live broadcasts. It controls **PTZ cameras over ONVIF** and the **OBS Studio scene switcher over OBS WebSocket**, and ties them together: clicking a preset moves the camera *and* swaps the OBS scene.

Target user: streamers / AV operators (church, studio, event production). UI text is in **Portuguese (pt-BR)**.

## Stack

- **Electron 38** (main + preload + renderer) scaffolded with `electron-vite`
- **React 19 + TypeScript** in the renderer
- **Tailwind v4** (`@tailwindcss/vite`) + Radix primitives + lucide-react + react-hot-toast
- **Zustand** for client state, **TanStack Query** for async/server state
- **react-hook-form + Zod** for forms and validation
- **Firebase** (Auth + Firestore) for per-user config sync
- **`onvif`** for PTZ control, **`obs-websocket-js`** for OBS

## Layout

```
src/
  main/           # Electron main process
    index.ts      # window creation, IPC wiring, menu
    Cam.ts        # ONVIF PTZ controller + CamMock for dev
    ImageCache.ts # IPC for caching base64 images under userData/images/
  preload/
    index.ts      # contextBridge: window.ptz, window.clipboard, window.imageCache
    index.d.ts    # global Window typings
  renderer/src/
    main.tsx              # router root (HashRouter)
    providers/            # AuthProvider, OBSProvider, QueryClientProvider
    hooks/
      firebase.ts         # auth + firestore init, useAuth zustand store
      config.ts           # firestore-synced user config (zustand)
      obs.ts              # OBS WebSocket singleton + zustand store
      ptz/                # PTZ contexts and hooks
      utils/confirm.tsx   # confirm dialog hook
    routes/
      home/               # main panel: PTZ cards + OBS card
      settings/           # /settings/{obs,ptz,history}
    components/           # Box, Button, Dialog, ContextMenu, form/, containers/, ...
    schemas/              # Zod schemas (CameraPTZ, OBSConfig, Overlayer)
    libs/cn.ts            # tailwind-merge helper
```

Path alias: `@renderer` → `src/renderer/src` (configured in [electron.vite.config.ts](electron.vite.config.ts) and `tsconfig.web.json`).

## Architecture notes

### Window
[src/main/index.ts](src/main/index.ts:18) creates a fixed-width 400px window pinned to the right edge of the primary display, at full work-area height. It is meant to sit alongside OBS, not on top of it. Hardware acceleration is disabled (`app.disableHardwareAcceleration()`).

### IPC surface
Four preload bridges, all in [src/preload/index.ts](src/preload/index.ts):
- `window.ptz`: `init`, `getPresets`, `goto`, `getPosition`, `onConnected`, `onLogs`
- `window.clipboard`: `writeText`
- `window.imageCache`: `save`, `get`, `clear`, `clearFolder` — files written to `app.getPath('userData')/images/<folder>/<filename>.cache`
- `window.overlays`: `put({ apiUrl, payload })` — proxies an HTTP PUT to the overlays.uno (Singular) API from the main process; avoids CORS issues that block direct fetch from the renderer

When you change the IPC surface, update **all three**: the handler in `main/`, the preload definition, and the typings in [src/preload/index.d.ts](src/preload/index.d.ts).

### PTZ
[src/main/Cam.ts](src/main/Cam.ts) wraps `onvif/promises` with a `Cam` class and a `CamStore` registry keyed by config id. There is also a `CamMock` enabled when `PTZ_CAM_DEV=true` (set in the `dev` npm script via `cross-env`). The mock fakes 30 presets and a moving position. Use it for renderer work without real cameras.

### OBS
[src/renderer/src/hooks/obs.ts](src/renderer/src/hooks/obs.ts) holds a module-level `OBSConnectionStore` singleton and a `useOBS` zustand store. It listens to OBS events (`CurrentProgramSceneChanged`, `CurrentPreviewSceneChanged`, `SceneListChanged`) and reflects them into the store. There is a `delay(2000)` after `ConnectionOpened` before fetching scenes — leave it alone unless you confirm OBS is ready earlier.

### PTZ ↔ OBS coupling
The interesting logic lives in [`useInitPTZPreset`](src/renderer/src/hooks/ptz/hooks.ts) (`gotoPresetBase`):
1. If the camera is currently on Program and the user picks a different preset, switch Program to `axSceneId` (auxiliary scene) **first** so viewers don't see the camera moving.
2. Move the camera (`window.ptz.goto`).
3. Wait `transitionTime` ms, then read the new position and (optionally) push the camera's scene back to Program.

`axSceneId` exists specifically to mask camera transitions. Don't refactor it away.

### Config persistence
Config is synced per `auth.uid` via Firestore `onSnapshot` ([src/renderer/src/hooks/config.ts](src/renderer/src/hooks/config.ts:50)). The `Config` shape is `{ obsConfig, cameraPTZConfig, presetsAlias, presetsHidden }`:
- `presetsAlias: Record<string, string>` — flat map keyed by `${cameraId}-${presetId}` → display name (renames in the PTZ panel).
- `presetsHidden: Record<string, string[]>` — per-camera array of preset IDs hidden from the panel.

Both go through `setConfig` like any other config field, so each rename/hide creates a new history version (intentional — the user accepted the trade-off).

Cached preset *positions* (used to detect "this preset is currently active") remain in `localStorage` keyed by `ptz-<configId>-presets-position-<presetId>` — that's runtime cache, per-device, not synced.

`useConfig` exposes two write paths and one helper:
- **`setConfig(partial, opts?)`** — public action, called from settings forms. Atomically writes `configs/<uid>` *and* a new entry in `configs_history/<uid>/versions/<auto>` via `writeBatch`. The optional `{ restoredFromId, restoredFromCreatedAt }` opts mark a write as coming from a restore so the new history entry is traceable.
- **`seedConfig(uid, config)`** (module-private, not on the store) — used only by the snapshot listener seed paths when a user has no config doc yet, or when the listener errors. Writes only `configs/<uid>`, **never** the history collection — those code paths don't represent user intent.
- **`setContextConfig(config)`** — pure local-state setter, called by the snapshot listener whenever Firestore pushes a fresh config.

### Config history
`configs_history/<uid>/versions/<autoId>` is a write-once subcollection. Each doc:
```ts
{
  createdAt: Timestamp                     // serverTimestamp()
  config: { obsConfig, cameraPTZConfig }   // full snapshot at this moment
  restoredFromId?: string                  // set when this entry came from a restore
  restoredFromCreatedAt?: Timestamp        // ditto, denormalized for display
}
```

Read path: [`useConfigHistory`](src/renderer/src/hooks/configHistory.ts) wraps `useInfiniteQuery` over the subcollection (page size 50, `orderBy('createdAt', 'desc')`, cursor via Firestore `startAfter(lastDocSnapshot)`).

Restore path: [`useRestoreVersion`](src/renderer/src/hooks/configHistory.ts) is a `useMutation` that calls `setConfig(version.config, { restoredFromId, restoredFromCreatedAt })` — i.e. restores by **writing forward**. The history is never rewound; a restore appends a new version tagged with the source id. UI lives in [routes/settings/history.tsx](src/renderer/src/routes/settings/history.tsx) and uses [components/ConfigDiff.tsx](src/renderer/src/components/ConfigDiff.tsx) inside a `Dialog` to preview before applying.

### Config export / import
The history page also lets the user export any version as a JSON file and import a JSON file as a new version. Helpers live in [src/renderer/src/libs/configFile.ts](src/renderer/src/libs/configFile.ts):
- `downloadConfigFile(config, filename)` — wraps the snapshot in `{ version, exportedAt, config }`, blob-downloads it.
- `parseConfigFile(text)` — JSON.parse + Zod validates against `ExportedFileSchema`, returns the inner `ConfigSnapshot`. Rejects unknown `version` numbers with a friendly message.
- `EXPORT_FORMAT_VERSION` (currently `1`) — bump this if you change the snapshot shape in a non-backwards-compatible way; the parser will refuse old files instead of silently corrupting state.

Import goes through [`useImportConfig`](src/renderer/src/hooks/configHistory.ts), which is just `setConfig(snapshot)` wrapped in `toast.promise` and a query invalidation — i.e. an imported file always becomes the latest history version (no special marker). The history page shows the same `ConfigDiff` preview before committing, so a malformed-but-validated file is still reviewable.

**Immutability** is enforced by Firestore security rules — apply this in the Firebase Console (the rules file is not in the repo):
```
match /configs_history/{uid}/versions/{versionId} {
  allow read:   if request.auth != null && request.auth.uid == uid;
  allow create: if request.auth != null && request.auth.uid == uid;
  allow update, delete: if false;
}
```
Without these rules, the immutability is only a UI convention. With them, the client can never edit or delete a version.

### Version names (mutable, separate from history)
Users can label any version with a custom name (e.g. "Pré-culto domingo"). Names are **deliberately not part of the immutable history** — they live in a separate, mutable doc:

```
configs_history_names/{uid}: { [versionId]: name, ... }
```

```
match /configs_history_names/{uid} {
  allow read, write: if request.auth != null && request.auth.uid == uid;
}
```

Trade-off: the snapshot stays tamper-proof (the audit-trail use case), but the human label can be renamed/deleted (the UX use case). Same idea as git: commit hash is immutable, tag pointing to it isn't. Hooks in [hooks/configHistory.ts](src/renderer/src/hooks/configHistory.ts):
- `useConfigVersionNames()` — onSnapshot listener, returns `Record<versionId, name>`.
- `useSetVersionName()` — mutation with `{ versionId, name }` — `name=null` deletes the entry via `deleteField()`.

UI: when a name is set, it replaces the timestamp on the row's primary line; the timestamp moves to a small subtitle below. The history page renders two sections — first a "Versões nomeadas" section showing only the entries that have a name (sourced from the `names` doc), then "Todas as versões" showing the full chronological list. A named version appears in BOTH sections; an unnamed version appears only in the chronological list. Each row in either section gets the same actions (rename, download, restore) and the "Atual" tag is rendered on whichever rows correspond to the latest version.

Caveat with pagination: the named-versions section is built from the in-memory `versions` array (i.e. the loaded pages). If a user has named a version that lives outside the loaded pages, it won't appear in the named-versions section until they hit "Carregar mais" enough times to load that page. For the typical scale (small number of named entries, recent ones at the top) this is fine; if it becomes a problem, fetch named versions individually by id from `configs_history_names`.

### Overlayer controls (overlays.uno integration)
The app drives external [overlays.uno](https://overlays.uno/) overlays as a control panel. No custom web overlay page is hosted — the visual layer stays in overlays.uno; this app only fires API calls.

API reference: official Singular/UNO command list (ShowOverlay, HideOverlay, GetOverlays, GetOverlayContent, TakeOverlaySlotName, etc.). The doc is reachable via the `?` → "API Description" menu inside any overlay on overlays.uno; if you need it, paste the relevant excerpt into the conversation rather than committing it to the repo.

Config field: `overlayerControls: Record<string, OverlayerControl>` where each is just `{ id, name, url }`. The list of items inside an overlay is **fetched at runtime** via the `GetOverlays` command — items are NOT persisted in Firestore (the source of truth is the user's overlays.uno setup).

Helpers in [src/renderer/src/schemas/OverlayerControl.ts](src/renderer/src/schemas/OverlayerControl.ts): `parseOverlayUnoControlUrl`, `apiUrlFromAppId`, `apiUrlFromControlUrl` derive the API endpoint `https://app.overlays.uno/apiv2/controlapps/{appId}/api`.

API call goes through `window.overlays.put({ apiUrl, payload })` — handled in [src/main/Overlays.ts](src/main/Overlays.ts) which does `fetch(apiUrl, { method: 'PUT', body: JSON.stringify(payload) })` from the main process. The endpoint is **not GET-based** — it's RPC over PUT, with the operation in the `command` field. Calling from the renderer would hit CORS.

Payloads (Uno App API):
- Show: `{ command: 'ShowOverlay', id: '<overlayId>' }`
- Hide: `{ command: 'HideOverlay', id: '<overlayId>' }`
- Enumerate: `{ command: 'GetOverlays' }` → returns the list of overlay items (the response shape isn't pinned in the doc, so the renderer parser is defensive — see `parseOverlaysResponse` in [hooks/overlayerControls.ts](src/renderer/src/hooks/overlayerControls.ts))

Hooks in [src/renderer/src/hooks/overlayerControls.ts](src/renderer/src/hooks/overlayerControls.ts):
- `useOverlayerControls()` — selector pro `config.overlayerControls`
- `useOverlayItems(control)` — TanStack `useQuery` que faz `GetOverlays` e parseia o response. `staleTime: 30s`, retry 1.
- `usePlayingItems()` — zustand local com `Record<controlId, itemId | null>`. Não vai pro Firestore (runtime, não merece histórico)
- `usePlayItem()` — mutation: stops the previously-playing item in the same control if any, then plays the new one. **One item per control may be playing at a time.**
- `useStopItem()` — mutation: stops the currently-playing item in the control

UI:
- Settings: [routes/settings/overlayers.tsx](src/renderer/src/routes/settings/overlayers.tsx) — list + add + edit + delete controls (just name + URL; nothing about items here since they live in overlays.uno)
- Home: [routes/home/overlayers.tsx](src/renderer/src/routes/home/overlayers.tsx) — accordion card per control. Items are auto-listed via `useOverlayItems`; loading/error/empty states; no item creation UI (the user manages items inside overlays.uno itself). When collapsed AND an item is playing, the header shows the item name plus a small Stop button.

Two state caveats:
- **What's playing** is local to the app session — there's no WebSocket back from overlays.uno. Restarting the app forgets which item was on-air. The Singular API has a `GetOverlayVisibility` command that could be polled if real sync becomes important.
- **The `GetOverlays` response shape isn't fully pinned** in the doc — the parser tries `Array<{id,name}>`, then `{overlays: ...}`, `{items: ...}`, `{data: ...}`, then throws with the raw JSON in the error message. If a user's overlay returns something different, surface that error and update the parser.

### Manual snapshot (Criar versão agora)
[`useCreateVersionNow`](src/renderer/src/hooks/configHistory.ts) is a no-op `setConfig({})` wrapped in `toast.promise` — writes the current config back to `configs/<uid>` and creates a new history entry that's identical to the previous one. Used as a marker / checkpoint when the user wants to bookmark the current state without making an actual config change. The button sits at the top of the history page.

### Schemas
All persisted shapes are Zod-validated:
- [CameraPTZ.ts](src/renderer/src/schemas/CameraPTZ.ts)
- [OBSConfig.ts](src/renderer/src/schemas/OBSConfig.ts)

When you add a config field, update the Zod schema, the form in `routes/settings/`, and any default-construction site (e.g. `addCamera` in [routes/settings/ptz.tsx](src/renderer/src/routes/settings/ptz.tsx)).

## Conventions

- **All UI strings are pt-BR.** Don't translate them to English unless asked.
- **Toasts via `react-hot-toast`** with `toast.promise` for any async user action — keep the `loading / success / error` triplet in Portuguese.
- **Forms** use `react-hook-form` + `zodResolver`. Use `FormControl` + `TextField` from `components/form/`.
- **Styling:** Tailwind utility classes, plus `cn()` from [libs/cn.ts](src/renderer/src/libs/cn.ts) for conditionals. Variants on buttons use `class-variance-authority`. Don't add a CSS-in-JS library.
- **Icons:** lucide-react. The `OBSIcon` and `GoogleIcon` are custom in `components/icons/`.
- **State:** prefer existing zustand stores (`useAuth`, `useConfig`, `useOBS`) and the PTZ contexts over adding new top-level stores.
- **Don't add comments** that just restate what the code does. The codebase is intentionally sparse on comments.

## Dev workflow

```bash
npm run dev          # electron-vite dev with PTZ_CAM_DEV=true (mocked cameras)
npm run typecheck    # tsc -p tsconfig.node.json && tsc -p tsconfig.web.json
npm run lint         # eslint --cache .
npm run format       # prettier --write .
npm run build        # typecheck + electron-vite build
npm run build:mac    # → dmg
npm run build:win    # → portable exe
npm run build:linux  # → AppImage / snap / deb
```

There is **no test suite**. Don't claim "tests pass" — there are no tests to run. If you change behavior, verify by running `npm run dev` and exercising the feature.

Firebase config is loaded from `.env.development.local` (Vite `VITE_FIREBASE_*` vars). Don't commit secrets; the file is gitignored.

## Things to be careful about

- **Credentials in Firestore are plaintext.** Camera and OBS passwords ride along in the user's config doc. Don't widen access; if you touch this, prefer adding client-side encryption over loosening rules.
- **`fs`, `path`, `url` in `dependencies`** are placeholder/squat npm packages, not the Node built-ins. They can be removed but verify nothing imports them as modules first.
- **`sandbox: false`** is set on the BrowserWindow ([main/index.ts:33](src/main/index.ts#L33)). Required because the preload uses `ipcRenderer.invoke` patterns that the sandboxed preload can't do here without rework. Don't flip it without testing the IPC bridges.
- **Window position assumes a single primary display.** Multi-monitor handling is not implemented.
- **HashRouter, not BrowserRouter** — required because the renderer is loaded from `file://` in production.
- **`onvif` is CommonJS-ish** and pulled in via `onvif/promises`. It's externalized by `externalizeDepsPlugin()` in [electron.vite.config.ts](electron.vite.config.ts) — keep it that way.
- **`globalShortcut.register('CommandOrControl+Shift+I', ...)`** is registered globally; it will steal that shortcut from other apps while this one is running. Be aware before adding more global shortcuts.

## When making changes

- **Touching IPC?** Update main handler + preload bridge + `index.d.ts` in the same change.
- **Touching config shape?** Update the Zod schema + the settings form + `initialConfig` defaults + any `addCamera`-like default constructors + [components/ConfigDiff.tsx](src/renderer/src/components/ConfigDiff.tsx) so the new field shows up in the restore preview (either add to the relevant `*_FIELD_LABELS` map for flat scalar fields, or add a dedicated section like the existing `presetsAlias` / `presetsHidden` blocks for nested data). Existing user docs in Firestore won't have new fields — make them optional or write a migration.
- **Adding a new write path for config?** Use `useConfig.setConfig` (or wrap it). Do **not** call `setDoc(doc(db, 'configs', uid), ...)` directly — that bypasses the history write. The only legitimate exception is `seedConfig` for the no-doc-yet case, which already exists.
- **Touching OBS event handling?** Test reconnect (kill OBS, restart it) — `OBSConnectionStore` is a singleton and reconnect logic goes through `init(config, setState, true)`.
- **Touching PTZ goto flow?** Re-read [`gotoPresetBase`](src/renderer/src/hooks/ptz/hooks.ts) end-to-end before editing. The interplay between `axSceneId`, `transitionTime`, position polling and OBS scene swap is subtle.
- **Adding a route?** Register it in [src/renderer/src/main.tsx](src/renderer/src/main.tsx) and add a nav button in [routes/settings/layout.tsx](src/renderer/src/routes/settings/layout.tsx) if it belongs under Settings.

## Out of scope (don't add unless asked)

- Tests / CI
- i18n (UI is pt-BR by design)
- Auto-update flow (electron-builder `publish` is configured but not wired into the app)
- Multi-window support
- The commented-out `OverlayersCard` / `Overlayer` schema — work in progress, leave it.

---
> Source: [gabrielrosas/transmission-control-app](https://github.com/gabrielrosas/transmission-control-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-13 -->
