## camera-fitness-app

> Browser-based fitness coach — a Wii-Fit-style app that uses the user's webcam to count reps, deliver a structured workout plan, and track progress over time. Runs entirely client-side: no server, no account, no telemetry. The user installs nothing; they open a URL.

# camera-fitness-app

## Project mission

Browser-based fitness coach — a Wii-Fit-style app that uses the user's webcam to count reps, deliver a structured workout plan, and track progress over time. Runs entirely client-side: no server, no account, no telemetry. The user installs nothing; they open a URL.

This repo is the successor to a Python/pygame prototype (`FitnessApp`) that proved out the core ideas (camera-driven rep counting, progression systems, trainer dialogue). The prototype is preserved at the legacy path below for reference; this repo replaces it.

---

## Stack

| Layer | Choice |
|---|---|
| Framework | **React 18 + TypeScript + Vite** |
| Pose tracking | **`@mediapipe/tasks-vision`** (Google's official JS/WASM build of MediaPipe Pose Landmarker) |
| State | **Zustand** (one store per data domain) |
| Persistence | **IndexedDB** via the [`idb`](https://github.com/jakearchibald/idb) wrapper |
| Styling | **Tailwind CSS** (subject to review during UI design pass) |
| Audio | **Web Audio API** for voice clips and music |
| Camera | `navigator.mediaDevices.getUserMedia()` + `<video>` + `<canvas>` overlay |
| Build | Static — deployable to GitHub Pages, Vercel, Netlify, or any static host |

No backend. No auth. Everything the user creates lives in their browser's IndexedDB.

---

## Architecture map (legacy → new)

The legacy pygame repo is read-only reference material. When porting any concept, **read the legacy file first** and link it in a header comment in the new TS file.

| Concern | Legacy file (in pygame repo) | New location |
|---|---|---|
| Athlete persistence (coins, history, ORM, progression state, awarded session ids) | `data/athlete.py` | `src/data/athlete/athlete.ts` (IndexedDB-backed) |
| Plan + workout-day schema | `data/plans.py`, `data/workout_plans.py` | `src/data/plans/` |
| Progression strategies (Linear, 5/3/1, Boring But Big) | `data/progressions/{base,linear,five_three_one,boring_but_big}.py` | `src/data/progressions/` |
| App settings (theme, volumes, rest seconds, weight step, trainer toggle) | `data/settings.py` | `src/data/settings/settings.ts` |
| Body measurements | `data/body.py` | `src/data/body/` |
| Calibration profile + threshold math | `data/calibration.py` | `src/data/calibration/` |
| Trainer system (the `Trainer` dataclass + `line()` method **only**) | `data/trainers.py` | `src/data/trainers/trainer.ts` |
| Camera frame loop | `tracking/camera.py` | `src/hooks/useCamera.ts`, `src/hooks/useMediapipe.ts` |
| Angle / landmark helpers | `tracking/helpers.py` | `src/tracking/helpers.ts` |
| Rep counters (per exercise) | `tracking/exercises/*.py` | `src/tracking/exercises/*.ts` |
| All UI scenes | `scenes/*.py` | `src/components/<scene>/` — **not 1:1; UI is being redesigned** |

---

## What is **not** migrated

- **The "Ellie" trainer.** Her sprite (`assets/trainers/ellie/sprite.png`), voice clips, and the `ellie = Trainer(...)` definition in `data/trainers.py` are out of scope. The `Trainer` *system* is migrating; the *character* is not. Replacement trainers will be designed in a future pass.
- **All pygame draw code in `scenes/`.** The UI is being redesigned as React components, not ported pixel-for-pixel.
- **The legacy RenPy `.rpy` files under `game/`.** Dead code in the prototype.
- **`pygame.mixer` audio code.** Replaced by Web Audio API.
- **The Python virtualenv (`venv/`)** and any `__pycache__/`.
- **The prototype's `saves/*.json`.** Users will start fresh in the browser.

---

## Development rules

1. **All app code lives under `src/`.** No top-level `.ts`/`.tsx` files except `src/main.tsx`.
2. **Persistence goes through IndexedDB** via the per-domain stores in `src/data/*`. The only acceptable `localStorage` usage is small UI preferences (theme, last active tab) that are fine to lose.
3. **Pose tracking math is ported verbatim** from `tracking/exercises/*.py`. No re-derivation, no "improvements" during the port. If something looks wrong, fix it in a separate commit so the diff with the legacy code is auditable. Each ported TS file MUST include a header comment of the form:
   ```ts
   // Ported from: tracking/exercises/<file>.py (legacy FitnessApp repo)
   // Last sync: <git sha of legacy repo at port time>
   ```
4. **The `Trainer` interface stays open-ended.** Adding a new trainer is one `.ts` file that exports an object matching the `Trainer` type — same shape the Python dataclass had: name, sprite path, voice directory, and dialogue pools per category.
5. **No backend.** If a feature needs a server, it doesn't ship.
6. **MediaPipe `.task` model files live under `public/models/`** so they're served as static assets at known URLs.

---

## Repository layout

```
camera-fitness-app/
├── .claude/
│   ├── launch.json              # dev server config (npm run dev → :5173)
│   └── settings.json            # permissions, hooks (empty for now)
├── public/
│   └── models/                  # MediaPipe .task files
├── src/
│   ├── components/              # React components (was scenes/)
│   ├── data/
│   │   ├── athlete/
│   │   ├── body/
│   │   ├── calibration/
│   │   ├── plans/
│   │   ├── progressions/
│   │   ├── settings/
│   │   └── trainers/            # Trainer system, no Ellie
│   ├── tracking/
│   │   └── exercises/           # rep counters
│   ├── assets/
│   │   └── trainers/            # sprites, voice clips per trainer
│   ├── hooks/                   # useCamera, useMediapipe, ...
│   ├── stores/                  # Zustand stores
│   └── main.tsx                 # entry (not yet written)
├── CLAUDE.md                    # this file
├── README.md
├── .gitignore
├── package.json                 # stub — no deps installed yet
├── tsconfig.json
├── vite.config.ts               # stub — no plugins yet
└── index.html
```

---

## Development quickstart

```bash
npm install        # works against the stub package.json (no deps yet, so instant)
npm run dev        # will fail until src/main.tsx is added in a future pass
npm run build      # likewise
```

The current scaffold is intentionally a no-op: it locks in the structure and the docs so the actual porting work can be planned and executed in clean, scoped passes.

---

## Migration phases (high level)

| Phase | Scope | Status |
|---|---|---|
| 0 | Scaffold + docs (this commit) | ✅ done |
| 1 | Install React + Vite + TS deps; wire `src/main.tsx`; hello-world page renders | pending |
| 2 | Port pure data models (`athlete`, `settings`, `plans`, `progressions`, `body`, `calibration`) to TS with IndexedDB-backed stores | pending |
| 3 | Port the `Trainer` system + define at least one replacement trainer | pending |
| 4 | Port `tracking/helpers.ts` + `useCamera` + `useMediapipe` hooks; demo page shows a live skeleton overlay | pending |
| 5 | Port rep counters (`bicep_curls`, `squat`, `lateral_raise`, `generic`) | pending |
| 6 | Design + build the React UI (home, training, rest, complete, stats, settings, plans, calibration) | pending |
| 7 | Audio: voice + music via Web Audio API | pending |
| 8 | Deploy (static host) | pending |

Each phase gets its own planning pass; this file is the durable reference, not a per-phase tracker.

---

## Legacy reference

The prototype lives at `D:\Lucas\Programas\github\minigames\FitnessApp`. Treat it as read-only history. Last commits at scaffold time:

- `ea06751` — Fix calibration `_STEPS` unpack and add launch/calibration defaults
- `19e6a11` — Add AMRAP sets, smart Quick Start, rest settings, and fix all tech debt

When porting, run the prototype side-by-side (`venv\Scripts\python.exe main.py`) so behaviour matches.

---
> Source: [lucascantos/camera-fitness-app](https://github.com/lucascantos/camera-fitness-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
