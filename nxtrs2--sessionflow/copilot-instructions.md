## sessionflow

> Guidance for Claude Code (or any AI agent) working in this repository.

# CLAUDE.md

Guidance for Claude Code (or any AI agent) working in this repository.

## Project Overview

**Session Flow** is a browser-based music player/practice tool built for session
musicians. A musician uploads a "master" mix (MP3) plus optional isolated
instrument stems, and the app renders a scrolling vertical timeline synced to
the audio via [Tone.js](https://tonejs.github.io/). While the track plays, the
timeline shows bar/beat numbers, user-placed **markers** (named positions) and
per-instrument **events** ("prompts" — timed text cues like "Chorus!" or "Come
in here") that count down before they appear. It supports tempo/time-signature
config, a metronome click, count-in/count-out, loop practice between two
markers, and per-instrument volume/mute/solo mixing.

- Live demo (legacy web/Supabase build): https://sessionflow.nxt.rs
- This app is being migrated from a Supabase-backed web app to a **Tauri**
  desktop app with **local filesystem storage** — there is no backend/server
  and no user accounts. All project data (metadata + audio files) lives under
  the OS app-data directory, managed via `src/local/projectStore.ts`. See
  "Local storage model" below. Supabase has been fully removed from the app
  (no `@supabase/supabase-js` dependency, no network calls for project data).

## Tech Stack

- **React 18** + **TypeScript**, built with **Vite 6**
- **Tauri 2** (`src-tauri/`, Rust) — desktop shell; `@tauri-apps/plugin-fs` and
  `@tauri-apps/plugin-dialog` for local filesystem access from the frontend
- **Tone.js** — Web Audio playback, transport/clock, scheduling, synth click
- **lucide-react** — icon set
- Plain CSS (`App.css`, `index.css`, component-level `.css` files) — no CSS
  framework, no CSS-in-JS
- **ESLint 9** flat config (`eslint.config.js`) with `typescript-eslint`,
  `eslint-plugin-react-hooks`, `eslint-plugin-react-refresh`

No test framework is configured in this repo (no test runner, no test files).

## Commands

```bash
npm run dev         # start Vite dev server only (browser preview; local fs calls will fail — no Tauri IPC bridge)
npm run tauri dev   # start the actual desktop app (Vite + Rust, hot-reloaded)
npm run build       # tsc -b (project references) + vite build -> dist/
npm run tauri build # produce a distributable desktop app bundle
npm run lint        # eslint .
npm run preview     # preview the production web build locally (same caveat as `npm run dev`)
```

There is no `npm start` script despite what the README says, and no test
command — do not assume either exists.

Building/running the Tauri shell requires a working Rust toolchain (`cargo`,
`rustc`) in addition to Node — install via [rustup](https://rustup.rs) if
missing. First build compiles ~350 Rust crates and can take a couple of
minutes; subsequent builds are incremental.

## Environment Variables

Vite is configured (see `vite.config.ts`) to expose the entire loaded `.env`
as `process.env.*` (not the usual `import.meta.env.VITE_*` pattern). There are
currently no required environment variables for the app to run — the old
`REACT_SUPABASE_URL` / `REACT_SUPABASE_ANON_KEY` vars are vestigial leftovers
from the pre-migration Supabase backend and are no longer read anywhere in
`src/`.

## Architecture

### Provider hierarchy (`src/main.tsx`)

```
initProjectStore()      awaited before the app is rendered (caches appDataDir)
<SessionProvider>        local auth stub — always "logged in", no backend
  <AppProviders>
    <ProjectsProvider>   list of local projects, CRUD via src/local/projectStore.ts
      <CurrentProjectProvider>  in-memory state of the currently loaded project
        <App />
```

- `main.tsx` calls and awaits `initProjectStore()` (`src/local/projectStore.ts`)
  before calling `createRoot(...).render(...)`, so every component can assume
  the local project store is ready and `resolveAssetUrl` can resolve
  synchronously.
- `AppProviders` (`src/contexts/AppProviders.tsx`) no longer threads any
  session/user info into the providers below it — `ProjectsProvider` and
  `CurrentProjectProvider` are self-sufficient now that there's no per-user
  backend.
- Each context has a matching `use*` hook in `src/hooks/` that throws if used
  outside its provider (`useSession`, `useProjects`, `useCurrentProject`).

### State ownership

- **`SessionProvider`** — a local no-op stub (`isLoggedIn: true` always,
  `session: null`, `signIn`/`signUp`/`signOut` are no-ops resolving
  immediately). The interface (`src/contexts/SessionContext.tsx`) is kept
  intact — including its own local `AuthUser`/`AuthSession`/`AuthErrorLike`
  types (no `@supabase/supabase-js` dependency anymore) — so a future
  cloud-sync backend can implement it for real without touching call sites
  like `Login`/`Register`/`Profile`.
- **`ProjectsProvider`** — `projects[]` (from `listProjects()`),
  `currentProject`, demo-loaded flag, and `createProject` / `updateProject` /
  `deleteProject`, all delegating to `src/local/projectStore.ts` for the
  actual filesystem reads/writes.
- **`CurrentProjectProvider`** — the *loaded* project's mutable editing state:
  `instruments[]`, `selectedInstrument`, `projectNeedSave` (dirty flag),
  `updateProjectSongData` (persists the full `SongData` blob back to
  `project.json` on disk via `projectStore.updateProjectSongData`).
- **`App.tsx`** (~1000 lines) — the rest of the runtime state lives here as
  plain `useState`: playback position, tempo, time signature, loop
  start/end, markers, timeline events ("song time lines"), beat data,
  count-in/out, master volume/pan/mute/solo, dialog visibility flags, etc.
  This is the main integration point and is the most important file to read
  when making feature changes. It is not split into smaller hooks/reducers —
  be careful about effect ordering when adding new state derived from
  `songData`, `duration`, `tempo`, or `timeSignature`.

## Local storage model (`src/local/projectStore.ts`)

Projects live entirely on disk under the Tauri app-data directory (macOS:
`~/Library/Application Support/app.sessionflow.desktop/`), resolved once via
`@tauri-apps/api/path`'s `appDataDir()` and cached:

```
<appDataDir>/projects/<projectId>/
  project.json           the full Project object (metadata + data: SongData), pretty-printed
  master.<ext>            master mix audio
  cover.<ext>              cover art (optional)
  instruments/<id>.<ext>   one file per instrument stem that has an uploaded track
```

- `projectId` is a `number`, generated with `Date.now()` at creation time
  (kept as a plain `number` — not a UUID — to minimize the diff from the old
  Supabase-integer-id-based code across the app).
- All reads/writes go through `@tauri-apps/plugin-fs` with
  `BaseDirectory.AppData`, scoped via `src-tauri/capabilities/default.json`
  (`fs:allow-appdata-*-recursive`, `fs:scope-appdata-recursive`) and
  `src-tauri/tauri.conf.json`'s `app.security.assetProtocol` (`enable: true`,
  `scope: ["$APPDATA/**"]`) so `convertFileSrc()` can build playable
  `asset://` URLs pointing into that folder.
- **Picking files never needs a broad/arbitrary filesystem read scope**: file
  imports (master track, cover art, instrument stems) all go through plain
  HTML `<input type="file">` elements, and the resulting `File` object's
  `.arrayBuffer()` is written directly into the project folder. This sidesteps
  Tauri's fs scope entirely for arbitrary source paths — only the app's own
  `$APPDATA/**` needs to be writable/readable.
- `resolveAssetUrl(projectId, filename)` (sync, once `initProjectStore()` has
  resolved) is the one place that turns a stored relative filename into a
  webview-loadable URL — used for master/cover art `<img>`/`<audio>` `src`s
  and for building each instrument's `Tone.Players` track URL
  (`src/helpers/FileFunctions.ts`'s `loadTracksFromInstruments`).
- **Demo projects are the one exception**: they're static assets bundled in
  `public/demo/` and `public/demo2/` (downloaded once from the old Supabase
  Storage bucket during the migration) and referenced by plain `/demo/...`
  paths — no `resolveAssetUrl` involved, no on-disk project folder, and they
  bypass `ProjectsProvider` entirely (`isDemoLoaded` flag in `App.tsx`).
- There is currently no library index/cache file — `listProjects()` just
  scans `<appDataDir>/projects/*/project.json` on every call. Fine at small
  scale; revisit if the project count grows large.

### Data flow for loading a project

1. User picks a project or demo in `ProjectsList` → `handleLoadSongJSON(songData)`
   or `handleLoadSongJSONFile(url)` (for demos, which fetch a static JSON file
   from `public/data/*.json` via `loadSongFile` in `src/utils/index.ts`).
2. `App`'s `useEffect` on `songData` unpacks the `SongData` blob into all the
   individual state slots (tempo, notes, markers, timeline, instruments,
   master volume/pan/mute/solo, etc.) and calls `handleLoadMasterTrack`.
3. `handleLoadMasterTrack` → `loadMasterTrackFromJson` (`src/helpers/FileFunctions.ts`)
   creates a `Tone.Players` with a `"master"` track, syncs it to
   `Tone.Transport`, and sets `duration` once buffers are loaded.
4. A separate effect reacts to `instruments` and calls
   `loadTracksFromInstruments`, which adds each instrument's stem as another
   named player (`id.toString()`) on the same `Tone.Players` instance so they
   all stay in sync via the shared Transport.
5. `generateBeatData` (`src/utils/index.ts`) walks `duration` beat-by-beat
   (accounting for `structure` tempo/time-signature changes and any
   `songTimeLines` events) to build the array the timeline renders and the
   metronome/message scheduler uses.
6. Editing (adding/removing markers or events, changing instruments, tempo,
   etc.) sets `projectNeedSave = true`; the user explicitly saves via
   `handleUpdateProjectSongData`, which re-serializes everything into a
   `SongData` object and calls `updateProjectSongData`, which overwrites
   `project.json` on disk via `src/local/projectStore.ts`.

### Audio/Transport model

- A single shared `Tone.Players` instance (`playersRef`) holds the master
  track plus one player per instrument, all `sync()`'d to `Tone.getTransport()`
  so play/pause/seek/loop apply uniformly.
- `clickSynthRef` is a separate `Tone.Synth` used purely for the metronome
  click (`C5` on bar starts, `C6` on other beats), scheduled with
  `transport.scheduleRepeat(..., "4n")`. **Note:** the click currently
  assumes the beat subdivision is always a quarter note ("4n"), regardless of
  the configured time signature denominator.
- Timeline position/scroll is computed from `Tone.Transport.seconds` on every
  `requestAnimationFrame`, converted to a fractional beat via `tempo`, and
  used to derive `currentBar`/`beatCount` and the vertical timeline transform.

### Domain types (`src/types/index.ts`)

- `SongData` — the full serialized project state (`project` settings +
  `notes` + `structure` + `instruments` + `timeline` (events) + `markers`).
  This is the shape stored in each project's `project.json` `data` field on
  disk, and in the demo JSON files under `public/data/`.
- `BeatData` — one entry per beat of the track (time offset, bar/beat number,
  whether it's a bar start, any attached message/countOut/instrument), used
  to drive the timeline and metronome.
- `Structure` — a tempo/time-signature override for a beat range (for songs
  with tempo changes) — supported by `generateBeatData` but there is no UI
  yet to create/edit these.
- `MarkerData` — `{ beat, label }`, a named jump point (used for Go-to and
  loop start/end selects).
- `EventData` — `{ beat, instrumentId, message, countOut }`, a timed text
  prompt shown to a specific instrument (or all, if `instrumentId` is null)
  that appears `countOut` beats early and counts down (see `CountOut`).
- `Instrument` — `{ id, name, filename, url?, color, bgcolor, volume, pan,
  mute, solo }`. `filename` is a path relative to the project's local folder
  (e.g. `"instruments/3.mp3"`) for real projects; `url` is only set (and only
  meaningful) for bundled demo instruments, as a `public/`-relative prefix
  (e.g. `"/demo/"`) — see "Local storage model" above.
- `Project` — the on-disk shape (`id`, `title`, `data: SongData`, `filename`,
  `coverart`, `notes`, `created_at`, `modified_at`). No `user_id`/`url` fields
  — those were Supabase-specific and were removed during the local-storage
  migration.

## Directory Structure

```
src/
  App.tsx                    Main app shell + most runtime state (see above)
  App.css, index.css         Global styles
  components/                UI components (see below)
  contexts/                  React Context definitions + Providers
  hooks/                     use* hooks: context accessors + reusable dialog
                              hooks (useConfirm/useAlert/useInputPrompt render
                              a promise-based modal via a returned component)
  helpers/                   Non-React helper functions grouped by concern
                              (FileFunctions: track loading via Tone.js,
                              TransportFunctions: play/pause/seek/loop)
  local/projectStore.ts      Local filesystem-backed project CRUD (see above)
  utils/index.ts             generateBeatData, approximatelyEqual, loadSongFile
  types/index.ts             All shared TypeScript types
public/
  data/song.json, song2.json Demo project SongData fixtures
  demo/, demo2/               Bundled demo project audio + cover art assets
  clicks/, songs/            Static audio assets
src-tauri/
  tauri.conf.json            App window config, identifier, asset protocol scope
  capabilities/default.json  Tauri permission grants (fs scoped to $APPDATA, dialog)
  src/lib.rs, src/main.rs    Minimal Rust entrypoint — just registers the fs/dialog/log plugins
```

Key components (`src/components/`):

| Component | Responsibility |
|---|---|
| `Header`, `Footer`, `Tabs` | Chrome / navigation |
| `TransportControls` | Play/pause/skip/restart buttons, wraps `TransportFunctions` |
| `TimeLineRenderer.tsx` (`renderTick2`) | Renders a single timeline tick (bar/beat/skip), markers, event messages, and the hover edit menu |
| `CountIn` / `CountOut` | Pre-roll counter and mid-song event countdown overlays |
| `ProjectsList` | Demo + user project browser, load/save/delete entry points |
| `NewProject` / `EditProject` | Create/edit project dialogs (title, tempo, time sig, master file, cover art, notes) |
| `Instruments` | Per-instrument mixer (volume slider, mute/solo) + selection for filtering the timeline |
| `AddInstrumentDialog` | Add a new instrument (optionally with an uploaded stem) |
| `EventDialog` | Create/edit a timed prompt (`EventData`) for the selected instrument at a tick |
| `Login` / `Register` / `Profile` | Auth dialogs — currently unreachable in normal use since `isLoggedIn` is hardcoded `true` by the local `SessionProvider` stub; kept for a future cloud-sync feature |
| `Notes` | Read-only notes viewer dialog |
| `VerticalSlider` | Custom vertical volume slider used in the mixer |
| `Loader` | Full-screen loading overlay with message |

## Known Quirks / Gotchas

- Large amounts of commented-out code are left in place throughout
  `ProjectsProvider.tsx`, `NewProject.tsx`, `EditProject.tsx`,
  `TimeLineRenderer.tsx`, `CountIn.tsx`, etc. (earlier implementations kept
  for reference). Don't assume it reflects current behavior; prefer deleting
  dead code you touch rather than adding more of it.
- The time-signature denominator `<select>` in `App.tsx`'s Settings tab has a
  duplicate `<option value="4">4</option>` and is missing common values —
  likely worth fixing if touching that UI.
- The metronome/click scheduler always uses `"4n"` regardless of the actual
  denominator in `timeSignature`.
- `README.md` describes an `npm start` script and a `CONTRIBUTING.md`/`LICENSE`
  that don't exist in this repo — don't rely on it for setup instructions,
  use this file and `package.json` instead.
- No automated tests exist; verify changes manually via `npm run tauri dev`
  (not `npm run dev` — that's a plain browser preview with no Tauri IPC
  bridge, so any `@tauri-apps/plugin-fs` call will fail/no-op there).
- `stats.html` at the repo root is a generated Rollup bundle-visualizer
  artifact, not source — ignore it.
- Instrument IDs come from `getNextInstrumentIndex()` in
  `CurrentProjectProvider.tsx`, which is just `instruments.length + 1` — not
  guaranteed unique after deletions. Since instrument audio files are now
  named `instruments/<id>.<ext>` on disk, an ID collision could silently
  overwrite another instrument's stem file. Pre-existing fragility, not
  introduced by the local-storage migration, but worth fixing if you're
  touching instrument add/delete logic.
- `.env` still has unused `REACT_SUPABASE_URL`/`REACT_SUPABASE_ANON_KEY`
  entries left over from before the migration; harmless (nothing reads them)
  but fine to delete.

---
> Source: [nxtrs2/sessionflow](https://github.com/nxtrs2/sessionflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
