## workout-tracker

> > Last updated: May 6, 2026 (Batch 61 — Machine chip persistence fix: v11 library backfill + inheritance promotion + gym-prefer pass-3 fallback)

# Gains — Project State

> Last updated: May 6, 2026 (Batch 61 — Machine chip persistence fix: v11 library backfill + inheritance promotion + gym-prefer pass-3 fallback)

## Rules for Claude

1. **Read this file first** at the start of every new session before doing anything else.
2. **Update this file** after every batch of changes. Add new features to "Recent Changes", update file structure if files were added/removed, and update store shape if state changed. Update the "Last updated" date.
3. **Git is fully writable from the sandbox.** Claude runs `git` directly — creates worktrees, commits, pushes feature branches, and merges to main. Never `--force` push. Never skip hooks (`--no-verify`).
4. **Validate builds** with `npx vite build --outDir /tmp/test-build`. Never build to the mounted `dist/` folder (Vite can't emit there — EPERM).
5. **Run `npm run lint` before every commit.** ESLint is wired with `react-hooks/rules-of-hooks` + `no-undef` as errors — these catch the two bug classes that crashed the app on April 26 (dangling identifier ref + hook after early return). Exit code must be 0 before merging. `exhaustive-deps` and `no-unused-vars` stay as warnings (visible but non-blocking).
6. **Feature branches for non-trivial changes.** Not for review (user doesn't review), but to give a clean revert point and a Vercel preview URL before merging to main. Small fixes can go straight to main.

## Pre-flight checklist for redesign batches

The April 26 crash bugs (`missedYesterdayWorkout` dangling ref + `CreateExerciseModal` hooks-after-early-return) both slipped past ~12 polish rounds because verification was happy-path-only — the dev loop only exercised one split type and one modal-open state. Run this checklist before merging any redesign batch:

- [ ] `npm run lint` → exit 0 (errors block; warnings are fine).
- [ ] `npx vite build --outDir /tmp/test-build` → success.
- [ ] Walk the redesigned surface in preview against THREE state combinations:
  - [ ] **Empty data** (`sessions: []`, no splits beyond built-in).
  - [ ] **Populated weight-only data** (BamBam's Blueprint active, sessions logged).
  - [ ] **HYROX-active data** (HYROX Hybrid active, with or without sessions).
- [ ] For surfaces with rotation/calendar logic: test today=rest AND today=workout (HYROX Hybrid Sunday=rest is a quick toggle).
- [ ] For surfaces with modals/sheets: open it, close it, re-open it. Watch console for "Rendered more hooks than during the previous render."
- [ ] Open browser console → verify zero new errors AND zero new warnings.
- [ ] If the surface reads `theme.hex` for foreground colors: test daylight + a light accent (the pale-green case is the known-bad combination).

The top-level `ErrorBoundary` (April 27) catches any render error and shows a recovery banner with the stack trace + Reload + recovery-page link, so the worst-case future bug is contained — but the checklist still matters because the boundary is recovery, not prevention.

---

## Overview

**Gains** — a mobile-first PWA for tracking weight training and cardio sessions. Built around a customizable split rotation system with automatic workout progression, PR tracking, session grading, and detailed history/stats.

**Live URL:** Deployed to Vercel via GitHub auto-deploy from `main` branch.
**Repo:** `github.com/bradenlanham/workout-tracker`
**Primary user:** Braden

---

## Tech Stack

- **Framework:** React 18.3.1 + React Router v6 (HashRouter)
- **State:** Zustand 4.5.2 with `persist` middleware (localStorage key: `workout-tracker-v1`)
- **Styling:** Tailwind CSS 3.4.3 with CSS custom properties for theming
- **Charts:** Recharts 2.12.2
- **Build:** Vite 5.2.0
- **Image export:** html2canvas (for share card JPEG export)
- **No backend.** All data lives in localStorage. Export/import via JSON backup files.

---

## File Structure

```
src/
├── App.jsx                    # HashRouter, route definitions, theme init, split auto-creation
├── main.jsx                   # React root mount
├── index.css                  # Tailwind config + CSS variables (obsidian/daylight themes)
├── theme.js                   # 10 accent color definitions (violet, blue, emerald, orange, rose, cyan, red, pink, white, black)
│
├── store/
│   └── useStore.js            # Zustand store — ALL app state + actions
│
├── data/
│   ├── exercises.js           # Built-in "BamBam's Blueprint" workout data (5 workouts, exercise groups)
│   ├── exerciseLibrary.js     # 140+ exercises by muscle group for the exercise picker
│   ├── splitTemplates.js      # Batch 17f — 6 curated templates for ChooseStartingPoint (BamBam / FullBody×3 / Upper-Lower×4 / PPL×3 / PPL×6 / Bro / 5×5) + loadTemplateForDraft(id)
│   └── hyroxStations.js       # Batch 37 — closed catalog of 8 HYROX stations (sta_skierg, sta_sled_push, sta_sled_pull, sta_burpee_broad, sta_row, sta_farmers, sta_sandbag_lunges, sta_wall_balls). Each carries id/name/dimensions/raceStandard. buildHyroxStationLibraryEntry(station) is the canonical converter consumed by buildBuiltInLibrary + migrateLibraryToV8.
│
├── utils/
│   └── helpers.js             # getNextBbWorkout, getLastBbSession, perSideLoad, getExercisePRs, isSetPR, isPR,
│                              # getWorkoutStreak, getBestStreak, getRotationItemOnDate, getAchievements,
│                              # migrateSessionsToV2, migrateSessionsToV3, normalizeExerciseName,
│                              # similarExerciseScore, findSimilarExercises, formatDate/Time, playBeep, generateId,
│                              # e1RM, percent1RM, getExerciseHistory, getCurrentE1RM, getProgressionRate,
│                              # getRecommendationConfidence, recommendNextLoad   ← recommender engine (Batch 16a)
│                              # detectPlateau, detectRegression, detectSwing, detectAnomalies  ← anomaly detectors (Batch 16q)
│                              # getWorkoutStreak signature: (sessions, cardioSessions, restDaySessions)
│                              # getBestStreak    signature: (sessions, cardioSessions, restDaySessions)
│                              # perSideLoad(set): canonical per-side load accessor — rawWeight ?? weight ?? 0
│                              # findSimilarExercises(query, library, opts): trigram+token-sort fuzzy match
│                              # recommendNextLoad({history, targetReps, mode, progressionClass, loadIncrement})
│                              #   → { mode, confidence, prescription: {weight, reps}, reasoning, meta }
│                              # detectAnomalies(history): runs all three detectors, returns highest-priority
│                              #   hit or null; priority: regression > swing > plateau
│                              # predictExerciseMeta(name): Batch 17j — keyword-based best-effort
│                              #   guess of {primaryMuscles, equipment} from an exercise name, or null
│                              #   if nothing matches. Feeds CreateExerciseModal's auto-fill.
│                              # normalizeExerciseEntry(ex): Batch 18a — lossless exercise-shape
│                              #   normalizer used by SplitCanvas + WorkoutEditSheet. Accepts
│                              #   string / {name} / {name, rec} / {exercise} and returns the
│                              #   smallest renderable shape or null for truly nameless entries.
│                              # getSplitSessionCount, getSplitLastUsedDate: Batch 18b — split
│                              #   usage stats for the SplitManager card. Pure, session-slice input.
│                              # formatRelativeDate(iso), formatStartDate(isoOrDateStr): Batch 18b —
│                              #   SplitCard date formatters. formatStartDate local-parses
│                              #   date-only strings to avoid the Batch 16k timezone drift.
│                              # (theme.js also exports getSaveTheme(accentColor) as of Batch 18e —
│                              #   red→emerald fallback for Save/commit buttons.)
│                              # getExerciseHistory(sessions, id, name?, equipmentInstance?):
│                              #   Batch 19 — optional fourth arg scopes history to sessions
│                              #   tagged with that instance (case-insensitive). Unscoped
│                              #   when null/empty. Each history item also echoes
│                              #   equipmentInstance so detectors can inspect machine context.
│                              # getInstancesForExercise(sessions, id, name?, gymId?):
│                              #   Batch 19 / Batch 20 — distinct non-empty
│                              #   equipmentInstance strings for this exercise,
│                              #   most recent first, case-insensitive dedupe.
│                              #   Batch 20 adds optional gymId scoping so the
│                              #   picker at VASA doesn't show TR's machines.
│                              # getExerciseHistory(..., equipmentInstance?, gymId?):
│                              #   Batch 20 adds a 5th optional arg — when set,
│                              #   only sessions whose session.gymId matches
│                              #   contribute. Legacy (pre-16n) sessions without
│                              #   a gymId never match a scoped query (§3.5.6 —
│                              #   "unspecified" stays unspecified). Callers do
│                              #   the <3-session widen-scope fallback themselves.
│                              #   History items echo gymId on each entry.
│                              # isExerciseAvailableAtGym(exercise, gymId),
│                              # shouldSkipGymTagPrompt(exercise, gymId),
│                              # shouldPromptGymTag(exercise, gymId): Batch 20 —
│                              #   three pure readers for the §3.5 gym-tagging
│                              #   rules. Empty / missing sessionGymTags = "available
│                              #   everywhere", skipGymTagPrompt silences the
│                              #   auto-tag prompt per (exercise, gym) pair.
│                              # classifyType(name): Batch 37 — keyword-based type
│                              #   classifier returning 'weight-training' |
│                              #   'running' | 'hyrox-station' | 'hyrox-round'.
│                              #   Order matters: composite-round terms ("hyrox
│                              #   round", "run + skierg") fire before station
│                              #   singletons ("skierg") so composites win.
│                              # defaultDimensionsForType(type): Batch 37 —
│                              #   dimension preset per type. Five axes:
│                              #   weight | reps | distance | time | intensity.
│                              # migrateLibraryToV8(library): Batch 37 — v7→v8
│                              #   library migration, idempotent. Adds type +
│                              #   dimensions to every entry, seeds the 8 HYROX
│                              #   stations if missing, preserves any user-created
│                              #   entry whose id collides with a station id.
│                              # predictExerciseMeta(name): Batch 37 — extended
│                              #   to return { primaryMuscles, equipment, type }.
│                              #   Falls back to type-only result when no muscle/
│                              #   equipment keyword matches but classifyType
│                              #   returns a non-default type.
│                              # lbsToKg / kgToLbs / milesToMeters / metersToMiles:
│                              #   Batch 38 — unit conversions per design doc
│                              #   §11.2 (1 lb = 0.45359237 kg, 1 mi = 1609.344
│                              #   m). 3-decimal precision; defensive against
│                              #   null / non-numeric / undefined.
│                              # migrateSessionsToV9(sessions): Batch 38 — v8→v9
│                              #   session migration, idempotent. Walks every
│                              #   weight-training set (top-level + nested
│                              #   drops) and derives weightKg + rawWeightKg
│                              #   alongside the existing lbs fields. Sets
│                              #   already carrying weightKg are skipped.
│                              # migrateCardioSessionsToV9(cardioSessions):
│                              #   Batch 38 — cardio v8→v9 migration. Adds
│                              #   distanceMiles + distanceMeters when
│                              #   distanceUnit === 'miles'. 'floors' / null
│                              #   units pass through unchanged.
│
├── components/
│   ├── BottomNav.jsx          # 4-tab nav: Dashboard, Log, History, Progress (hidden during logging)
│   ├── HamburgerMenu.jsx      # Slide-in menu: My Splits, Progress, Settings, Info (incl. "How Streaks Work"), Manage Data
│   ├── RestTimer.jsx          # Floating draggable rest timer with progress ring (visible only during logging)
│   ├── CustomNumpad.jsx       # Numpad overlay used by BbLogger for weight/reps entry
│   ├── CreateExerciseModal.jsx # Shared modal for adding a new library entry. Batch 17j: 300ms debounced auto-predict on name (predictExerciseMeta) + "Skip for now — tag later" button that calls onSkip({name}) when provided by the caller.
│   ├── ExerciseEditSheet.jsx   # Bottom-sheet editor for existing library entries (Batch 16j)
│   ├── Toast.jsx               # Batch 17e — event-bus undo toast; showToast({message, undo}) from anywhere
│   ├── RestDayChip.jsx         # Batch 17f — shared dashed-circle "R" rest chip (D3); size prop
│   ├── EmojiPicker.jsx         # Batch 17g — curated 32-emoji grid + OS fallback via paste input (D5)
│   ├── WorkoutEditSheet.jsx    # Batch 17g + 17i — bottom-sheet editor for a single workout; editable section labels, structured rec editor (via RecPill + RecEditor as of 17h), accepts recentInSplit prop and renders the shared ExercisePicker component.
│   ├── ExercisePicker.jsx      # Batch 17i — shared exercise picker extracted from WorkoutEditSheet's inline version. Adds "Recent in this split" tab (union of exercises from sibling workouts) + "Search all muscles" checkbox (default on). Drives both the Canvas sheet and any future logger surface.
│   ├── RecPill.jsx             # Batch 17h — shared rec-display pill; renders formatRec(rec) in any supported shape
│   ├── RecEditor.jsx           # Batch 17h — structured rec editor sheet (sets / reps / note with live preview)
│   ├── DragHandle.jsx          # Batch 18c — shared decorative drag-handle glyph. REMOVED from usage in Batch 18f (no real drag wired; it suggested an interaction that didn't exist). Kept on disk for when DnD ships.
│   ├── RowOverflowMenu.jsx     # Batch 18e — shared row-level ⋯ popover. Batch 18f portals the panel to document.body with position:fixed so it can't be clipped by an ancestor overflow-hidden. Accepts items: [{label, onSelect, destructive?, icon?}]; null onSelect filters the item out so boundary Move up/down vanish cleanly. Auto-flips above the anchor when it would overflow the viewport bottom.
│   └── SupersetSheet.jsx       # Batch 36 — superset initiate/re-pair/active sheet (z-245). Three variants driven by props: Initiate (partner picker, max 2), Re-pair (last-paired hint + Re-pair primary + Customize secondary), Active (cycle members + End superset). Pure presentation; parent owns state.
│
│
└── pages/
    ├── Welcome.jsx            # Onboarding: name entry, split choice (Blueprint or build own), theme, import
    ├── Dashboard.jsx          # Main screen: greeting, 6 stat cards, CTA card, soreness check-in, weekly calendar, workout preview
    ├── Log.jsx                # Workout picker: shows rotation, split order editor modal, custom template list
    ├── History.jsx            # Calendar heatmap (13 weeks), session list by date, session detail modal, share card
    ├── Progress.jsx           # Bar chart of session volume (last 15 sessions via Recharts)
    ├── Guide.jsx              # Static training guide (sets, reps, hypertrophy tips)
    ├── CardioLogger.jsx       # Cardio session logger: type selection, stopwatch, manual entry, HR, distance, intensity
    ├── SplitManager.jsx       # List all splits, set active, edit, clone, export/import, delete
    ├── SplitCanvas.jsx        # Batch 17g + 17k — sole split editor (the /splits/new + /splits/edit/:id surface); identity + workouts + rotation always visible, sticky footer, cycle/week rotation toggle (D6), ⋮ overflow for Export/Delete. Computes `recentInSplit` (sibling-workout exercises) and threads to WorkoutEditSheet as of 17i.
    ├── ChooseStartingPoint.jsx # Batch 17f — /splits/new/start template chooser (6 templates + Blank + Import)
    ├── TemplateEditor.jsx     # Create/edit custom workout templates (legacy, pre-splits feature)
    ├── Backfill.jsx           # One-time tagging screen for library entries with needsTagging: true.
    │                          # Per-card draft state + Confirm button (Batch 16j) — auto-save
    │                          # reverted so users get an explicit "I'm done" before the card
    │                          # drops off the list.
    ├── ExerciseLibraryManager.jsx # /exercises — list/filter/search/edit all library entries
    │                          # via ExerciseEditSheet. Batch 16j.
    │
    └── log/
        ├── BbLogger.jsx       # THE MAIN SESSION LOGGER — exercise cards, sets, plates mode, uni toggle,
        │                      # previous session ghost rows, add exercise panel, finish modal, session comparison,
        │                      # auto-persist to split template, share card integration
        ├── Recommendation.jsx # Recommender UI: RecommendationChip (toolbar-row "Tip" pill,
        │                      # 16n-1), RecommendationSheet (bottom-sheet w/ one-line header
        │                      # "Recommended top set: W×R" + exercise name above + last-session
        │                      # subtitle, e1RM sparkline w/ explicit Peak/Growth key, compact
        │                      # 1-line mode chips, tap-to-explain confidence %, plain-English
        │                      # reasoning, expandable Details section w/ "vs last session" delta,
        │                      # accepts aggressivenessMultiplier + defaultMode from readiness
        │                      # check-in), AnomalyBanner (Batch 16q — inline card banner for
        │                      # plateau/regression/swing, severity-keyed tint, dismiss-per-session
        │                      # X; rendered between toolbar row and column headers),
        │                      # GymTagPrompt (Batch 20b — accent-tinted banner inside the
        │                      # expanded exercise card surfacing "Tag X as available at Y?"
        │                      # with Yes / Not this time / Always skip actions per §3.5.4;
        │                      # rendered above AnomalyBanner so tagging decisions come first).
        │                      # Batches 16b + 16f + 16n + 16n-1 + 16q + 20b.
        │                      # Exports: RecommendationChip, RecommendationSheet, AnomalyBanner, GymTagPrompt.
        ├── ReadinessCheckIn.jsx # Batch 16n / 35 — pre-session overlay (§2.5). Prominent GYM row
        │                      # at top + Energy / Sleep / Goal + Exercise Order rows.
        │                      # Goal→mode, Energy+Sleep→multiplier, orderMode flows to
        │                      # BbLogger's handleStartSession which reorders exercises by
        │                      # last session's completedAt within each section when set to
        │                      # 'lastSession'. Defaults Mid/Mid/Push/Default = no-op (mult 1.0,
        │                      # template order preserved). Skip link + Go back.
        ├── SessionGymPill.jsx # Batch 20a — compact in-session gym indicator shown
        │                      # below the workout title in BbLogger. Hidden when
        │                      # no gyms configured. Tap opens a portal popover
        │                      # (z-240) with the gym list + add-new input + a
        │                      # "Clear gym for this session" button. Makes the
        │                      # gym-driven AI inferences (Machine chip auto-fill,
        │                      # recommender history scoping) visible per user UX.
        ├── ShareCard.jsx      # Trading card share card: 5 tiers by streak, selfie, stats bar, JPEG export
        └── CameraCapture.jsx  # Selfie camera component for share card
```

---

## Zustand Store Shape (`useStore.js`)

```javascript
{
  // ── Sessions ──
  sessions: [],                    // Array of completed workout sessions
  activeSession: null,             // In-progress workout (survives reload/backgrounding)

  // ── Settings ──
  settings: {
    restTimerDuration: 90,         // Seconds
    accentColor: 'violet',         // Theme key (see theme.js)
    backgroundTheme: 'obsidian',   // 'obsidian' | 'daylight'
    userName: '',
    autoStartRest: false,          // Auto-start rest timer after logging a working set
    defaultFirstSetType: 'warmup', // 'warmup' | 'working'
    restTimerChime: true,          // Play beep when timer ends
    hasSeenTutorial: false,        // Dashboard tutorial overlay
    enableAiCoaching: true,        // Batch 16i — gates the recommender UI (banner + sheet)
    showRecPill: true,             // Batch 16i — gates the per-exercise blue REC chip
    gyms: [],                      // Batch 16n — [{id, label}] — populated inline from readiness chip picker
    defaultGymId: null,            // Batch 16n — last-selected gym (seeds the chip next session)
    dismissedAnomalies: {},        // Batch 16q — { [exerciseKey]: sessionId } — anomaly-banner dismissals scoped to current active session
    dismissedGymPrompts: {},       // Batch 20b — { [`${exerciseId}:${gymId}`]: sessionId } — "Not this time" dismissals of the auto-tag-on-use prompt, session-scoped. "Always skip" writes to Exercise.skipGymTagPrompt instead (persists).
  },

  // ── Splits ──
  splits: [],                      // Array of split objects (built-in + user-created)
  activeSplitId: null,             // Currently active split ID
  splitDraft: null,                // Batch 17a — in-progress wizard/canvas draft; { originalId, draft, updatedAt } | null
  exerciseLibrary: [],             // Canonical Exercise[] — seeded by initLibrary() on mount
                                   // from data/exerciseLibrary.js + extended by the v2→v3
                                   // migration with needsTagging:true entries for any
                                   // user-created session names that didn't resolve.
                                   // Entry shape: { id, name, aliases, primaryMuscles[],
                                   // equipment, isBuiltIn, defaultUnilateral, loadIncrement,
                                   // defaultRepRange, progressionClass, needsTagging, createdAt,
                                   // sessionGymTags?: string[],      // Batch 20, spec §3.5:
                                   //                                   gymIds where this
                                   //                                   exercise IS available.
                                   //                                   Empty/missing = "universally
                                   //                                   available / unspecified".
                                   // skipGymTagPrompt?: string[] }   // Batch 20, spec §3.5.4:
                                   //                                   gymIds where the user
                                   //                                   chose "Always skip" on
                                   //                                   the auto-tag prompt.

  // ── Legacy ──
  customTemplates: [],             // Pre-splits custom workout templates
  workoutSequence: null,           // Legacy rotation order (synced to active split)

  // ── Cardio ──
  cardioSessions: [],              // Standalone + attached cardio sessions
  customCardioTypes: [],           // User-added cardio type strings
  activeCardioSession: null,       // In-progress cardio

  // ── Rest Day Logging ──
  restDaySessions: [],             // Explicitly logged rest days { id, date: 'YYYY-MM-DD', note? }

  // ── Timer ──
  restEndTimestamp: null,          // ms timestamp when rest timer expires

  // ── Onboarding ──
  hasCompletedOnboarding: false,
}
```

### Session Object

```javascript
{
  id: 'generated-id',
  date: '2026-04-01T14:30:00.000Z',
  mode: 'bb',                          // 'bb' for weight training
  type: 'push' | 'legs1' | 'generated-id',  // workout ID from split rotation
  duration: 45,                         // Minutes
  grade: 'A+' | 'A' | 'B' | 'C' | 'D' | null,
  completedCardio: true | false,
  cardio: { completed, type, duration, heartRate, notes },
  notes: '...',
  selfie: 'data:image/...' | null,
  soreness: { rating: 'notsore'|'sore'|'verysore'|'wrecked', date: '2026-04-01' } | null,
  readiness: {                          // Batch 16n — present when user answered the check-in
    energy: 'low'|'ok'|'high',
    sleep: 'poor'|'ok'|'good',
    goal: 'recover'|'match'|'push',
    aggressivenessMultiplier: 0.85|1.00|1.15,   // energy + sleep → scales push-mode nudge
    suggestedMode: 'deload'|'maintain'|'push',   // goal → recommender mode
    timestamp: '...',
  } | null,                             // null when Skip check-in tapped
  gymId: 'gym_...' | null,              // Batch 16n — where the session was logged
  data: {
    workoutType: 'push',
    exercises: [
      {
        name: 'Bench Press',            // Canonical name — matches a library entry
        exerciseId: 'ex_bench_press',   // Stable library link (added in Batch 15)
        notes: '...',
        completedAt: timestamp,
        unilateral: false,              // If true, volume was doubled at save time
        equipmentInstance: 'Hoist',     // Batch 19 — optional §3.4 machine tag
                                        // (e.g. "Hoist leg press" vs "Cybex leg press").
                                        // Omitted when blank. Picker seeds from last session.
        sets: [
          {
            type: 'warmup' | 'working' | 'drop',
            reps: 8,
            weight: 185,                // Already doubled if unilateral
            rawWeight: 92.5,            // Original input (only present if unilateral)
            isNewPR: false,
            plates: { 45: 2, 25: 1, ... },  // Only present if logged in plate mode
            barWeight: 45,                    // Only present if logged in plate mode
            // rpe: 8                          // (16c shipped + reverted in 16d — engine
            //                                  //  still reads rpe if present, but no UI
            //                                  //  captures it; field no longer saved)
          }
        ]
      }
    ]
  }
}
```

### Split Object

```javascript
{
  id: 'split_bam' | 'generated-id',
  name: "BamBam's Blueprint",
  emoji: '🏋️',
  isBuiltIn: true | false,
  createdAt: '2026-03-22',
  workouts: [
    {
      id: 'push' | 'generated-id',
      name: 'Push — Chest',
      emoji: '🏋️',
      sections: [
        {
          label: 'Primary',
          exercises: ['Pec Dec', 'Incline DB Press', ...]  // String names
        },
        { label: 'Choose 1', exercises: [...] },
        { label: 'If You Have Time', exercises: [...] }
      ]
    }
  ],
  rotation: ['push', 'legs1', 'rest', 'pull', ...],  // Workout IDs + 'rest'
}
```

---

## Built-in Split: BamBam's Blueprint

5 workouts in default rotation order: `push → legs1 → pull → push2 → legs2`

| ID | Name | Emoji | Primary Exercises |
|---|---|---|---|
| push | Push — Chest | 🏋️ | Pec Dec, Incline DB Press, Flat Bench Press |
| legs1 | Legs 1 — Quads | 🦵 | Leg Curls, Leg Extensions |
| pull | Pull — Back | 💪 | Single Arm Lat Pulldown, Single Arm Row, Chest Supported Wide Row |
| push2 | Push 2 — Shoulders & Arms | 🎯 | Rear Delts, DB Lateral Raises, Military Press |
| legs2 | Legs 2 — Hams | 🦿 | Seated Leg Curl, DB Romanian Deadlifts |

Each workout has 3 sections: "Primary" (always do), "Choose 1" (pick one), "If You Have Time" (optional).

---

## Key Features & How They Work

### Workout Rotation & Progression
- `getNextBbWorkout()` finds the last logged session type in the rotation and returns the next one (skipping rest days).
- `getRotationItemOnDate()` calculates what workout falls on any given date (including rest days) using the most recent session as an anchor.
- Rest days in the rotation are markers that prevent streak-breaking on off days.

### Session Logger (BbLogger.jsx) — The Core
- **Initialization:** Loads exercises from the active split's workout template. Also merges in any exercises from the last session of the same type that aren't in the template (ensures custom exercises persist).
- **Active session persistence:** Every change to exercises or notes auto-saves to `activeSession` in the store, surviving page reload and app backgrounding.
- **Plate mode:** Per-exercise toggle. Shows plate buttons (100, 45, 35, 25, 10, 5, 2.5) with a bar weight cycler (45/0/25). A **1× / 2× multiplier toggle** controls whether plates are counted once or on both sides (default: 2×). Plate breakdown data is saved to the session for accurate ghost row display next time. **Plates toggle is pre-selected** from last session's data (`plateLoaded` state initialized from last session).
- **Unilateral (Uni) toggle:** Per-exercise. When active, volume is doubled at save time. The doubled weight is stored in `weight`, original in `rawWeight`. **Uni toggle is pre-selected** from last session's `unilateral` flag.
- **Previous session ghost rows:** Shows last session's sets as faded, non-interactive rows. Ghost rows display `rawWeight` (not doubled weight) for unilateral exercises. Includes plate breakdown if logged in plate mode. Toggled by "Last Time" button per exercise.
- **Set types:** warmup / working / drop. Drop sets auto-suggest 75% of previous working set weight.
- **PR detection:** Weight-anchored — a set is a PR only if (a) its weight exceeds the all-time max weight for that exercise, or (b) its weight equals the all-time max and reps exceed the best rep count ever achieved at that max weight. Reps alone at sub-max weight are never a PR. Single source of truth lives in `isSetPR()` in `helpers.js` and is used by the live per-set trophy, the card-level trophy, and the saved `isNewPR` flag. An amber "PR 185×8" chip on every exercise card surfaces the all-time max at a glance.
- **Auto-sync custom exercises to split:** After saving, any exercises that were in the session but not in the template get added to the appropriate section of the split definition. Placement is intelligent — inserts near the exercises it was positioned next to during the session.
- **Finish flow:** Grade picker → optional cardio attachment (attach today's cardio, log inline, or log now via CardioLogger) → **Session Comparison screen** (shows volume % diff per exercise vs last session, green ↑ / red ↓) → Share Card.
- **Session timer:** Timestamp-based, survives backgrounding via `visibilitychange` listener.

### Grade System
- Grades: D, C, B, A, A+ (assigned at session finish or edited later in History).
- Grades drive the color of heatmap cells in History: A+ = accent color, A = emerald, B = amber, C = red, D = dark red.
- **Editable after save:** The grade badge in History session detail is tappable and shows a picker to set/change grade anytime.

### Share Card (ShareCard.jsx)
- **Fixed-viewport trading card** — always fits one screen (max 380px wide, no scrolling).
- **5 rarity tiers** based on workout streak from `getWorkoutStreak()`:
  - Common (0–5): user's accent color border/glow
  - Rare (6–14): silver
  - Epic (15–19): gold
  - Legendary (20–49): animated shimmer gradient border (orange/red/gold)
  - Mythic (50+): animated shimmer border (purple/cyan/pink) + sparkle particles
- **Layout (top to bottom):** tier badge row → 4:3 photo window (selfie or placeholder) → workout title + meta → exercise list (max 6 + overflow) → stats bar (VOL / SETS / STREAK 🔥 / GRADE).
- **JPEG export:** Share button captures card via `html2canvas` at 2× scale, exports via Web Share API (iOS) or download fallback.
- **Share/Done buttons** are outside the `cardRef` div and not captured in the JPEG.
- **Data shape:** `data` prop includes `streak` (number) in addition to the standard fields. Both `BbLogger.jsx` and `History.jsx` pass `streak: getWorkoutStreak(sessions, activeSplit?.rotation)`.
- **Dependency:** `html2canvas` (installed).

### Session Comparison (in BbLogger.jsx)
- Shown automatically after saving a session (before share card), if a previous session of the same workout type exists.
- Per-exercise volume comparison with % change (green ↑ increase, red ↓ decrease).
- Overall volume diff prominently displayed at the top.
- "Continue" button advances to share card. Skipped entirely if no previous session exists.

### History (History.jsx)
- **Calendar heatmap:** 13 weeks × 7 days. Color-coded by best session grade that day.
- **Session list:** Grouped by date, newest first. Shows workout name, emoji, exercise count, set count, time, grade.
- **Session detail modal:** Exercise breakdown with sets, attached cardio, soreness, notes.
- **Workout name resolution:** Uses `resolveWorkoutName()` / `resolveWorkoutEmoji()` which check all splits in the store (not just built-in names). Fixes display of user-created split workout names.

### Dashboard (Dashboard.jsx)
- **Greeting:** Time-based ("Good morning/afternoon/evening, {name}") with increased top padding for proper spacing.
- **6 stat cards** (compact layout): Last Week Volume, This Week Volume, Sessions (Split), Day Streak, Total Sessions, PRs This Split.
- **Active split label:** Shows current split name and emoji (no "X/7 days" text — removed).
- **Main CTA card:** Shows next workout in rotation with "Start Session" button. Shows "Rest Day" message on rest days. "Preview" button shows the workout's exercise list. Increased gap between Preview and Start Session buttons.
- **Soreness check-in:** Prompted the day after a workout (yesterday's session). `yesterdayLogged` checks sessions, cardioSessions, AND restDaySessions. Ratings: Not Sore, Sore, Very Sore, Wrecked. Can be skipped.
- **Action buttons row:** "Log Cardio" and "Log Rest Day" buttons side by side below the CTA card.
- **StreakBadge:** Always rendered (even at 0, shows "0🔥"). Not conditionally hidden.
- **Weekly calendar strip:** Sun–Sat with emoji indicators for completed, planned, and rest days. Explicitly logged rest days show a brighter dot. Tapping a future day shows workout preview.
- **Monthly calendar view** (toggleable): Full month view.
- **Tutorial overlay:** 4-step walkthrough on first visit.

### Cardio Logger (CardioLogger.jsx)
- Type selection: Treadmill, Stairmaster, Running, Walking, Assault Bike, Other + user-added custom types.
- Two modes: Stopwatch (live timer) or Manual Entry (h:m:s input).
- Fields: duration, distance (unit varies by type), intensity (Easy/Moderate/Hard/All Out), min/max HR, notes.
- Can attach to a previous weight session from the same day, or log standalone.
- Navigated to directly from Dashboard or via "Log Cardio Now" in the finish modal.

### Splits System
- **SplitManager:** List, activate, edit, clone, export (JSON), import, delete splits.
- **SplitBuilder:** 4-step wizard. Step 1: split name + emoji. Step 2: add/edit workouts (each with name, emoji, exercise sections pulled from the exercise library or custom input). Step 3: set rotation order (drag workouts + rest days). Step 4: review & save.
- Splits are auto-created on first app load (built-in "BamBam's Blueprint").
- When a split is created/edited, workout IDs are generated via `generateId()`. These IDs are used in the rotation array and as the `type` field in logged sessions.

### Rest Timer (RestTimer.jsx)
- Floating circular button, visible only on logging screens (`/log/*`).
- **Draggable** (touch-based repositioning).
- Scales up 1.5× while running. Progress ring shows remaining time.
- Color states: idle (gray), running (blue), almost done ≤10s (amber), done (green ✓).
- Settings gear opens duration config (preset buttons: 60/90/120/180s, or custom).
- Auto-start option: begins countdown after logging a working set.
- Timestamp-based: survives backgrounding. Chime plays via Web Audio API on completion.

### Streaks
- **"Active day" definition (single source of truth):** a calendar day with at least one of a weight session, a cardio session, or an explicitly-logged rest day. Nothing else counts — rotation rest slots do NOT bridge gaps.
- `getWorkoutStreak(sessions, cardioSessions, restDaySessions)` returns the current unbroken run of active days ending at (or just before) today. Today is exempt from breaking the streak so it doesn't zero out before the user logs.
- `getBestStreak(sessions, cardioSessions, restDaySessions)` returns the longest consecutive-active-day run anywhere in history, using the exact same active-day definition. Guaranteed `best >= current`.
- **Both functions share the `buildActiveDaySet()` helper** in helpers.js so they can never drift.
- **Historical note:** Earlier versions bridged rotation rest slots (e.g. if the rotation said Sunday was a rest day, a missed Sunday didn't break the streak). That produced surprising jumps and a "best < current" inconsistency on Progress. Removed in Batch 11.
- `getAchievements(sessions, cardioSessions, restDaySessions)` — also dropped the `rotation` param.
- All call sites (Dashboard, History, BbLogger, Progress, ShareCard) call with the 3-arg signature.

### PR Tracking
- **Weight-anchored model.** `getExercisePRs(sessions, exerciseName)` returns `{ maxWeight, maxRepsAtMaxWeight }` — the heaviest weight ever lifted on that exercise, plus the best rep count achieved specifically at that max weight. Reps at sub-max weight are not tracked as PRs.
- **Single source of truth:** `isSetPR(sessions, exerciseName, weight, reps)` is the canonical PR check. A set is a PR if weight > maxWeight, OR weight === maxWeight AND reps > maxRepsAtMaxWeight. Used by per-set trophies (`PlateSetRow`, `SetRow`), the card-level `hasPR` trophy, and the save-time `isNewPR` flag baked onto each set. `isPR()` is a back-compat alias.
- **Scoped to workoutType.** All PR comparisons (live UI and save-time flag) use `scopedSessions` — sessions filtered to the current workout type — so "Pull-ups" in Back Day and Full Body track independently.
- **All-time PR chip.** Each exercise card header shows a small amber chip (`PR {maxWeight}×{maxRepsAtMaxWeight}`) as soon as the exercise has any logged history, giving immediate context for what threshold a new PR has to beat.
- New PRs are flagged with 🏆 in the session logger, share card, and history. Since Batch 14, historical `isNewPR` flags on all persisted sessions are recomputed under the weight-anchored-per-side rule by the v1→v2 persist migration, so they match what the live UI shows.
- **Known gap:** `getExercisePRs` signature changed from `{ maxWeight, maxReps }` to `{ maxWeight, maxRepsAtMaxWeight }`. No other call sites remain (verified via grep), but watch for stale destructuring if resurrecting old branches.

### Achievements
- Badge system based on milestones: session counts (1/10/25/50/100), PR counts (1/10/25), streaks (3/7 days), grade quality (5 A-grade sessions).
- Computed by `getAchievements()` in helpers.js.

### Settings (via HamburgerMenu)
- **Your Name:** Displayed in greeting and share card.
- **Theme:** Obsidian (dark) or Daylight (light). Controlled via `data-theme` attribute on `<html>`.
- **Accent Color:** 10 options. Each provides a full set of Tailwind class strings (bg, text, border, ring, hex, contrastText).
- **Auto-start rest timer:** Toggle.
- **First set defaults to:** Warmup or Working.
- **Rest timer chime:** Toggle.
- **Info section — "How Streaks Work":** Explains streak mechanics: logging any workout, cardio, or rest day keeps the streak alive. Rest day allotment = number of `'rest'` entries in the rotation. Example shown in the UI.
- **Manage Data:** Export/Import full backup as JSON (includes `restDaySessions`).

### Theming
- Two background themes via CSS custom properties: Obsidian (dark, default) and Daylight (light).
- 10 accent colors defined in `theme.js`. Each color provides Tailwind classes and hex values used for inline styles.
- CSS variables in `index.css` define: `--bg-base`, `--bg-card`, `--bg-item`, `--text-primary`, `--text-secondary`, `--text-dim`, `--text-muted`, `--text-faint`, `--border-base`, `--border-subtle`.
- Tailwind utility classes map to these: `bg-base`, `bg-card`, `bg-item`, `text-c-primary`, `text-c-secondary`, `text-c-dim`, `text-c-muted`, `text-c-faint`.

---

## Routes

| Path | Component | Description |
|---|---|---|
| `/` | Redirect | → `/welcome` (new users) or `/dashboard` (returning) |
| `/welcome` | Welcome | Onboarding wizard |
| `/dashboard` | Dashboard | Main screen |
| `/log` | Log | Workout picker |
| `/log/bb/:type` | BbLogger | Session logger (type = workout ID) |
| `/cardio` | CardioLogger | Cardio session logger |
| `/history` | History | Calendar heatmap + session list |
| `/progress` | Progress | Volume bar chart |
| `/guide` | Guide | Training tips |
| `/templates/new` | TemplateEditor | Legacy template creation |
| `/templates/:id` | TemplateEditor | Legacy template editing |
| `/splits` | SplitManager | Split list & management |
| `/splits/new/start` | ChooseStartingPoint | Template chooser (6 templates + Blank + Import) |
| `/splits/new` | SplitCanvas | Single-canvas editor (Batch 17g) |
| `/splits/edit/:id` | SplitCanvas | Single-canvas editor for existing splits (Batch 17g) |
| `/backfill` | Backfill | One-time muscle-group + equipment tagging for user-created exercises |
| `/exercises` | ExerciseLibraryManager | Full library management — filter, search, edit any entry |

---

## Data Persistence

- **Zustand persist middleware** with localStorage key `workout-tracker-v1`, current persist `version: 9`.
- Custom `merge` function in the persist config handles schema evolution: new fields get defaults, existing user settings are preserved via deep merge, existing users auto-skip onboarding.
- `migrate` hook handles versioned schema changes.
  - V1→V2 (Batch 14): backfills `rawWeight` on every set and recomputes `isNewPR` via `migrateSessionsToV2()` in helpers.js.
  - V2→V3 (Batch 15): seeds `exerciseLibrary` from built-in data if empty, assigns stable `exerciseId` to every LoggedExercise, canonicalizes display names against the library (Title Case variants win), flags unresolved names as `needsTagging: true`, and recomputes `isNewPR` keyed by exerciseId. Runs via `migrateSessionsToV3({sessions, library})`.
  - V3→V4 (Batch 17a): additive — ensures `splitDraft` slot exists (defaults to `null`). Pre-v4 users simply don't have a draft, which is the correct initial state.
- **Active session** (`activeSession`) persists across page reloads and app backgrounding so in-progress workouts are never lost.
- **Rest timer** uses absolute timestamps (`restEndTimestamp`) rather than countdown values, so it stays accurate across backgrounding.

---

## Development Workflow (Claude Code Desktop)

**Git is fully writable from the sandbox.** Verified April 15, 2026. An earlier rule prohibited sandbox git operations, but that was a stale scar from a transient issue — git works cleanly. Claude runs git directly.

**Standard workflow for a non-trivial change:**
1. `git worktree add -b claude/<descriptive-name> .claude/worktrees/<name>` — isolated branch + checkout
2. `cd .claude/worktrees/<name> && npm install` — each worktree has its own gitignored `node_modules`
3. Edit files in the worktree
4. Validate: `cd .claude/worktrees/<name> && npx vite build --outDir /tmp/test-build`
5. For logic-heavy changes, run a data sanity check (e.g. `streak-debug.mjs` against `debug-backup.json`)
6. Add a worktree-specific entry to `.claude/launch.json` (see Claude Preview workflow below)
7. `preview_start` against the worktree entry — walk affected surfaces with `preview_snapshot` / `preview_console_logs` / `preview_screenshot`
8. Commit in the worktree (heredoc commit message, Co-Authored-By Claude line)
9. **For user-visible changes: report findings + screenshots to the user and WAIT for explicit "merge it" / "looks good" before merging.** No silent merges of UI work.
10. For pure data-layer / engine changes that already pass sanity + build: merge to main directly.
11. `git checkout main && git merge --ff-only claude/<name> && git push origin main`
12. Delete the worktree: `git worktree remove .claude/worktrees/<name>`, prune the launch.json entry.

**Small fixes (typo, one-line patch):** straight to main is fine — no worktree.

**Claude Preview is the only review surface.** Vercel previews are auth-walled on the team plan and the user's browser can't load them — confirmed April 25, 2026. Always verify user-visible changes via `preview_start`, never via a shared Vercel URL.

**Worktree preview pattern.** Each active worktree gets its own `.claude/launch.json` entry named after the worktree directory:
```json
{
  "name": "<worktree-name>",
  "runtimeExecutable": "bash",
  "runtimeArgs": ["-c", "cd .claude/worktrees/<worktree-name> && npm run dev"],
  "port": <unique-port>,
  "autoPort": true
}
```
Use a unique port per worktree (5173 main, 5174+ for worktrees) so multiple servers can run side by side. `launch.json` is gitignored — local config, never committed.

**PowerShell note:** If the user needs to run a git command for any reason, use PowerShell syntax (`Remove-Item`, backslashes, etc.) since they're on Windows.

**Vercel project:** Team `team_Ol4ZacaHh0oiEz562VTQLwRg` (slug: `bbblueprint`), Project ID `prj_PFuFC2BuTn6LFhR03fODL5Poc0eo` (name: `bambam`). Auto-deploys production from `main`. Preview URLs exist but are auth-walled — not a review surface for this user.

**Build validation:** `npx vite build` will fail with EPERM when writing to the mounted `dist/` folder. Always use `--outDir /tmp/test-build`.

---

## Known Architectural Notes

- The app uses `HashRouter` (not `BrowserRouter`) for Vercel SPA deployment compatibility.
- Max width is constrained to `max-w-lg` (32rem / 512px) and centered — mobile-first design.
- Bottom nav hides during logging sessions AND during the split builder / canvas routes (`/splits/new`, `/splits/new/start`, `/splits/edit/:id`). Hamburger menu mirrors the same fullscreen-flow hide predicate. The `/splits` list view still shows the nav.
- `customTemplates` is a legacy feature predating the splits system. Templates use `tpl_` prefixed IDs.
- The exercise library (`exerciseLibrary.js`) has 140+ exercises organized by muscle group and is used by the SplitBuilder's exercise picker. The session logger's "Add Exercise" panel has its own smaller hardcoded suggestion list of 15 common exercises.

---

## Recent Changes (April 1, 2026)

### Batch 1
1. **Dashboard spacing:** Increased top padding on greeting from 32px to 56px + safe area offset.
2. **Stat cards compact:** Reduced padding, font sizes, divider heights for tighter layout while keeping full width.
3. **Custom exercise persistence:** Logger now merges exercises from last session of the same type that aren't in the template, placed in an "Added" group. Ensures custom exercises always reappear.
4. **Reorder button removed:** Stripped the "Reorder" button and reorder mode from session logger exercise groups.
5. **History name resolution:** Added `resolveWorkoutName()` / `resolveWorkoutEmoji()` that look up names from all splits in the store, fixing garbled IDs for user-created split workouts.

### Batch 2
6. **Editable grade in History:** Grade badge in session detail is now tappable — shows inline picker to set/change grade post-save. Also shows cardio ✓ badge when cardio is attached (even if not inline).
7. **Share card cardio fix:** `buildShareData` in History now accepts and uses `attachedCardio` to populate cardio details when viewing from History.
8. **Unilateral (Uni) toggle:** Purple "Uni" button next to Plates on each exercise. Doubles volume at save time. Stored as `unilateral: true` on the exercise, `rawWeight` preserved.
9. **Plates 1× / 2× multiplier:** Global toggle per exercise in plate picker area. Replaces auto-doubling. All set weights recalculate when toggled. Default: 2× (both sides).
10. **Plate data in session history:** Set data now includes `plates` and `barWeight` fields. Ghost rows display plate breakdown format when available.
11. **Post-session comparison screen:** Auto-shown after saving if previous session exists. Per-exercise volume % diff with green ↑ / red ↓ arrows. Overall volume change at top. "Continue" button proceeds to share card.

### Batch 3 (April 2, 2026) — UX simplification based on user feedback
12. **Removed pre-populated weights/reps:** Set inputs now start blank with generic placeholders ("lbs" / "reps"). No more previous-session weight/rep hints in placeholders, no plate mode weight pre-fill from last session, no drop set 75% auto-suggestions. "Last Time" ghost rows remain fully intact for on-demand reference.
13. **Keyboard-driven set input flow:** Weight input uses `enterKeyHint="next"` (iOS keyboard shows "Next"), reps input uses `enterKeyHint="done"`. Pressing Next jumps from weight → reps. Pressing Done auto-adds a new set row and focuses its weight field. Enables a seamless number → next → number → done → repeat loop.
14. **Contextual green ✓ button on set rows:** The × (delete) button morphs into a green ✓ when both weight and reps are filled (or reps + plates in plate mode). Tapping ✓ advances to the next set, same as the keyboard Done action. Empty/incomplete rows keep the × for deletion.

### Batch 4 (April 3–5, 2026) — Custom numpad and focus mode

15. **Custom numpad (CustomNumpad.jsx):** Replaces the iOS system keyboard for all weight/reps inputs. Uses `inputMode="none"` to suppress system keyboard, `onPointerDown` with `preventDefault` on keys to keep inputs focused. Phone layout (1-2-3 on top). NumpadContext provided at BbLogger page level for SetRow/PlateSetRow to register focus.
16. **Numpad action row:** Shows field label (WEIGHT/REPS), Next → (secondary outlined), Done ✓ (accent primary). Next on weight → focuses reps. Next on reps → submits set and adds new row. Done → marks exercise as completed (calls `markDone`) and closes numpad.
17. **Hide button:** "Hide ↓" button in the bottom-left key grid cell (reps fields, where decimal isn't needed). On weight fields, a full-width Hide bar below the grid. Styled with same outline as Next. Just closes the numpad without marking done.
18. **Focus mode:** When numpad is open, all non-active exercises, section labels, Add Exercise button, Session Notes, and the sticky footer are hidden. Only the exercise owning the active input field remains visible. Determined by `activeFieldKey.includes(exercise.name)`.
19. **Focus mode exit:** "Tap to show all exercises" zone appears below the active exercise when numpad is open. Tapping it blurs input and closes numpad, restoring the full exercise list.
20. **Anti-jostling:** `focus({ preventScroll: true })` when auto-focusing newly added set rows prevents browser viewport jumping.
21. **Compact numpad:** Keys 44px tall (down from 56px), gap 6px (down from 8px), reclaiming ~60px vertical space.
22. **Stable ref callbacks:** All numpad config callbacks (`onChange`, `onNext`, `onDone`) use ref-backed patterns to avoid stale closures when set state changes while an input is focused.
23. **Next button stale-state fix:** The numpad now passes `currentValueRef.current` (the live numpad buffer) as an argument to `onNext(value)`. This fixes the critical bug where pressing Next on the reps field did nothing because `setRef.current.reps` was stale (React hadn't re-rendered yet after the last `onChange`). The `handleNextSet` callbacks in both SetRow and PlateSetRow now use this passed value instead of reading from the stale ref.

### Batch 5 (April 5, 2026) — Session timer UX overhaul

24. **"Start Session" overlay:** When entering BbLogger, a full-screen overlay appears with the workout emoji, name, and a "Start Session" button. Timer stays at 0:00 until tapped. Includes "Go back" link. Overlay does not appear when resuming a saved session (already started).
25. **Timer repositioned & bolder:** The elapsed timer badge in the clipboard header is now `text-base font-extrabold` (up from `text-sm font-bold`) with tighter tracking. Positioned in the top-right of the header bar alongside a new pause/play button.
26. **Pause/resume button:** A play/pause toggle button appears next to the timer once the session has started. Pausing switches the timer badge to a `bg-white/30` style to visually indicate paused state.
27. **Pause persists across navigation:** Tapping the back arrow auto-pauses the session. All timer state (`sessionStarted`, `startTimestamp`, `isPaused`, `totalPausedMs`, `pauseStartedAt`) is persisted in `activeSession` via Zustand, so navigating away and returning preserves exact timer state. Timer stays paused until manually resumed.
28. **Accurate elapsed time with pause tracking:** Elapsed seconds now calculated as `(now - startTimestamp - totalPausedMs) / 1000`, properly subtracting all accumulated pause durations. Duration saved to the session remains accurate.
29. **Dashboard "Resume Workout" CTA:** When an active session exists (started but not finished), the Dashboard CTA card changes to show the in-progress workout name/emoji, a pulsing "In Progress" or "Paused" indicator, a count of exercises logged so far, and a "Resume Workout →" button that navigates back to the logger. This replaces the normal "Start Session" CTA so users always know their progress is preserved.

### Batch 6 (April 6, 2026) — Share Card redesign

30. **Share Card full rewrite (`ShareCard.jsx`):** Replaced old scrollable card with a fixed-viewport trading card design (max 380px, no scrolling). Uses all-inline styles (no Tailwind) matching the approved mockup.
31. **5-tier rarity system:** Tier determined by `getWorkoutStreak()` at render time — Common (0–5, user accent color), Rare (6–14, silver), Epic (15–19, gold), Legendary (20–49, animated shimmer orange/gold gradient border), Mythic (50+, animated shimmer purple/cyan border + 12 sparkle particles). CSS keyframes injected via `<style>` tag.
32. **JPEG export + Web Share API:** "Share ↗" button captures the card via `html2canvas` (scale:2, retina quality), exports as JPEG via `navigator.share({ files })` on iOS; falls back to download on unsupported browsers. Share/Done buttons are outside `cardRef` so they don't appear in the exported image.
33. **New stat bar:** VOL / SETS / STREAK 🔥 / GRADE — replacing old PRs slot with streak. Per-exercise PR badges still shown in exercise list.
34. **`streak` added to share data:** Both `BbLogger.jsx` (uses `getWorkoutStreak(sessions, activeSplit?.rotation)`) and `History.jsx` (`buildShareData` now accepts `sessions` + `activeSplitId`) pass streak into the data object.
35. **Cleanup:** Deleted `ShareCardMockup1.jsx`, `ShareCardMockup2.jsx`, `ShareCardTiers.jsx` and removed their routes from `App.jsx`.
36. **New dependency:** `html2canvas` added to `package.json`.

### Batch 7 (April 6, 2026) — Rest day calendar + streak bug fix

37. **Fixed `getFullRotationItem` timezone bug (`Dashboard.jsx`):** The old implementation computed `daysSinceAnchor` as `Math.round((today - new Date('YYYY-MM-DD')) / 86400000)` — `today` included current local time while the anchor date was UTC midnight, so after ~12pm the round went to 1 instead of 0, shifting the entire rotation index forward by one slot. Replaced with a delegate to `getRotationItemOnDate(toDateStr(d), sessions, rotation)` which uses UTC-midnight strings on both sides and is always timezone-safe.
38. **Rest days now show in the weekly and monthly calendars for past dates (`Dashboard.jsx`):** `getDayInfo` previously only checked the rotation for future days (`ahead > 0`). Past days with no session fell through to `{ type: 'empty' }`. Now past rest days are detected via `getRotationItemOnDate` and returned as `{ type: 'past-rest' }`, rendering a dimmed "R" in both the weekly strip and monthly grid (matching the existing future-rest "R" display).
39. **Monthly view `isFutureRest` fixed:** The monthly calendar was rendering `isFutureRest` via a condition that required `info.emoji` (which rest days don't have), so rest days were invisible. Now explicitly renders an "R" badge for `isFutureRest` and `isPastRest`.

### Batch 8 (April 6, 2026) — Beta UX polish

40. **Force portrait orientation:** `main.jsx` calls `screen.orientation.lock('portrait')` on load (works for installed PWAs). CSS fallback in `index.css` covers non-PWA browsers with a black overlay via `@media (orientation: landscape) { body::before }`.
41. **Completed exercises sorted by completion time (`BbLogger.jsx`):** `completedExes` is now sorted ascending by `completedAt` timestamp before being added to `renderGroups`, so exercises appear in the order they were marked done.
42. **Compact sticky header (`BbLogger.jsx`):** Removed the ClipGraphic decorative SVG from the header. Reduced the back/timer row from `pb-2` to `pb-1.5`, shrunk control buttons (w-9→w-8, w-8→w-7), reduced timer badge padding (`px-3 py-1.5`→`px-2.5 py-1`) and font size (`text-base`→`text-sm`), reduced workout title from `text-2xl pb-4` to `text-lg pb-2`. Net ~35% less vertical header height.
43. **Auto-collapse completed exercises in focus mode (`BbLogger.jsx`):** Removed the `&& !exercise.done` guard from the `focusCollapsed` condition. Completed exercises now collapse along with pending ones when the numpad is open and another exercise owns the active field.
44. **Fix numpad hide scroll jump (`CustomNumpad.jsx`):** `handleHide` now captures `window.scrollY` before blurring the active input, then calls `window.scrollTo({ top: y, behavior: 'instant' })` to restore the position immediately, preventing the page from scrolling up when the numpad slides away.

### Batch 9 (April 12, 2026) — Streak overhaul + rest day logging

45. **Streak forward-check fix (`helpers.js`):** `getWorkoutStreak()` now counts backwards from *yesterday* (not today), so the streak doesn't drop to 0 before the user logs today's activity.
46. **Cardio counts toward streak (`helpers.js`):** `getWorkoutStreak()` accepts a `cardioSessions` param. Cardio session dates are merged into the activity day set, so standalone cardio keeps the streak alive. All call sites (Dashboard, History, BbLogger, Progress, ShareCard) updated to pass `cardioSessions`.
47. **Rest day logging (`useStore.js`, `Dashboard.jsx`):** New `restDaySessions` array in the store with `addRestDaySession` / `deleteRestDaySession` actions. "Log Rest Day" button added to Dashboard (next to Log Cardio). Included in full JSON export/import.
48. **Rest days count toward streak (`helpers.js`):** `getWorkoutStreak()` also accepts `restDaySessions`. Logging a rest day on any given calendar day keeps the streak alive. All call sites pass `restDaySessions`.
49. **Flexible rest day allotment (`helpers.js`):** Rest days are no longer position-based. The streak algorithm uses a pool equal to the count of `'rest'` entries in the rotation. Any empty day within a rotation cycle consumes one from the pool instead of breaking the streak.
50. **Streak always visible (`Dashboard.jsx`):** `StreakBadge` renders unconditionally. At 0 it shows "0🔥" instead of being hidden.
51. **"MISSED YESTERDAY" fix (`Dashboard.jsx`):** `yesterdayLogged` now checks all three arrays (sessions, cardioSessions, restDaySessions) so cardio-only or rest-day-only days suppress the soreness prompt correctly.
52. **Removed "X/7 days" display (`Dashboard.jsx`):** The `weekCompletedCount/7` text was removed from the active split label. Now just shows the split name and emoji.
53. **Calendar strip Sun–Sat (`Dashboard.jsx`):** Weekly strip changed from Mon–Sun to Sun–Sat to align with weekly volume stats.
54. **Hero card spacing (`Dashboard.jsx`):** Increased gap between Preview and Start Session buttons; reduced bottom padding on the CTA card.
55. **Unilateral ghost row fix (`BbLogger.jsx`):** `PrevSetRow` now displays `rawWeight` instead of the doubled `weight` for unilateral exercises, so "Last Time" shows the actual per-side input the user entered.
56. **Uni toggle pre-selected (`BbLogger.jsx`):** The Uni toggle initializes to `true` if the last session had `unilateral: true` for that exercise.
57. **Plates toggle pre-selected (`BbLogger.jsx`):** `plateLoaded` state is initialized from last session's set data — if the last session had plate data, the plate picker opens pre-loaded.
58. **getDayInfo handles logged rest days (`Dashboard.jsx`):** Calendar day pills distinguish explicitly logged rest days (brighter dot / different style) from rotation rest days.
59. **`getAchievements` updated (`helpers.js`):** Now passes `cardioSessions` (and `restDaySessions`) to `getWorkoutStreak` for accurate streak-based badge evaluation.
60. **"How Streaks Work" info section (`HamburgerMenu.jsx`):** Added an expandable explainer in the Info menu covering streak rules, what counts as activity, and rest day allotment logic.
61. **Drag-and-drop exercises between sections (`SplitBuilder.jsx`):** Long-press drag to reorder and move exercises between workout sections within the SplitBuilder. *(Note: implemented in worktree `claude/brave-chandrasekhar`, not yet merged to main.)*
62. **Persistent added exercises (`BbLogger.jsx`):** Logger now scans ALL past sessions of a workout type (not just the most recent) to build the exercise list. Any exercise ever added to a workout is permanently remembered. Uses `lastExDataByName` map keyed by exercise name from newest-first session scan.
63. **Last Time always most recent (`BbLogger.jsx`):** Ghost row (`PrevSetRow`) data pulls from the most recent session that actually included that specific exercise, not just the immediately preceding session. Fixes stale/missing Last Time data when exercises are skipped.
64. **Plates/Uni init from all sessions (`BbLogger.jsx`):** Template exercise plates/uni pre-selection also uses `lastExDataByName` instead of only last session, ensuring correct toggle state even when exercises were skipped recently.

### Batch 10 (April 15, 2026) — PR refactor + all-time PR chip

65. **PR model rewritten to weight-anchored (`helpers.js`):** `getExercisePRs` now returns `{ maxWeight, maxRepsAtMaxWeight }` instead of the old `{ maxWeight, maxReps }`. Only the heaviest weight ever lifted is tracked, along with the best rep count achieved specifically at that top weight. Reps at lighter weights no longer qualify as PRs.
66. **`isSetPR()` single source of truth (`helpers.js`):** New canonical helper. A set is a PR iff `weight > maxWeight` (new heaviest) or `weight === maxWeight && reps > maxRepsAtMaxWeight` (more reps at top weight). Zero-weight and zero-rep sets never qualify. First-ever logged set for an exercise counts as a PR.
67. **`isPR()` kept as back-compat alias (`helpers.js`):** Delegates to `isSetPR`. The old `isPR` semantics (reps-PR only if weight >= maxWeight) is gone; the new stricter rule applies everywhere.
68. **All inline PR checks consolidated (`BbLogger.jsx`):** `PlateSetRow` per-set trophy, `SetRow` per-set trophy, card-level `hasPR` trophy, and save-time `isNewPR` flag all now call `isSetPR(scopedSessions, ...)`. Eliminates the four-way drift that previously produced inconsistent trophies.
69. **Save-time PR scoping fix (`BbLogger.jsx`):** `isNewPR` baked onto each saved set now uses `scopedSessions` (filtered to the current workout type) instead of all sessions. Aligns the saved flag with what the UI shows during logging and makes an exercise reused across splits (e.g., Pull-ups in Back Day vs. Full Body) track independently end-to-end.
70. **All-time PR chip on every exercise card (`BbLogger.jsx`):** Small amber chip `PR {maxWeight}×{maxRepsAtMaxWeight}` renders next to the exercise name in the card header whenever any history exists. Visible in both collapsed and expanded states. Makes the PR threshold explicit so users know what they're chasing — ghost "Last Time" rows were misleading users into thinking they had a PR when their actual all-time max was higher.
71. **Historical flags unchanged:** Sessions saved before this batch retain their original `isNewPR` flags (computed under the old looser rule). Only newly saved sessions reflect the weight-anchored model. A one-time migration pass over historical sessions is available if desired but not yet run. *(Batch 14 ran it.)*

### Batch 11 (April 15, 2026) — Streak unification + sandbox git rule removed

72. **Unified streak definition (`helpers.js`):** `getWorkoutStreak` and new `getBestStreak` now share a single `buildActiveDaySet()` helper and a single "active day" definition: a calendar day with a weight session, a cardio session, or an explicitly-logged rest day. Rotation rest slots NO LONGER bridge empty days. This kills the prior bug where current-streak (rotation-aware) could be larger than best-streak (rotation-unaware) — classic symptom: Dashboard shows 14, Progress shows "Best: 8".
73. **`getWorkoutStreak` signature change:** dropped `rotation` param. Now `(sessions, cardioSessions, restDaySessions)`. All 4 call sites updated (Dashboard, History, BbLogger, Progress).
74. **`getAchievements` signature change:** dropped `rotation` param. Now `(sessions, cardioSessions, restDaySessions)`.
75. **New `getBestStreak` (`helpers.js`):** scans the full activity set and returns the longest consecutive-active-day run. Replaces the inline calc that previously lived in `Progress.jsx`'s `ConsistencyHeatmap`, which used BB-sessions-only (no cardio, no rest-day logs).
76. **`ConsistencyHeatmap` now takes `bestStreak` as a prop** from the parent `Progress` component, which computes it via `useMemo`. Inline calc removed.
77. **Sandbox-git prohibition removed (`CLAUDE.md`):** Verified that git is fully writable from the sandbox. Updated workflow docs to have Claude run git directly (worktree → edit → build → preview → commit → merge → push), and dropped the prior "give user PowerShell commands" dance. User still owns the final merge gate for anything risky via Vercel preview URLs. No `--force` pushes, no hook skipping.

### Batch 12 (April 16, 2026) — Critical hotfix: Finish Session modal buttons dead

78. **`scopedSessions` scope bug (`BbLogger.jsx`):** Batch 10's PR refactor introduced `isSetPR(scopedSessions, ex.name, w, r)` inside `buildExerciseData()` at line 1634, but `scopedSessions` was only defined inside the `ExerciseItem` sub-component (line 466) — not in the parent `BbLogger` scope. Every Finish-modal button (Attach / Keep Separate / Log Another / Save / Log Now) threw `ReferenceError: scopedSessions is not defined` when clicked, silently aborting the save. Users saw buttons that "don't respond." Fixed by defining `scopedSessions` at the BbLogger scope alongside `lastSession` (line 1576), mirroring the ExerciseItem filter. Verified by reproducing the error live (clicking Save returned `save-err: scopedSessions is not defined`), applying the fix, and re-running both Scenario A (cardio today → Attach) and Scenario B (no cardio → Save): both now persist correctly with `isNewPR` flag set.

### Batch 13 (April 16, 2026) — Per-exercise REC (coach's prescription) pill + PR chip relocation

79. **REC field on split exercises (`BbLogger.jsx`, data model):** Split workout exercises can now be either a bare string (legacy) OR an object `{ name, rec }` where `rec` is a free-text prescription string like `"3x20 (warmup)"` or `"4x10-10-10 drop"`. Length-capped at 20 chars to keep the title row from overflowing. Loaded via the existing `typeof e === 'string' ? e : e.name` unwrap pattern that already existed in the save-time auto-persist code.
80. **Blue REC pill on every exercise card header (`BbLogger.jsx`):** When an exercise is expanded, a small pill appears next to the title: 📋 with the rec text if set (filled blue pill, `bg-blue-500/15 border-blue-500/40 text-blue-300`), or dashed-outline `📋 REC` placeholder when empty (`bg-item text-c-muted border-dashed`). Hidden when the card is collapsed. Tapping the pill swaps it for a 20-char-capped input field; Enter/blur commits, Esc cancels. `stopPropagation` on the pill/input prevents the card collapse toggle from firing.
81. **Header wrapper changed from `<button>` to `<div role="button">` (`BbLogger.jsx`):** The collapse-toggle header is now a div so nested interactive elements (the REC pill button + its inline `<input>`) are valid HTML. `cursor-pointer select-none` + `role="button"` + `tabIndex={0}` preserve click-to-expand semantics.
82. **PR chip moved + hidden when collapsed (`BbLogger.jsx`):** The amber `PR {maxWeight}×{maxRepsAtMaxWeight}` chip no longer sits in the title row. It's now grouped with `Last Time` in the toolbar row (`[Plates] [Uni] …[Last Time] [PR chip]`, right-aligned via a `ml-auto` group wrapper) and only renders when the card is expanded. Distinct blue (REC) vs. amber (PR) keeps the two concepts visually separate.
83. **Rec auto-persists to split on session save (`BbLogger.jsx`):** The existing auto-persist block (which added new exercises to the split template) now also detects `rec` changes on existing template exercises. If any `rec` differs from what's in the split — or if new exercises exist with a `rec` — the block rebuilds the `sections` array, promoting exercises to `{name, rec}` when rec is set and keeping them as strings when it isn't. Result: once the user fills in a REC in the logger and saves, it pre-populates next time they open that workout.
84. **REC loads from split on template init (`BbLogger.jsx`):** `templateExercises` now unwraps `{name, rec}` from the split's section.exercises entries and seeds `exercise.rec` on the logger's exercise state. Also survives active-session reload since `exercise.rec` persists through the existing `saveActiveSession` flow.
85. **SplitBuilder rec data-loss fix + inline rec entry (`SplitBuilder.jsx`):** Both places where `SplitBuilder` loaded existing split exercises (outer `useState` at line ~964 and `WorkoutBuilder` sub-component at line ~230) previously called `typeof ex === 'string' ? ex : (ex?.name || '')` — actively flattening `{name, rec}` objects back to strings. After Batch 13 landed, this silently wiped any rec set via the logger whenever the user edited the split. Fixed by preserving objects as-is (stripping only malformed entries). Also added a `RecInline` component that renders next to each exercise name in the workout builder so coaches can prescribe recs up front; same pill style as the logger, same 20-char cap, same stop-propagation behavior so editing doesn't interfere with drag-to-reorder.
86. **Label + style tweaks (`BbLogger.jsx`, `SplitBuilder.jsx`):** "REC" → "Rec" (title case). Empty-state pill dropped the dashed white outline and softened to `bg-item text-c-faint` (no border) to de-emphasize when no recommendation is set. Filled pill remains blue; collapsed state still hides the pill entirely.

### Batch 14 (April 17, 2026) — Canonical weight field + v2 migration (AI coaching prereq, step 1)

Step 1 of the AI Coaching Recommender v1 plan (see `coaching-recommender-spec-v3.pdf` §3.1 and `.claude/plans/c-users-user-claude-code-workout-tracke-misty-hearth.md`). Fixes three live production bugs that silently corrupted the data the recommender depends on.

87. **`perSideLoad(set)` helper (`helpers.js`):** New canonical accessor — `set.rawWeight ?? set.weight ?? 0`. Use this wherever a set's load is read for comparison or display. For non-unilateral sets `rawWeight` is undefined and `weight` IS the per-side load, so the fallback is correct. For unilateral sets, `rawWeight` holds the per-side input; `weight` holds the doubled volume value.
88. **Phantom PR fix (`helpers.js:93`):** `getExercisePRs` reads `perSideLoad(set)` instead of `set.weight`. Previously post-2026-04-02 unilateral sets trivially beat pre-cutover records because the doubled `weight` field was compared against historical per-side values. Live PR trophies, all-time PR chips, and save-time `isNewPR` all now track per-side strength progression.
89. **"Last:" display fix (`BbLogger.jsx:654-655`):** Collapsed-card inline "Last: 180×9" hint reads `perSideLoad(lastTopSet)` instead of `lastTopSet.weight`. Aligns with `PrevSetRow` (ghost rows) which was already correct. Same fix applied to both the plate-mode total (`= ${perSideLoad(lastTopSet)}`) and the non-plate-mode display.
90. **Save-time PR check fix (`BbLogger.jsx:1695`):** `buildExerciseData()` now passes `rawW` (per-side) to `isSetPR`, not `w` (doubled). Aligns saved flags with the weight-anchored-per-side rule so newly saved unilateral sets don't trivially beat their own scoped history.
91. **V1→V2 persist migration (`useStore.js:317`, `helpers.js`):** Persist version bumped `1 → 2`. New `migrate` hook runs `migrateSessionsToV2(persistedState.sessions)` — backfills `rawWeight` on every set (defaults to `weight`) and recomputes every `isNewPR` chronologically per exercise name using `perSideLoad`. O(sets), idempotent. Validated against `debug-backup.json` (24 sessions, 425 sets preserved, 188 rawWeight backfills, phantom PRs 232 → 135, unilateral PRs 48 → 18). Updates the Batch 10 "known gap" — historical flags now match the live UI rule.
92. **Minor: `.js` extension on internal import (`helpers.js:1`):** `'../data/exercises'` → `'../data/exercises.js'`. Vite supports both; bare Node 24 requires the explicit extension. Enables `node migration-sanity.mjs` without a bundler step.
93. **`migration-sanity.mjs` checked in (repo root):** Node ESM script — loads `debug-backup.json`, runs `migrateSessionsToV2`, reports before/after PR counts, rawWeight coverage, per-exercise max weight spot-checks, and name-collision candidates (preview of step 2's dedup work). Mirrors the `streak-debug.mjs` pattern already in the repo.

### Batch 15 (April 17, 2026) — Exercise IDs + canonical library + backfill UI + picker dedup (AI coaching prereq, step 2)

Step 2 of the AI Coaching Recommender plan. Eliminates name-based identity for exercises and replaces it with a stable, deduplicated library that every session references by `exerciseId`. Unblocks the recommender (step 3) by giving it a reliable per-exercise history to fit against. Shipped in four substeps (15a–15d) — each reviewable on its own.

#### 15a — Library foundation

94. **`exerciseLibrary` slice seeded from built-in data** (`useStore.js`, `data/exerciseLibrary.js`). `buildBuiltInLibrary()` transforms the 90 raw `{name, muscleGroup, equipment}` entries in `data/exerciseLibrary.js` into the canonical Exercise entity with slug-derived IDs (`ex_{slug(name)}`), `primaryMuscles[]`, `progressionClass` (compound/isolation/bodyweight), and the other recommender-facing fields. `builtInExerciseIdForName(name)` is exported for the migration and dedup paths. `EQUIPMENT_TYPES` array exported from `exerciseLibrary.js` (Barbell, Dumbbell, Machine, Cable, Bodyweight, Kettlebell, Other).
95. **`initLibrary()` store action** — idempotent seed called from `App.jsx` alongside `initSplits()` on mount. Fresh installs get the seeded library immediately; returning users with a populated library skip the seed. Paired with an adjusted merge hook so persisted empty libraries don't stick on upgrade.
96. **Library CRUD store actions.** `addExerciseToLibrary(exercise)` enforces the §3.2.1 required-field constraint at runtime (rejects empty `primaryMuscles`, missing `equipment`, empty `name`). `updateExerciseInLibrary(id, patch)`, `deleteExerciseFromLibrary(id)`, and `mergeExercises(keepId, mergeIds)` complete the surface. `mergeExercises` rewrites every session's `exerciseId` from any merge-id → keep-id before removing the merged entries, so history is never lost.

#### 15b — V2→V3 migration

97. **`migrateSessionsToV3({sessions, library})` helper** (`helpers.js`). Collects every distinct session-exercise name, pre-sorts by descending capital-letter count (so Title Case variants win the canonical slot), then resolves each against the library by normalized key (`normalizeExerciseName` — case-insensitive, whitespace-collapsed). Matches become aliases on the existing library entry; unresolved names become new library entries with `primaryMuscles: []`, `equipment: 'Other'`, `needsTagging: true`. Every LoggedExercise is rewritten with canonical `name` + stable `exerciseId`. `isNewPR` is then recomputed chronologically keyed by exerciseId so post-canonicalization duplicates share PR progression.
98. **Persist version bumped `2 → 3`** with the migrate hook wired up. V2→V3 seeds the library from built-in if the persisted slice is empty (supports the direct v1→v3 upgrade path) then runs `migrateSessionsToV3`. Idempotent — re-running is a no-op.
99. **Sanity-checked against `debug-backup.json`** (`migration-v3-sanity.mjs` checked in): 90 built-ins + 19 user-created = 109 library entries (all 19 flagged `needsTagging`), 136 LoggedExercises get exerciseId (100% coverage), "Seated cable row" correctly canonicalizes to "Seated Cable Row" with the lowercase form preserved as an alias. PR flags 135 → 134 (one phantom cleared by unified history).

#### 15c — Backfill UI

100. **`/backfill` route + `Backfill.jsx` page.** Lists every library entry with `needsTagging: true` as cards. Each card shows canonical name + most-recent logged set for context (`Last: 190 × 9 · 10 Apr 2026`) plus pill rows for primary muscles (multi-select, min 1) and equipment (single-select from `EQUIPMENT_TYPES`). Completion rule: entry drops off the list once `primaryMuscles.length > 0` AND `equipment` is set to a non-`'Other'` value. Success state at 0 remaining: "All exercises tagged!" + "Back to Dashboard" CTA.
101. **`tagExercise(id, patch)` store action** (`useStore.js`). Atomic merge-and-recompute — needsTagging is evaluated against the merged state inside a single `set(state => ...)`, not against a stale render snapshot. This was a real bug — back-to-back pill taps in Backfill.jsx were losing the completion check because each click's handler read `exerciseLibrary` from the prior render's closure and overwrote the other's update.
102. **Dashboard banner.** Small blue pill above the calendar, only rendered while `pendingCount > 0`, links to `/backfill`. Message mentions "smarter recommendations" to preview the v1 payoff. Non-intrusive — no auto-redirect from Dashboard mount, so the user can defer indefinitely. Disappears when tagging is complete.

#### 15d — Picker dedup + creation modal

103. **Fuzzy match helpers** (`helpers.js`). `similarExerciseScore(a, b)` returns 0.0–1.0 by maxing three signals: exact-after-normalization, token-sort equality (catches "DB Lateral Raises" vs "Lateral DB Raises"), and trigram Jaccard. `findSimilarExercises(query, library, {suggestThreshold, max})` scans each library entry's canonical name + aliases, returns the top-N above threshold sorted by score desc. Spec §3.3 thresholds: `0.85` for auto-dedup, `0.7` for the suggest-list. `normalizeExerciseName(name)` exposed for callers that want to compare without scoring.
104. **`CreateExerciseModal.jsx`** — shared bottom-sheet-on-mobile / centered-on-desktop form. Collects `name`, `primaryMuscles` (multi-select, ≥1), `equipment` (single-select), and `defaultUnilateral` toggle. Save button is disabled until required fields are set, so the §3.2.1 constraint is enforced at the UI layer in addition to the store action's runtime check.
105. **BbLogger `AddExercisePanel` rewired.** Suggestions come from the store library via fuzzy match when the user types; a 12-name starter list (pulled from built-ins) shows on empty query. Tapping "+ Add [typed]" auto-uses any library match scoring ≥0.85 (so "pec dec" silently resolves to `ex_pec_dec`); otherwise opens `CreateExerciseModal` pre-filled with the typed name. `addExercise(name, exerciseId)` accepts an optional id so picked-from-library exercises carry the link through to save.
106. **`buildExerciseData` saves canonical name + exerciseId** (`BbLogger.jsx`). On session save, resolves `exerciseId` — prefers the row's linked id from the picker selection, falls back to a library lookup by canonical/alias name. Canonicalizes `name` to the library's value too. Going forward every saved session has `{name: 'Canonical Name', exerciseId: 'ex_...'}` on every LoggedExercise.
107. **SplitBuilder `ExercisePicker` wired to the same dedup path.** "Add your own" now fuzzy-matches against the store library before creating; high-similarity matches reuse the existing entry, low-similarity opens `CreateExerciseModal`. The full picker list now reads from the store library when populated (so user-created entries show up alongside built-ins), falling back to the raw data import for fresh installs before `initLibrary()` fires.

### Batch 16a (April 18, 2026) — Recommendation engine (AI coaching prereq, step 3, substep a)

Step 3 of the AI Coaching Recommender plan (see `coaching-recommender-plan.md` Part 3). Pure algorithm code — no UI wiring yet. Sanity-validated via `recommender-sanity.mjs` against `debug-backup.json`.

108. **`e1RM(weight, reps)` + `percent1RM(targetReps)` (`helpers.js`).** Layer 1 (Epley: `w × (1 + r/30)`) and Layer 2 table lookup per spec §2.2. `percent1RM` linearly interpolates between the 7 anchors (3→93%, 5→86%, 6→83%, 8→78%, 10→73%, 12→69%, 15→63%) and clamps at the endpoints so off-table inputs still return a sane number.
109. **`getExerciseHistory(sessions, exerciseId, exerciseName?)` (`helpers.js`).** Walks `bb`-mode sessions, prefers `exerciseId` match (with name fallback for pre-v3 safety), returns the per-session TOP SET (highest e1RM across working+drop sets with w>0 and r>0). Chronological ascending so the caller can `slice(-2)` / `slice(-6)` without resorting. Warmups are excluded.
110. **`getCurrentE1RM(history)` (`helpers.js`).** Layer 1 input to the recommender — max of the last two sessions' top-set e1RMs. Filters out a single bad day per spec §2.1.
111. **`getProgressionRate(history)` (`helpers.js`).** §2.3 linear regression of e1RM on days over the last 6 sessions. Returns `{ rate, rSquared, n, slope, meanE1RM }` where `rate` is the fractional weekly gain (`slope × 7 / mean`). Caller gates usability with `n ≥ 4 && R² ≥ 0.4` and falls back to `0.01` (compound) or `0.005` (isolation) when the fit is weak.
112. **`getRecommendationConfidence(n, rSquared)` (`helpers.js`).** §2.4 labels: `high` (n≥6, R²≥0.9), `moderate` (n≥4, R²≥0.6), `building` (n≥3), `none` (<3 — caller shows "Last:" only, no prescription).
113. **`recommendNextLoad({history, targetReps, mode, progressionClass, loadIncrement, now})` (`helpers.js`).** Top-level. Returns `{ mode, confidence, prescription: {weight, reps} | null, reasoning, meta }`. Implements the full §2.2 decision rule:
    - **`mode: 'push'`** (default) — Layer 3 nudge with aggressiveness multiplier 1.15 on the progression rate, capped at 3%/wk. `nextWeight = lastWeight × (1 + P·α) + 0.033·lastWeight·Δreps` where α = daysSince/7 and Δreps = lastReps − targetReps. Clamped to Layer 2 floor.
    - Hit target last session → apply Layer 3 nudge.
    - Missed by 1 rep → hold weight, "go for the reps."
    - Missed by 2+ reps → hold weight, push for reps (unless triggers auto-deload).
    - Auto-deload: missed by 2+ reps for two consecutive sessions → override to `mode: 'deload'`, prescribe 10% off last weight.
    - **`mode: 'maintain'`** — Layer 2 only (e1RM × %1RM(targetReps)), no nudge.
    - **`mode: 'deload'`** — 65% of current e1RM for recovery (midpoint of §2.5's 60–70% range).
    - <3 sessions → confidence=`none`, prescription=`last` (or null at n=0); reasoning counts down until recommendations unlock.
    - Result weight rounded to `loadIncrement` (default 5).
114. **`recommender-sanity.mjs` (repo root).** Node ESM — loads `debug-backup.json`, runs v2+v3 migrations to canonicalize sessions, then reports per-exercise fits + recommendations against the spec §5 high-confidence-ready list. Also exercises mode comparison on Pec Dec (push/maintain/deload produce distinct prescriptions), cold-start behavior (n=0/1/2), and the auto-deload trigger on a simulated 2-miss streak. Mirrors `migration-sanity.mjs` / `migration-v3-sanity.mjs` pattern — run from a worktree root with `node recommender-sanity.mjs`.
115. **Nothing wired to UI yet.** Step 3b (next batch) adds the inline `Try: 185×10` hint + expand-on-tap bottom sheet per §9.1 to the exercise card. Step 3c adds RPE/RIR capture per §3.7. This substep is a clean revert point.

### Batch 16b (April 18, 2026) — Recommender UI in exercise card (AI coaching prereq, step 3, substep b)

Step 3 UI surface — exposes the Batch 16a engine to the user per spec §9.1 Option C (inline hint + expand-on-tap) and §9.4 Option C (color + one-word confidence label).

116. **`Recommendation.jsx` — three display components** (`src/pages/log/Recommendation.jsx`). `RecommendationHint` renders the inline `● Try: 185×10` snippet appended to the collapsed card's "Last: 175×11" line; colored dot encodes confidence (green=Solid, amber=Maybe, gray=New); hidden when confidence='none'. `RecommendationBanner` is the prominent tappable banner at the top of the expanded body — two-line format with "TRY 185 × 10" / "●Maybe · short reasoning". `RecommendationSheet` is the bottom-sheet modal (createPortal, z-index 250 so it clears CustomNumpad's 200 and RestTimer's 50) with headline prescription, confidence chip, three side-by-side mode buttons (↑Push / →Maintain / ↓Deload — each showing its own weight×reps), reasoning card, and stats rows (current e1RM, last session + days-ago, progression fit, Layer 2 target).
117. **ExerciseItem integration** (`BbLogger.jsx`). Subscribes to `exerciseLibrary`; looks up entry by id (fallback by name); derives targetReps from `defaultRepRange` midpoint (default 10). Computes `recHistory` via `getExerciseHistory(allSessions, id, name)` — passing ALL bb sessions intentionally, NOT scopedSessions. The recommender is cross-workout-type by design (spec §1.3, §3.2: Pec Dec in push and push2 should share history), which differs from PR logic (workout-type-scoped per Batch 10/12). `recommendation` computed once via useMemo in mode='push'.
118. **Collapsed-card restructure.** The existing "Last:" `<p>` had `opacity-50` which was washing out the confidence dot on the appended hint. Replaced the single-paragraph with a flex div holding two siblings — the "Last:" span with its original opacity and the `RecommendationHint` at full opacity so the colored dot is legible.
119. **Sheet default mode = 'push'.** So the sheet opens aligned with what the banner renders (the banner always shows recs.push's output, even if auto-deload triggered within push mode). The DELOAD chip in the sheet still surfaces the user-selected deload (65% of e1RM) as the alternative — distinct number and reasoning from the decision rule's auto-triggered 10% off last. Two distinct deload prescriptions by design, per the spec.
120. **Verified live in preview** (mobile 375×812, debug-backup.json migrated to v3 with 24 sessions, 109 library entries). Pec Dec: banner "TRY 185×10 · ● Maybe · Hit target last session — pushing +2.3%…", sheet chips push 185 / maintain 175 / deload 155 with correct reasoning per mode. Overhead DB Extension (auto-deload triggered): banner 80 (10% off last); DELOAD chip 75 (65% of e1RM). DB Shrug (n=0): no banner. Cross-workout-type hint ("● Try: 80×10" without "Last:") for exercises logged elsewhere but not in the current workout. No console errors. ✓

### Batch 16c (April 18, 2026) — RPE capture + Layer 3 rep-accuracy boost (AI coaching prereq, step 3, substep c)

Step 3 per-set effort signal — spec §3.7 calls RPE "the single largest accuracy improvement available to the recommender." Optional at every layer (type, UI, persistence, engine) so forced entry doesn't kill adherence.

121. **Set schema + `getExerciseHistory` carry `rpe: number | null`** (`helpers.js`). `e1RM` unchanged (still uses raw reps). `getExerciseHistory`'s per-session top-set now carries rpe through when set, validated 1–10 inclusive. Saved-set shape in `buildExerciseData` gets `...(typeof s.rpe === 'number' && 1–10 ? { rpe: s.rpe } : {})`.
122. **Layer 3 uses effective reps when rpe is present** (`recommendNextLoad`). `rir = max(0, 10 − rpe)`, `effectiveReps = reps + rir`. Hit-target check is now `last.reps >= targetReps || effectiveReps >= targetReps`; Δreps in the nudge formula uses `effectiveReps − targetReps`. Same effective-reps logic feeds the auto-deload trigger, so a user who misses raw reps but still hit target via RIR (e.g. 8 reps @ RPE 8 with target 10) is NOT false-alarmed into a deload. Reasoning string appends "(8×@RPE8≈10 effective)" when rir>0 so the math is visible.
123. **`RpePill.jsx` — per-set chip + picker** (`src/pages/log/RpePill.jsx`). 44×40px pill inserted into both SetRow and PlateSetRow after the reps input. Empty state: muted `bg-item text-c-faint` with small "RPE" text. Set state: fuchsia-bordered pill (`bg-fuchsia-500/20 border-fuchsia-500/40 text-fuchsia-300`) with the number. Tapping opens a bottom-sheet `RpePicker` (createPortal, z:250) — two rows of 5 buttons with 6–10 prominent and 1–5 dimmed (matches the typical RPE range for strength training), plus an inline description per value ("8 → 2 reps in reserve — challenging" etc.), a 2-sentence explainer for new users ("RPE = reps in reserve. 10 is all-out failure; 8 means '2 more reps possible.'"), and a Clear button to unset.
124. **`PrevSetRow` (ghost rows) show RPE read-only.** If the prior top set had an rpe, shows it in a muted version of the pill. If not, renders a 44×36 invisible placeholder so column widths still align with the live row. No pill is shown for pre-16c sessions (they have no rpe field).
125. **Sanity (node, against debug-backup.json + synthetic rpe):** Pec Dec last 175×11, target 10 → no-RPE pushes 185, RPE 8 pushes 195 (Δreps=3 bonus), RPE 10 pushes 185 (rir=0, identical to no-RPE). Auto-deload bypass: 115×8×2 with RPE 8 (effective 10) does NOT auto-deload; pushes instead. All expected behaviors hit.
126. **Verified live in preview:** Pec Dec expanded → RPE pill tap → picker → select 8 → pill shows fuchsia "8". Toggle to Plates mode → RPE pill persists with same value. Clear button → pill returns to muted "RPE". Reload page mid-session → RPE value preserved via `activeSession` roundtrip. Ghost-row alignment correct. No console errors. ✓

Step 3 complete — the recommender is wired end-to-end: engine (16a) → inline+sheet UI (16b) → per-set RPE signal (16c). Step 4 (readiness check-in per §2.5) and step 6 (fatigue signals per §4) are next up in the plan.

### Batch 16d (April 18, 2026) — Revert RPE UI + back-off-sets future scope

First feedback round on the recommender. User observation: "most sets go to failure" → RPE captures zero signal for their logging pattern and adds visible clutter. Spec called RPE "largest accuracy improvement" but that assumed varied effort across sets.

127. **Reverted RPE UI** — deleted `src/pages/log/RpePill.jsx`. Removed the pill + picker from SetRow / PlateSetRow / PrevSetRow. Removed the `rpe` field from `buildExerciseData`'s saved set shape. Engine-side plumbing is **kept** (cheap and invisible): `getExerciseHistory` still carries `rpe` if present, `recommendNextLoad` still does RIR-aware Layer 3 math if set. Re-enabling later is a UI-only change.
128. **Back-off-sets future scope flagged** (`coaching-recommender-plan.md` Part 4). v1 prescribes the top working set only; a future expansion could prescribe back-off sets (e.g. `top: 185×10, back-off: 2× 175×10`). Trivial formula extension, interesting UI work. Revisit after readiness (step 4) ships.

### Batch 16e (April 18, 2026) — Plate clutter → nested popover + compact picker

User: "nest buttons within buttons" to conserve real estate. The bar selector, multiplier toggle, and 7-plate picker all living persistently on the card was too much.

129. **`PlateConfigPopover` (`BbLogger.jsx`)** — anchored-below-button popover rendered via `createPortal` (escapes the card's `overflow-hidden`). Fixed positioning computed from the anchor's `getBoundingClientRect` at open. Outside-click dismiss via `mousedown` + `touchstart` document listeners, with a 0ms `setTimeout` so the opening click doesn't immediately close it. z-index 220 — below RecommendationSheet (250) and above page content. Contains: Bar (45 / None / 25), Multiplier (1× / 2×), and "Turn off plate mode" button (the only way to disable plate mode — tapping Plates when already on re-opens the popover instead of toggling off, to prevent accidental data loss).
130. **Plates button summary inline.** When plate mode is active the Plates toggle shows a compact `45·2×` config summary so users see current state without opening the popover.
131. **`PlatePicker` (extracted sub-component in `BbLogger.jsx`)** — plate row now renders 5 common plates always (45/35/25/10/5), rare plates (100, 2.5) only when loaded OR user tapped the expand icon. Expand icon is a single-character `+` (collapsed) / `×` (expanded) — no text label per user feedback. Hidden entirely when all rare plates are already loaded. Loaded-rare plates stay visible even after collapse so users can still decrement them.

### Batch 16f (April 18, 2026) — Coach's Call sheet redesign (sparkline + plain English)

Second feedback round: the math in the sheet was "esoteric," "Maybe confidence" was meaningless out of context, three modes was one too many. Largely a rewrite of `Recommendation.jsx`.

132. **Inline SVG sparkline** — new `E1RMSparkline` component inside `Recommendation.jsx`. Plots the last 6 e1RM data points as circles + connecting line with a subtle dashed linear trend line underneath, `+10.5%/wk` rate label in the top-right. Auto-scales to window's min/max with 15% padding. Uses the exercise's accent color (`theme.hex` threaded from `ExerciseItem` → sheet → sparkline). The "killer feature" per user feedback — makes progression visceral in one glance instead of demanding the user parse R² and weekly %.
133. **Two modes, not three.** Maintain (left, easier) | Push (right, harder). Deload is gone as a user-selectable mode — it's still an algorithmic override when `autoDeload` triggers (reflected in `recs.push`'s output and its reasoning string). Default-selected chip is always 'push' so sheet stays aligned with the banner.
134. **Confidence as a percentage with tap-to-explain.** Replaced the opaque "Maybe confidence" amber label with `N% confident` — percent computed from `min(1, n/6) × R² × 100`, clamped 1–99. Green ≥80%, amber ≥50%, blue below. Tap the row to expand an explainer: "Based on N prior sessions — your e1RM has been tracking [very closely / closely / roughly / loosely] against the trend line" plus agency language ("Log M more consistent sessions and confidence will climb" when n<6, or a noise callout when n≥6 but R²<0.9). Hidden entirely when n<3.
135. **Plain-English reasoning** in the WHY panel. All strings in `recommendNextLoad` rewritten:
    - Hit target → "You hit W×R last session. Adding a little more weight based on how fast you've been progressing."
    - Missed by 1 → "You got R reps at W last time (target was T). Same weight — go for all T this session."
    - Missed by 2+ → "... Holding the weight — push for the reps before adding load."
    - Auto-deload → "You've missed the rep target two sessions in a row. Backing off 10% today to reset before pushing again."
    - Maintain → "Matching your e1RM at T reps — a solid maintenance day."
    - Deload → "Recovery day — 65% of your e1RM for an easier session."
    - Cold start → "Log M more sessions and I'll start prescribing weights."
136. **Details ▾ accordion** for the math: Estimated 1-rep max, Last session, Trend fit (n/R²/rate), This session's nudge (push mode only), Layer-2 Floor @ targetReps. Each row with an ⓘ is tappable to reveal a one-sentence explainer (what e1RM is, what R² means, why the floor exists, what the +3% cap is). New meta field `thisSessionNudgePct` on the recommender output (cap-adjusted, α-scaled) surfaces only in Details.
137. **"Top set" labeling** on both banner and sheet headline. Explicitly frames the scope so users know the prescription is for the top working set, not every set — primes the UI for the future back-off-sets expansion flagged in 16d.

Step 3 follow-up round complete — the recommender UI now matches user's design feedback. Next up per plan: step 4 readiness check-in (§2.5) and step 6 fatigue signals (§4).

### Batch 16g (April 18, 2026) — Sheet polish round 2

Round-2 feedback on the redesigned Coach's Call sheet. Largely cosmetic.

138. **ModeChip sub-component** (`Recommendation.jsx`). Maintain chip now has a blue flat-line SVG icon + blue accent; Push has an emerald rising-arrow SVG icon + emerald accent. Selected state tints its own accent color (border 40%, bg 10%). Default selected mode stays `push` — now visually unambiguous thanks to the green treatment.
139. **Copy cleanup.** Dropped the "WHY" eyebrow above the reasoning box — reasoning stands alone. "81% confident" became "Confidence: 81%" with pct in accent color and label in `text-c-secondary`. Chevrons unified to `▾`/`▴` on both Confidence and Details (removed the `?` icon that was confusing out of context).
140. **Hint rewrites** (all em dashes stripped, multi-line via `\n` + `whitespace-pre-line`). Formula on e1RM hint now on its own line so it doesn't wrap mid-expression. Trend fit, this-session nudge, and Floor @ N reps rewritten to be more educational — the nudge hint directly answers "why cap at 3% if the trend is 10%?", the floor hint shows the actual math worked through.
141. **Engine reasoning strings** (`helpers.js`) also scrubbed of em dashes.

### Batch 16h (April 19, 2026) — Collapsed card cleanup + plate reorder + plate Confirm

142. **Collapsed exercise-card cleanup.** Removed the inline "● Try: 185×10" hint AND the faded "Last: 175×11" line from collapsed rows. All prescription / historical info lives only in the expanded card's banner + sheet. The collapsed card now shows exercise name + REC pill (if enabled) + in-progress summary + completed state only.
143. **Toolbar emojis removed.** 🏋️ from Plates and ⏱️ from Last Time buttons — Last Time was wrapping to two lines on mobile because of the emoji padding. Labels read cleanly without them.
144. **Banner hidden when `confidence === 'none'`.** Cold-start exercises (fewer than 3 logged sessions) no longer surface a "Log N more sessions" banner whose only content is saying there's no prescription yet. Banner returns as soon as there are 3+ sessions.
145. **Plate split reorder.** `COMMON_PLATES` = `[100, 45, 35, 25, 10]` (100 promoted inline since it's used regularly); `RARE_PLATES` = `[5, 2.5]` (behind the + expand icon). Loaded rare plates still always visible so decrement works even when collapsed.
146. **Bar weight chip order.** Popover chips now ascend: `[None] [25 lb] [45 lb]` (was `[45][None][25]`). 45 still the init default via `barDefault` fallback. `BAR_CYCLE` constant updated.
147. **Plate popover Confirm button.** Bottom row is now `[Turn off plate mode]` (flex-grow 2) + `[✓ Confirm]` (flex-grow 1, emerald pill). Tapping Confirm closes the popover — state already applies on each tap, but the button gives users an explicit "I'm done" affordance without having to tap outside.

### Batch 16i (April 19, 2026) — AI coaching + Rec pill toggles; Dashboard banner move

148. **`settings.enableAiCoaching`** (default `true`). Gates `RecommendationBanner` + `RecommendationSheet`. When off, both disappear from the expanded card; engine (`getExerciseHistory`, `recommendNextLoad`, etc.) still runs in the render path — flipping back on is instant. Data collection is unaffected.
149. **`settings.showRecPill`** (default `true`). Gates the per-exercise blue REC chip (added in Batch 13). Some users never use the free-text prescription slot. Does not hide existing `exercise.rec` values; flipping back on re-reveals them.
150. **HamburgerMenu toggles.** Both added under the existing "Workout Defaults" section using the same switch pattern as `autoStartRest` / `restTimerChime`. Same theme-colored track when on.
151. **Dashboard banner moved.** The "Tag N custom exercises to unlock smarter recommendations" banner now sits between `SECTION 3: Hero Workout Card` and `SECTION 4: Stat Cards`, instead of at the top of the page (where it was encroaching on the "This Week" calendar header). Also dropped the 📋 emoji per the 16h "labels over emojis" principle.

### Batch 16j (April 19, 2026) — Manage Exercise Library + Backfill confirm flow

152. **`/exercises` page** (`src/pages/ExerciseLibraryManager.jsx`). Full library management surface. Header with filter chips (`All` / `Custom` / `Built-in` / `Untagged` — Untagged hidden when count=0), optional search input, scrollable list. Needs-tagging entries sort to the top within any filter that includes them. Each row shows name, TAG badge (for needsTagging) or Custom label (for `!isBuiltIn`), one-line summary of muscles/equipment, most-recent logged set + date. Tap → opens `ExerciseEditSheet`.
153. **`ExerciseEditSheet`** (`src/components/ExerciseEditSheet.jsx`). Bottom-sheet portal at z-index 260 (above `RecommendationSheet`'s 250). Pre-fills from the exercise's current values. Fields: name input, multi-pill primaryMuscles (≥1 required), single-pill equipment (Other filtered out so user has to pick a real type), `loadIncrement` choice chips (2.5 / 5 / 10), unilateral checkbox. Save button disabled until name + ≥1 muscle + real equipment. Delete button only for non-built-in entries and goes through a two-step "Delete → Delete permanently" confirm to prevent accidental wipes. Built-in entries remain editable — "built-in" just means seeded, not immutable.
154. **"My Exercises" link in HamburgerMenu** between "My Splits" and "Settings" — the new surface is discoverable without users needing to know the URL.
155. **`Backfill.jsx` rewrite — confirm flow.** Each card now holds LOCAL DRAFT STATE for muscle/equipment instead of auto-committing on every tap. User can fiddle freely; only an explicit Confirm commits to store. Confirm button at the bottom of each card is disabled ("Pick muscle group + equipment" in muted text) until both fields valid, then transitions to emerald "✓ Confirm". On tap: phase → 'saving', inline "✓ Saved" text shows, 350ms CSS fade + translateY(-6px), then `tagExercise` fires and the card drops off the list. Copy also rewritten to emphasize "edit later in My Exercises, no pressure to get it perfect first try." Completion screen now links to `/exercises` explicitly.

### Batch 16k (April 19, 2026) — Two hotfixes after the first full-flow test

156. **Timezone-drift on the Dashboard calendar strip.** Evening entries in a western timezone were landing ✓ on the wrong local day. Root cause: `addRestDaySession` (and every save path) stores `new Date().toISOString()` — UTC. The Dashboard's weekly/monthly calendar strips and the soreness check-in read those via `date.split('T')[0]` (the UTC date) and compared against `todayStr` computed from local getDate/Month/Year. For a 6:30 PM Friday entry in UTC-5, that pushed ✓ to Saturday. Fix: new `isoToLocalDateStr(iso)` helper in `Dashboard.jsx` parses the ISO and returns `YYYY-MM-DD` from local getters. All 6 `.split('T')[0]` usages on session/cardio/rest-day fields replaced. `buildActiveDaySet` in helpers.js was already local-correct (Batch 11), so the streak number was fine — only the calendar rendering was drifting.
157. **Cardio "No" button auto-saving in Finish modal.** Scenario B (no cardio logged today) had three chips: Yes / Log Now / No. Tapping "No" was wired directly to `onSave({cardioAction: 'none'})`, short-circuiting past the main Save button with no way to undo. Fix: `cardioChoice` state expanded to 'yes' | 'no' | null. No button now toggles between 'no' and null (symmetric with Yes), highlights when selected, and lets the main Save button at the bottom commit. Removed unused `handleSaveNo` function.

### Batch 16l (April 19, 2026) — Manage Exercises filter chips

158. **Workout + usage filter row on `/exercises`.** 109 entries were too many to scroll through. Added a second chip row below the existing source filter (All/Custom/Built-in/Untagged): `Any workout` / `[each workout in the active split with its exercise count]` / `Logged` / `Never logged`. The two axes combine independently, so a user can show e.g. "Custom + Never logged" to find unused custom entries.
159. **`exerciseIdsByWorkout` resolution (`ExerciseLibraryManager.jsx`).** `useMemo` walks each active-split workout's sections, unwraps string-or-`{name,rec}` exercises, and resolves names via `normalizeExerciseName` against a pre-built `(normalized name → library id)` map covering canonical names + aliases. Same approach as the v3 migration — renamed variants still match. Workout chips with 0 matching exercises are hidden.
160. **`loggedIds` set.** Flat scan of sessions builds a Set of every `exerciseId` with at least one session set. O(sessions × exercises). Drives both the `Logged` / `Never logged` chip filters and their counts. Workout display name uses the first segment before " — " (e.g. "Push" instead of "Push — Chest") to keep chip widths reasonable.

### Batch 16m (April 19, 2026) — Monthly calendar redesign

161. **Monthly grid cells → circles matching the weekly pill strip** (`Dashboard.jsx`). Every day cell is now a 36px circle (aspectRatio 1, borderRadius 50%) with the day number centered inside. State mapping:
    - Active day (session or cardio done): filled accent circle, contrastText number, bold.
    - Today pending / today-rest: transparent with 2px accent border, number in accent.
    - Today-logged-rest: muted white fill plus 2px accent ring.
    - Logged rest (past): muted white 14% fill, secondary-text number.
    - Rotation rest (past or future): 1px **dashed** border in muted color, muted number — dashed line signals "planned rest, not logged".
    - Future planned workout: 1px subtle solid border, muted number.
    - Empty past (missed): no shape, faint number.
    Dropped the workout-type emojis (🏋️ / 🦵 / 💪 / 🎯 / 🦿), the R / C single-letter overlays, and the floating blue cardio dot. Workout-type detail lives in History → session detail on tap; the monthly grid now answers "what happened on this day" in terms of activity state only. Tap handlers preserved (active → session detail, future → preview sheet, future-rest → rest-day sheet). Grid gap 3px → 6px for visual balance.

### Batch 16n (April 19, 2026) — Readiness check-in (AI coaching step 4)

Step 4 of the AI Coaching Recommender plan — pre-session prompt per spec §2.5. Replaces the plain Start Session overlay with three tappable rows (Energy / Sleep / Goal) plus a gym chip. Captured data feeds the recommender: Goal → mode (Recover=deload, Match=maintain, Push=push), and Energy+Sleep → aggressivenessMultiplier (low+poor=0.85, neutral=1.00, high+good=1.15) that scales push-mode nudging.

162. **`recommendNextLoad` accepts `aggressivenessMultiplier` (default 1)** (`helpers.js`). Scales the push-mode 1.15 aggressiveness constant and the `thisSessionNudgePct` display value. Zero effect in maintain/deload modes, and zero effect at the default (1.0) — so pre-16n callers and OK/OK users get identical output.
163. **`buildReadiness({energy, sleep, goal})` + `readinessMultiplier()` + `READINESS_GOAL_TO_MODE`** (`helpers.js`). Centralized mapping so the UI doesn't duplicate the goal→mode or energy+sleep→multiplier tables. Sanity-checked against `readiness-sanity.mjs` at repo root.
164. **`ReadinessCheckIn` component** (`src/pages/log/ReadinessCheckIn.jsx`). Three-row chip grid (§9.2 Option A), defaults OK/OK/Push. Selected state uses the user's accent color (`theme.bg` + `theme.contrastText`). Inline gym chip below the rows (§9.6 Option D): single chip text "Gym: VASA change" when gyms exist, or "+ Where are you lifting?" placeholder. Tapping opens an anchored popover with existing-gym list + inline "Add gym name…" field. Start Session commits readiness + gym; "Skip check-in" commits readiness=null; "Go back" navigates away.
165. **Gym store actions** (`useStore.js`). `settings.gyms: []` + `settings.defaultGymId: null` under existing settings slice (rides along export/import/merge without new code). Actions `addGym(label)` (case-insensitive dedupe, auto-sets default if first), `removeGym(id)`, `setDefaultGymId(id)`. Gym CRUD lives in the readiness chip for now — full Settings UI is deferred until step 8 when sessionGymTags + picker filters ship.
166. **BbLogger wiring** (`BbLogger.jsx`). `readiness` + `gymId` state in the main component (loaded from savedSession?.readiness/gymId so reload preserves). `handleStartSession(payload)` receives `{energy, sleep, goal, gymId}` from the overlay and calls `buildReadiness()`. `saveActiveSession` useEffect and the final `addSession` call both persist them. ExerciseItem gets two new props — `aggressivenessMultiplier` and `suggestedMode` — threaded to both the useMemo that powers the banner and to the RecommendationSheet as `defaultMode`.
167. **RecommendationSheet wakes up on each open** (`Recommendation.jsx`). New `useEffect([open, validInitial])` syncs `selectedMode` to the readiness-derived `defaultMode` every time the sheet opens. Previously useState's initializer only ran on first mount, so a user who picked Recover after the sheet had been opened once in Push mode would still see Push selected. Also a new `showDeloadChip` path surfaces a 3-chip layout (Deload | Maintain | Push) when `defaultMode === 'deload'`; user can still tap Maintain/Push to compare. Orange accent for the Deload chip with a new `DescendingLineIcon` SVG so the three chips read as an easier→harder continuum.
168. **`readiness-sanity.mjs` at repo root.** Node ESM — loads `debug-backup.json`, migrates through v2+v3, runs the recommender across the 3 readiness bands and the 3 goals against Pec Dec's real history. 9/9 multiplier lookups and 3/3 goal→mode mappings verified. Confirms Recover goal → 155 lbs (65% of e1RM deload), Match → 175 (maintain), Push → 185 (push).
169. **Verified live in preview** (mobile 375×812, debug-backup.json data, theme=blue): overlay renders with rows + gym chip + Start Session; selecting Low/Poor/Match persists `readiness.aggressivenessMultiplier=0.85, suggestedMode=maintain`; expanded Pec Dec card shows maintain prescription on the banner; tapping banner opens sheet with Maintain chip selected + "Matching your e1RM at 10 reps" reasoning. Recover-goal flow: banner 155×10, sheet opens 3 chips with Deload selected + orange accent. Skip flow: `readiness=null`, sessionStarted=true, push defaults apply. No console errors. ✓

### Batch 16n-1 (April 19, 2026) — Coach's Call polish

UX review round on readiness + Coach's Call after 16n shipped. No engine changes, no new data. Pure copy, layout, and chart clarity.

170. **Readiness labels: OK → Mid** (`ReadinessCheckIn.jsx`). Energy `Low/Mid/High`, Sleep `Poor/Mid/Good`. Keys stay `'ok'` internally so nothing migrates; only the display label changed.
171. **Banner → compact `RecommendationChip`** (`Recommendation.jsx`, `BbLogger.jsx`). The wide `RecommendationBanner` at the top of the expanded card is replaced by a small emerald pill `[✨ Tip]` in the toolbar row. Opens the sheet on tap. Same rendering guard (`settings.enableAiCoaching && prescription && confidence !== 'none'`). Banner export kept for back-compat but no longer consumed — candidate for deletion with a cleanup pass.
172. **Toolbar row unified** (`BbLogger.jsx`). All five chips (Plates, Uni, Last, PR, Tip) use `px-3 py-1.5 text-xs font-semibold` with `gap-2` between them. "Last Time" shortened to "Last" for fit. PR chip bumped up to match the other heights (was `px-2 py-1 fontSize:11`). Removes the inner `ml-auto` group wrapper — chips flow naturally left-to-right with even spacing.
173. **Sheet header collapsed to one line** (`Recommendation.jsx`). Previous two-eyebrow layout (`COACH'S CALL / Pec Dec` left, `TOP SET / 175×10` right, close far right) replaced with a single-line `✨ Recommended top set: **175 × 10**` in emerald, with `Last session's top set: 150 × 10` as a subtle subtitle beneath. Close button stays right. Exercise name (`Pec Dec`) promoted to a small heading ABOVE the recommended row, left-indented to align with the sparkle icon.
174. **Sparkline rewritten with explicit variable labels** (`Recommendation.jsx`, `E1RMSparkline`). Removed all in-chart text (floating "EST. 1-REP MAX" caption, rate label, peak callout). The chart now renders ONLY the line, trend line, and dots — clean visual. Explicit labels wrap it:
    - **Title above**: `Estimated 1-rep max · last 6 sessions`
    - **Stat key below**: `Peak: 239 lbs` (left) · `Growth: +7.0%/wk` (right)
    Each number has a named label; no orphaned numbers on the chart. Trend line kept; peak dot is 4.5px (vs 3.5px for latest, 2.5px for others) so the peak stands out visually as the "239 lbs" value the label refers to.
175. **Mode chips collapsed to single line** (`Recommendation.jsx`, `ModeChip`). Was 3 stacked rows (icon+label / weight×reps / sub-label). Now one row: `[icon] LABEL  weight×reps`. Dropped the "Keep it steady" / "Go for progress" sub-labels — redundant with icon+label. Padding reduced from `py-3 px-3` to `py-2 px-3`. Significant vertical and horizontal space saved.
176. **Hit-target reasoning split by math driver** (`helpers.js`, `recommendNextLoad`). Vague "Adding a little more weight based on how fast you've been progressing" replaced with branch-specific strings:
    - **Floor-driven, catching up** (most common): "Your recent top sets put your estimated 1-rep max around {e1RM} lbs, which projects to {floor} for {targetReps} reps. Last session you went {last.weight}×{last.reps}, lighter than your strength suggests, so today's weight catches you back up to your actual level."
    - **Floor-driven, steady**: "Matching your current strength level: {e1RM} lb e1RM projects to {floor} for {targetReps} reps, right around last session's {last.weight}×{last.reps}."
    - **Nudge-driven**: "You hit {last.weight}×{last.reps} last session, right at your current strength level. Bumping load by +{bump} lbs today based on your progression trend (capped at +3% per elapsed week to keep it sustainable)."
177. **Details: `This session's nudge` → `vs last session: +25 lbs (+16.7%)`** (`Recommendation.jsx`). The old "nudge" row reported an internal engine sub-value (Layer 3's `P × alpha × 100`) that was 0% whenever `daysSince=0` and irrelevant when the Floor drove the prescription — confusing. Replaced with the actual session-over-session delta, which matches what the user sees on the banner. Old nudge row is gone from the UI; `thisSessionNudgePct` is still computed in the engine meta block for anyone who wants it.
178. **`daysAgoLabel(0)`: `today` → `earlier today`** (`Recommendation.jsx`). "Last session: 150×10 · today" was ambiguous alongside a session in progress. `earlier today` disambiguates — explicitly a prior session.
179. **Pec Dec heading spacing tightened** (`Recommendation.jsx`). `mb-1` (4px) below the exercise name reduced to `mb-0.5` (2px) to match the `mt-0.5` between Recommended and Last session subtitle. Visual rhythm now balanced across the three title lines.

No console errors. All changes verified in the preview across Push (default), Match (maintain), Recover (deload), and Skip flows.

### Batch 16o (April 19, 2026) — Fatigue signals (AI coaching step 6)

Step 6 of the AI Coaching Recommender plan per spec §4. Engine-only (no new UI surfaces) — just four contextual multipliers stacked alongside the readiness multiplier, plus a one-sentence fatigue prefix on the reasoning when a signal materially moved the prescription. User explicitly called for the cardio factor to be "slightest manner" since routine cardio shouldn't drag next-day readiness; implemented accordingly.

180. **`gradeMultiplier(grade)` (`helpers.js`).** Prior bb session's grade scales aggressiveness: A+=1.10, A=1.05, B=1.00, C=0.95, D=0.90, null=1.00. Mirror of readiness but from observed past performance. Scoped to most recent graded bb session (any workout type).
181. **`cardioDamping(cardioRecent)` (`helpers.js`).** Deliberately minimal per user feedback: only triggers on `intensity === 'allout'` AND `hoursAgo < 24`, at 0.98× (2% reduction). Easy/moderate/hard cardio has zero effect regardless of timing; all-out cardio >24h ago also has zero effect. Tiny guardrail for the extreme case without being noisy for routine cardio.
182. **Rest-day boost (`helpers.js`).** When a rest day was logged within ~36h (covers "yesterday" without timezone gymnastics), aggressiveness gets a 1.05× boost. No-op otherwise.
183. **`gapAdjustment(daysSince)` (`helpers.js`).** Long inter-session gap tempers the final prescription and caps the nudge alpha. `daysSince > 14`: `{ mult: 0.85, alphaCap: 2 }`. `daysSince > 10`: `{ mult: 0.95, alphaCap: 2 }`. Otherwise no-op. Protects against the 3-week-gap → `alpha=3` → tripled nudge → injury scenario.
184. **`buildFatigueSignals({sessions, cardioSessions, restDaySessions, now})` (`helpers.js`).** Resolves raw store slices into `{priorGrade, cardioRecent, restedYesterday}`. Called once per BbLogger render via `useMemo`; signals ride into every ExerciseItem's recommender call. Keeps the recommender pure (no direct store coupling).
185. **`recommendNextLoad` accepts `fatigueSignals = {}`** (`helpers.js`). All four multipliers stack onto the existing `aggressivenessMultiplier` inside the push branch: `aggressiveness = 1.15 × readinessMult × gradeMult × cardioMult × restMult`. Alpha is clamped by `gapAdjustment.alphaCap`. Final prescription is additionally scaled by `gap.mult` when the gap is long enough to warrant tempering. Maintain and deload branches ignore fatigue signals entirely (as they ignore readiness).
186. **`buildFatigueReasoningPrefix` (`helpers.js`).** Composes a one-sentence prefix for material signals: D/C grade → "Last session graded D, so holding back a touch."; A+/A → "Last session graded A — pushing a touch more today."; all-out cardio → "Taking it a touch easier after your all-out cardio session."; long gap → "It's been N days — ramping back up gradually, not catching up."; rest day (no cardio damp) → "Coming off a rest day, giving you a little more room." Only one prefix surfaces at a time; priority is warn-first (deload-ish signals before boost signals).
187. **`thisSessionNudgePct` reflects composed aggressiveness** (`helpers.js`). The engine meta value is now computed against the full `readiness × grade × cardio × rest` product and the gap-capped alpha, for accuracy if any future UI surface exposes it. Currently unused in UI (replaced by "vs last session" delta in 16n-1).
188. **BbLogger wiring** (`BbLogger.jsx`). `fatigueSignals` computed once at the top level via `useMemo(() => buildFatigueSignals({sessions, cardioSessions, restDaySessions}), [...])` and passed to every ExerciseItem. ExerciseItem threads it to both the recommendation useMemo and the RecommendationSheet. Sheet passes it into all three mode calls inside `recs`.
189. **`fatigue-sanity.mjs` at worktree root.** Node ESM — validates 6/6 grade multipliers, 6/6 cardio damping cases, 6/6 gap adjustments, end-to-end reasoning prefix firing on real Pec Dec history, plus a synthetic slow-progression (+1%/wk) scenario that proves multipliers bite when the 3% cap isn't saturated (baseline nudge 1.14% → D 1.03% → A+ 1.26%), plus a 21-day gap scenario (150→130, "ramping back up gradually").
190. **Verified live in preview** (mobile 375×812, debug-backup.json data, theme=blue). Opened Pec Dec Tip → sheet renders with reasoning: "Last session graded A — pushing a touch more today. You hit 175×11 last session, right at your current strength level. Bumping load by +10 lbs today based on your progression trend (capped at +3% per elapsed week to keep it sustainable)." Fatigue prefix fires correctly from the backup data's A-graded prior session. No console errors. ✓

### Batch 16p (April 19, 2026) — Polish sweep + §3.8 auto-classify

Small-scope housekeeping pass. Sweeps the UX items flagged in 16n-1, deletes dead code, and ships the remaining small v1 engine item (§3.8 warmup/working auto-classification on weight entry). No new features or schema changes.

191. **Maintain = Push chip collapse** (`Recommendation.jsx`). When `recs.maintain.prescription.weight === recs.push.prescription.weight` (the Floor-dominated case), the Maintain chip is hidden and the grid shrinks to `grid-cols-1`. `effectiveMode` forces selection to `push` when the stored `selectedMode` was `maintain`, so the reasoning and Details pane stay consistent with the visible chip. Deload-three-chip layout still takes precedence when the user's goal was Recover.
192. **"Floor @ N reps" → "Strength at N reps"** (`Recommendation.jsx`). The word "Floor" read as "lowest allowed" and confused users when the Floor equaled the prescription (common in catch-up mode). Renamed throughout the Details pane and updated the hint copy to describe it as a strength-level projection (e1RM × %1RM at target reps), not a floor. The "vs last session" hint also swapped the internal "floor / nudge" wording for user-facing "strength level / weekly nudge".
193. **Sparkline tap-to-select** (`Recommendation.jsx`, `E1RMSparkline`). Dots are tappable again with an invisible 14px hit area. Tapping swaps the "Peak: 239 lbs" key label to "Session N: 225 lbs · 150×10" (e1RM value + session's top set). Tapping the same dot clears back to Peak. State lives in the sheet (`sparkSelectedIdx`) and resets on each open. Re-enabled by lifting state out of the sparkline so the key line can reflect the selection.
194. **Dead export cleanup** (`Recommendation.jsx`). Removed the unused `RecommendationHint` and `RecommendationBanner` exports plus their helper fns (`hasRenderableHint`, `hintDotColor`). Neither was consumed since Batch 16n-1 (banner → chip) and 16h (hint removal). Comment in `useStore.js` also updated `RecommendationBanner` → `RecommendationChip` for the `enableAiCoaching` setting.
195. **Auto-classify warmup/working on weight entry** (`BbLogger.jsx`, spec §3.8). In `updateSet`, when the user changes the weight and the set isn't a drop set, we compute `weight / recentTopE1RM` and classify: `<60%` → `warmup`, `>80%` → `working`, middle band keeps current type. `recentTopE1RM` is read from the last session's top-set e1RM via `recHistoryRef` (a ref populated after `recHistory` is computed, so `updateSet`'s closure can read the latest value without a TDZ ReferenceError). No history → skip entirely. User can always manually override afterward.
196. **Verified live in preview** (mobile 375×812, debug-backup.json). Pec Dec Tip sheet: renders with "Estimated 1-rep max · last 6 sessions" title, "Peak: 239 lbs · Growth: +7.0%/wk" key, clean chart. Both Maintain (175×10) and Push (185×10) chips render when they diverge. Details pane: "Strength at 10 reps: 175 lbs" row + "vs last session: +10 lbs (+5.7%)" row, no "Floor @" anywhere. No console errors. ✓

### Batch 16q (April 19, 2026) — Anomaly coaching surfaces (AI coaching step 9)

Step 9 of the AI Coaching Recommender plan per spec §4.5 + §9.3. Engine-plus-small-UI pass: three detectors (plateau, regression, swing) run over the existing per-exercise e1RM history and surface a contextual banner on each expanded exercise card. No schema changes; dismissal rides the existing settings slice. User-clarified this round: banner body tap does nothing (icon + copy + dismiss X only, per "less is more"); dismiss lasts only the current active session — the banner returns next session if the detector still fires.

197. **`detectPlateau(history, {minSessions=6})`, `detectRegression(history, {minSessions=3, rateThreshold=-0.01})`, `detectSwing(history, {threshold=0.30})`** (`helpers.js`). Pure functions — no store access, no time dependency beyond the input history. Plateau uses the existing `getProgressionRate` window; fires when `|rate| < 0.005` (±0.5%/wk). No R² gate because a perfectly flat line yields rSquared=0 in our code via the `ssyy===0` branch, which would otherwise falsely block the detector. Regression uses the same fit plus an R² ≥ 0.4 gate (matches the recommender's `usedFit` cutoff) so noisy scatter doesn't trigger a warning. Swing is a simple session-over-session delta: `|last.e1RM - prev.e1RM| / prev.e1RM > 0.30`.
198. **`detectAnomalies(history)` aggregator** (`helpers.js`). Runs all three and returns the highest-priority non-null result, or null. Priority: **regression > swing > plateau** — warning first (things are getting worse), data-quality second (same machine?), passive observation third (you're stuck).
199. **`AnomalyBanner` component** (`Recommendation.jsx`). Inline-styled bottom-sheet-free div: 10px padding, 10px border-radius, severity-keyed tint (amber `rgba(245,158,11)` for `warn` — regression; blue `rgba(59,130,246)` for `info` — plateau / swing). One-line copy left, 28×28 dismiss ✕ button right. No portal — renders inside the exercise card. `buildAnomalyCopy(anomaly, name)` co-located: plateau "You've been flat on {name} for the last {n} sessions. Try dropping 10% and chasing reps this week to break through."; regression "Trend on {name} has dipped the last {n} sessions. Consider a lighter recovery week, then build back up."; swing "Your top set on {name} swung {dir} {pct}% from last session. Same machine? Same range of motion?" Copy respects the no-em-dashes preference throughout.
200. **`settings.dismissedAnomalies: {}`** (`useStore.js`). New settings field, `{ [exerciseKey]: sessionId }`. Additive — no persist version bump (rides the existing `settings` deep-merge on load). Companion action `dismissAnomaly(exerciseKey)` stamps the current `activeSession.startTimestamp` against the key, so a stale dismissal from a previous session fails the match check automatically and the banner reappears. No cleanup logic needed — the map stays small (at most one entry per exercise).
201. **`ExerciseItem` wiring** (`BbLogger.jsx`). `anomaly = useMemo(() => detectAnomalies(recHistory), [recHistory])`. Anomaly key is `exercise.exerciseId || exercise.name` (falls back to name because template-seeded exercises don't carry exerciseId until save time via `buildExerciseData`). Render gate: `settings.enableAiCoaching && anomaly && !dismissedThisSession`. Banner slots between the toolbar row (Plates / Uni / Last / PR / Tip) and the column headers (`Type / Lbs / Reps`). New prop `activeSessionId={startTimestamp.current || null}` threaded from parent BbLogger so the dismissal check has a stable per-session id.
202. **`anomaly-sanity.mjs` at worktree root.** Node ESM. 25/25 synthetic + edge-case assertions pass: plateau on 6 flat values, regression on 6 declining values (R²=1.0, rate=-9.3%/wk), swing on a +39% and -35% jump, priority order when multiple fire, healthy-gains case fires nothing, n<6 flat does NOT trigger plateau, noisy scatter does NOT trigger regression. Real-data pass against `debug-backup.json` logs informationally — Pec Dec / Chest Supported Wide Row / Seated Cable Row / Incline DB Press all return "nothing fires" on the user's healthy progression, as expected.
203. **Verified live in preview** (mobile 375×812, debug-backup.json with synthetic injections). Plateau scenario (6 Pec Dec sessions flat at 180×10): banner renders with blue `rgba(59,130,246,0.08)` tint + blue border and correct copy referencing "6 sessions". Dismiss ✕ hides the banner; `dismissedAnomalies.Pec Dec === activeSession.startTimestamp` in localStorage. Collapsing + re-expanding the card keeps the banner hidden. Regression scenario (200→100 linear decline): amber `rgba(245,158,11,0.1)` tint, regression copy. Swing scenario (+39% last-over-prev): blue tint, "swung up 39% from last session. Same machine?" copy. Flipping `settings.enableAiCoaching = false` hides both the banner AND the Tip chip; flipping back on restores both. No console errors throughout. ✓

### Batch 16r (April 19, 2026) — Menu restructure + How AI Coaching Works explainer

User-requested reorganization of the HamburgerMenu Settings screen to separate profile concerns from workout concerns, plus a new plain-English "How AI Coaching Works" explainer under Info. No engine changes, no schema changes.

204. **Settings restructure** (`HamburgerMenu.jsx`). The "Appearance" and "Account" sections collapsed into a single **Profile Settings** section containing Your Name (moved to top — users see their name field first), Theme, and Accent Color. **Workout Defaults** stays as-is: Default first set / Auto-start rest timer / Rest timer chime / AI coaching / Show Rec pill. Previous three-section layout (Appearance / Workout Defaults / Account) becomes two (Profile Settings / Workout Defaults). Matches the user's mental model: "who I am" vs "how I train."
205. **AI coaching description updated** (`HamburgerMenu.jsx:290`). Was "Show the coach's call banner + sheet" (written for Batch 16i when the only surface was Coach's Call). Now reads "Coach's Call tip + anomaly banners" so the toggle's expanded 16q scope is explicit in the UI.
206. **"How AI Coaching Works" expandable** (`HamburgerMenu.jsx`). New collapsible entry under Info, alongside "How Tracking Works" and "How Streaks Work". Plain-English 2400-char explainer covers: your-strength estimate (Epley in layman's terms), today's suggestion (Layer 2 + Layer 3, capped 3%/wk in plain English), the readiness check-in (energy/sleep/goal), what the coach watches between sessions (grade, cardio, rest day, gap), miss-the-reps logic (hold weight / auto-deload), the three anomaly banners (plateau / regression / swing), and what the coach won't do (local data only, suggestion not rule, waits for enough history). Respects user UX preferences: no em dashes, less-is-more defaults (section starts collapsed), labels + strong-tagged key terms rather than bullet-heavy formatting.
207. **Verified live in preview** (mobile 375×812, debug-backup.json). Menu → Settings: Profile Settings section lists Your Name → Theme → Accent Color in that order; Workout Defaults lists the five toggles in their original order. AI coaching toggle's description reads "Coach's Call tip + anomaly banners". Menu → Info: three expandables present — How Tracking Works / How Streaks Work / How AI Coaching Works — the third expands to 2435 chars of explainer content rendering without layout issues. No console errors.

### Batch 17a (April 19, 2026) — Split draft auto-save + persist v4 migration

First step of the Split Builder redesign (see `split-builder-redesign-handoff.md` Step 1). Closes the data-loss class where an accidental tab brush, OS kill, or tab close mid-wizard wiped the in-progress split. Additive store slice + persist bump + debounced auto-save + resume banner. No UI on the wizard side beyond the banner — engine-invisible otherwise.

208. **`splitDraft` store slice + `setSplitDraft` / `clearSplitDraft` actions (`useStore.js`).** Shape: `{ originalId: string | null, draft: PartialSplit, updatedAt: number } | null`. `originalId` is `null` when creating new, or a split id when editing. One draft at a time — a second create overwrites any prior create draft; same for edit drafts scoped to the same id. The `draft` object carries only what the user has typed so far (name / emoji / workouts / rotation) so partial state is fine. Actions are dumb setters — debouncing + dirty-tracking live in the consumer.
209. **Persist version bumped `3 → 4`** with additive `migrate` block — sets `splitDraft = null` when upgrading from v3. Pre-v4 users have no drafts on disk, which is the correct initial state. The `merge` hook also preserves `splitDraft` explicitly so it survives schema evolution. Idempotent: re-running the v4 block on a v4 state with a live draft preserves the draft untouched.
210. **SplitBuilder auto-save + resume banner (`SplitBuilder.jsx`).** New `isDirty` flag + debounced (500ms) `useEffect` that writes the current wizard state to `setSplitDraft` whenever name/emoji/workouts/rotation change. Every setter passed down to Step1 / Step3 / workout-builder is wrapped so typing / adding / reordering all flow through one dirty pipeline. Mount effect: if a matching draft exists for this route context (create OR edit-of-this-id), show an amber banner at the top of Step 1 offering Resume / Discard. Banner copy includes `formatTimeAgo(updatedAt)` so users see "last saved 3m ago" rather than a raw timestamp.
211. **Auto-clear safety rails (`SplitBuilder.jsx`).** Drafts older than 7 days auto-clear on mount (constant `DRAFT_STALE_MS`). Drafts with an `originalId` that no longer exists in `splits[]` (user deleted that split between drafting and returning) also auto-clear. Draft scoping: a create-mode draft shows the banner on `/splits/new` only, an edit-mode draft shows only on `/splits/edit/:id` where `id === originalId` — mismatched edit URLs leave the draft untouched (so returning to the owning url still surfaces it). `handleSave` and the explicit Discard button both call `clearSplitDraft()` so successful saves + intentional discards never leave orphan state behind.
212. **`formatTimeAgo(tsOrIso)` helper (`helpers.js`).** Accepts a ms epoch or an ISO string. Returns "just now", "Nm ago", "Nh ago", "yesterday", "Nd ago" (under 7 days), or `formatDate(...)` beyond. Handles clock-skew (negative diffs → "just now") and invalid inputs → empty string.
213. **`draft-sanity.mjs` at worktree root.** Node ESM — exercises the v3→v4 additive migration on a stripped backup, a full roundtrip through JSON.stringify/parse of a representative draft (create + edit), the clear path, and the edge cases `formatTimeAgo` handles (1m / 2h / yesterday / just-now / future / 10 days). 20/20 assertions pass. Mirrors the existing `migration-sanity.mjs` / `migration-v3-sanity.mjs` / `recommender-sanity.mjs` / `fatigue-sanity.mjs` / `anomaly-sanity.mjs` pattern.
214. **Verified live in preview** (mobile 375×812). Six scenarios all pass — (A) create new / leave / resume restores name+emoji+workouts; (B) edit-mode scoping — banner fires on `/splits/edit/split_bam` when the draft's originalId matches, silent on `/splits/edit/other_id`; (C) Discard button nulls the store slice + hides the banner; (D) completing the wizard and tapping Save Split nulls the draft and navigates to `/splits`; (E) manually aging `updatedAt` 8 days auto-clears on next mount; (F) orphaned-split draft (originalId pointing at a deleted split) auto-clears on mount. No console errors across all six flows. ✓

### Batch 17b (April 19, 2026) — Hide bottom nav on split editor routes

Step 2 of the Split Builder redesign (see `split-builder-redesign-handoff.md` Step 2). Closes the silent data-loss class where a mid-wizard tab brush killed the in-progress split. Works in concert with Batch 17a's auto-save: 17a makes the loss recoverable, 17b makes it rarer in the first place.

215. **`BottomNav` hide predicate extended (`BottomNav.jsx`).** Was `isLogging || isWelcome`; now is `isFullscreenFlow || isWelcome` where `isFullscreenFlow` matches any of `/log/bb/*`, `/splits/new*`, `/splits/edit*`. The `/splits/new/start` future route for Step 6's ChooseStartingPoint is covered by the `/splits/new` prefix check — no separate branch needed, zero cost before Step 6 lands. `/splits` list view stays visible so users can tap cards and overflow actions without the nav disappearing.
216. **`HamburgerMenu` mirrors the predicate (`HamburgerMenu.jsx`).** Same `isFullscreenFlow` check so a stale `open` state can't surface the slide-in menu mid-wizard. The menu also exits cleanly when the user enters a builder flow from the list view.
217. **Verified live in preview** (mobile 375×812). Nav hidden on `/splits/new`, `/splits/new/start`, `/splits/edit/split_bam`; visible on `/splits`, `/dashboard`, `/log`, `/history`, `/progress`, `/cardio`, `/guide`, `/exercises`, `/backfill`. Transition check `/splits → /splits/new → /splits`: visible → hidden → visible. No console errors. ✓ (Welcome route still hides the nav when reachable; for onboarded users the route redirects to /dashboard, so the direct test reported Welcome nav as visible — that's the redirect result, not Welcome itself.)

### Batch 17c (April 19, 2026) — SplitManager card-tap activation + overflow menu

Step 3 of the Split Builder redesign (see `split-builder-redesign-handoff.md` Step 3, decision D4). Honors the "Tap a split to activate it" copy that was previously inert. Folds the five per-split actions (Set Active, Edit, Duplicate, Export, Delete) into a single ⋯ overflow menu so the card body is free to act as the primary activation affordance.

218. **`SplitCard` root is now `role="button"` (`SplitManager.jsx`).** Card tap calls `setActiveSplit(split.id)` directly. `aria-pressed` reflects the active state; `aria-label` switches between `Activate {name}` and `{name}, currently active`. Keyboard path: Tab focuses the card with a ring-2 accent focus ring, Enter / Space activates. Already-active cards no-op on tap (no accidental re-activation toast or side effect).
219. **Inline action buttons removed in favor of ⋯ overflow (`SplitManager.jsx`).** The old row of `[Set Active]` / `Edit` / Export / Delete buttons is gone; a single 36×36 ⋯ button in the top-right of each card opens a popover menu. Menu items are conditional: Set Active hidden when already active, Delete hidden for built-in splits. The ⋯ button `stopPropagation()`s on click so the card tap doesn't fire simultaneously.
220. **`OverflowMenu` portal component (`SplitManager.jsx`).** Renders via `createPortal` into `document.body` at z-60 (above page content, below the already-established sheet/modal stack). Positioned with `getBoundingClientRect()` — right-edge aligned to the anchor, 6px below, with viewport-edge clamping at 8px. Dismisses on outside mousedown/touchstart (listener attached via 0ms setTimeout so the opening click doesn't immediately close it) OR Escape. Menu items carry inline SVG icons (star / pencil / copy / export / trash) for quick visual scan. Destructive action (Delete) renders in red.
221. **Duplicate stays on the list (`handleDuplicate`).** `cloneSplit` fires and the new `"(Copy)"` entry appears in place; no auto-navigate to the builder. Users who want to edit the duplicate can open its own ⋯ menu and pick Edit — keeps the surface predictable, since a surprise redirect breaks the "I was just making a copy to reference" flow.
222. **Copy updated (`SplitManager.jsx`).** The header instruction reads `Tap a split to activate it. Use ⋯ for more actions.` — matches the new model exactly. The old "Clone built-in splits to customise them" line is removed since Duplicate is now in the menu for every split.
223. **Verified live in preview** (mobile 375×812). Six scenarios pass: (a) Tapping an inactive card sets it active, `aria-pressed` + accent border update, active-card aria-label becomes `{name}, currently active`; (b) Tapping an already-active card is a no-op; (c) ⋯ on an active built-in split shows `Edit / Duplicate / Export` (3 items); (d) ⋯ on a non-active non-built-in split shows `Set Active / Edit / Duplicate / Export / Delete` (5 items); (e) Duplicate adds `"(Copy)"` entry without navigating away, splits count +1, hash stays at `#/splits`; (f) Edit navigates to `/splits/edit/:id`. Outside-click + Escape both dismiss the menu; tapping the ⋯ button itself doesn't fire the card-tap. No console errors. ✓

### Batch 17d (April 19, 2026) — Delete legacy SplitEditor + /split route

Step 4 of the Split Builder redesign (see `split-builder-redesign-handoff.md` Step 4). `SplitEditor.jsx` was a pre-splits artifact that reordered only the built-in split's rotation. SplitBuilder's Step 3 (rotation) replaces it entirely. Nothing in the current UI links to `/split` anymore.

224. **Deleted `src/pages/SplitEditor.jsx`.** Confirmed via grep that no other file references the component or the singular `/split` route. The related `customTemplates` slice and `workoutSequence` field are kept (still referenced by pre-splits session history and the TemplateEditor fallback).
225. **Removed import + `<Route path="/split">` from `src/App.jsx`.** The `/split` URL now falls through to React Router's default no-match behavior (blank), which matches every other unregistered route. Any stale bookmark lands in the same place; not worth a redirect shim for a route that was never linked from the UI.
226. **`useStore.js` comment clean-up.** The `updateWorkoutSequence` docblock used to reference "the existing SplitEditor" — rewritten to explain the real reason the action still exists (legacy built-in split rotation sync).
227. **Verified build** via `npx vite build --outDir /tmp/test-build` — bundle size down ~6 KB. Manual smoke: `/dashboard → /splits → /splits/edit/:id` unaffected. No broken imports, no unused-symbol warnings. No preview walkthrough needed — change is subtractive and the affected surface had no reachable users.

### Batch 17e (April 19, 2026) — Duplicate action + undo toast

Step 5 of the Split Builder redesign (see `split-builder-redesign-handoff.md` Step 5). Adds Duplicate to both split cards and workout cards in the builder — Push 2 is 80% the same as Push in most real splits, so this cuts real friction. The Duplicate action on SplitManager was shallow-cloning in Batch 17c (via the existing `cloneSplit`), which actually shares workout object references and mutates both splits when one is edited. This step replaces it with a proper deep clone.

228. **`Toast.jsx` component + `showToast` event bus (`src/components/Toast.jsx`, `App.jsx`).** Module-level listener registry so any component can call `showToast({ message, undo, duration=5000 })` without prop threading. Single toast at a time (a new call replaces the active one). Renders via `createPortal` at z-index 290 per the handoff's z-stack — above every established sheet / modal / popover. Tailwind-styled pill: accent-tinted border, bold "UNDO" in amber. Mounted once at the top of `App.jsx` alongside `<RestTimer />`.
229. **`duplicateSplit(splitId)` store action (`useStore.js`).** Proper deep clone with workout id regeneration and rotation remap. Builds an `old-id → new-id` map, rewrites each workout's id, deep-copies the `sections` / `exercises` arrays, preserves `{name, rec}` structure and scalar rec strings, then rewrites the split's `rotation` through the map (`'rest'` passes through unchanged; dangling ids survive via `|| r` fallback instead of silently dropping). Returns the dup so the caller can surface an undo toast with the dup's id. `cloneSplit` is kept untouched since it's still referenced by import flows.
230. **`removeSplitById(id)` store action (`useStore.js`).** Companion to `duplicateSplit` — symmetric removal by id. `deleteSplit` already exists but goes through the `confirmDelete` modal; this one is a direct-delete path for the undo toast flow. Also handles activeSplitId fallback if the removed split happened to be active.
231. **SplitManager Duplicate wired to the new action (`SplitManager.jsx`).** `handleDuplicate` now calls `duplicateSplit` and fires `showToast({ message: 'Duplicated "X"', undo: () => removeSplitById(dup.id) })`. 5-second auto-dismiss; Undo button inside the toast removes the dup. Still stays on the list view — no surprise redirect to the builder.
232. **SplitBuilder Step 2 workout duplicate (`SplitBuilder.jsx`).** New inline copy-icon button added to each workout card's action row (between "Move down" and "Remove"). `handleDuplicateWorkout(idx)` deep-clones sections + exercises + rec, generates a fresh workout id, appends `"(Copy)"` to the name, and appends to `workouts`. Does NOT auto-add to rotation (user typically wants Push 2 at a different rotation position than Push, so making the choice explicit is better than silent append). Toast with 5s undo restores the pre-duplicate workouts array via closure-captured snapshot.
233. **Verified live in preview** (mobile 375×812). Split duplicate scenarios pass: (a) built-in BamBam's Blueprint duplicated → new entry with `"(Copy)"` suffix, `isBuiltIn: false`, fresh workout ids, rotation correctly remapped to the new ids ('rest' passes through unchanged); (b) toast renders with emerald Undo button; (c) Undo removes the dup from the store, splits count back to pre-dup value. Workout duplicate scenarios pass: (d) Duplicate Push → "Push (Copy)" appended to Step 2 workout list, not added to rotation; (e) toast renders; (f) Undo removes "Push (Copy)" from the builder's local state. No console errors. ✓

### Batch 17f (April 19, 2026) — ChooseStartingPoint + 6 split templates

Step 6 of the Split Builder redesign (see `split-builder-redesign-handoff.md` Step 6, decisions D2 + D3). New users who don't think in BamBam's Blueprint terms had no scaffolding for what a "workout" / "rotation" means. This step introduces a runway screen at `/splits/new/start` with 6 opinionated templates plus a Blank slate plus an Import entry point. Tapping a template seeds `splitDraft` and routes to `/splits/new` where the existing resume-banner hands it off to the wizard's local state.

234. **`SPLIT_TEMPLATES` + `loadTemplateForDraft(id)` data file (`src/data/splitTemplates.js`).** All 6 templates per decision D2: BamBam's Blueprint, Full Body × 3/week, Upper / Lower × 4/week, PPL × 3/week, PPL × 6/week, Bro Split, 5x5 Strength. Each has `id`, `name`, `emoji`, `description`, `cycleLengthLabel` (e.g. `"5-day"`, `"7-day"`), `previewEmojis[]` (uses the `'rest'` sentinel for rest-day slots), `workouts[]`, `rotation[]`. Namespaced workout ids (`fb_a`, `upper_a`, `ppl_push`, `bro_chest`, `x5_a` …) so templates can't collide if someone imports multiple. `loadTemplateForDraft(id)` returns a deep-cloned partial split shape ready to seed `splitDraft`. BamBam's workouts are drawn from the canonical `data/exercises.js` so the template stays in sync with any future tweaks.
235. **`RestDayChip` shared component (`src/components/RestDayChip.jsx`).** Renders per decision D3: dashed-circle border, muted "R" glyph. Size is tunable (default 24px); the R scales proportionally. Used by ChooseStartingPoint's template preview strip and available to Step 7's SplitCanvas. Full D3 application across Dashboard's weekly/monthly calendar strips is flagged as a follow-up mini-batch — this step only surfaces it where Step 6 needs it.
236. **`loadTemplate(templateId)` store action (`useStore.js`).** Thin wrapper around `loadTemplateForDraft`. Sets `splitDraft = { originalId: null, draft, updatedAt: Date.now() }` on success. Returns `true` if the template id resolved, `false` otherwise (the caller just shouldn't navigate in that case).
237. **`/splits/new/start` page + route (`src/pages/ChooseStartingPoint.jsx`, `App.jsx`).** Sticky header with back arrow, heading `"How do you want to start?"`, subtitle `"Pick a template and customize, or build from scratch."`. 7 template cards (6 from `SPLIT_TEMPLATES` + a Blank slate with dashed accent border + `✨` emoji). Each card shows the template's emoji, bold title, cycle badge, description, and a rotation preview row of 22px chips (workout emojis and `RestDayChip`s). Tapping a template calls `loadTemplate` → `navigate('/splits/new')`. Tapping Blank clears any existing draft and navigates to `/splits/new` with no banner. An inline file input at the bottom surfaces the Import path without having to route into SplitManager with a query string — accepts the same `bambam-split-export` shape as SplitManager's import. Route registered BEFORE `/splits/new` in App.jsx so React Router v6's specificity-first matching picks it up.
238. **`SplitManager` + `Welcome` wired through the chooser.** SplitManager's "Create New Split" button now routes to `/splits/new/start`. Welcome's Build-Own path (`pendingDestination === 'splits-new'`) also routes there, so new users see the scaffolding on their very first wizard visit instead of dropping cold into an empty form. Existing `/splits/new` URL stays live — direct-URL enthusiasts + any bookmarks still work.
239. **Verified live in preview** (mobile 375×812). Route renders all 7 cards (6 templates + Blank) with correct rotation preview (including dashed-R rest chips). Nav is hidden on `/splits/new/start` (the `/splits/new` prefix predicate from Batch 17b covers it for free). Tapping Full Body × 3/week templates seeds `splitDraft` with `{name: 'Full Body × 3/week', emoji: '🏋️', workouts: [FullBodyA/B/C], rotation: ['fb_a','rest','fb_b','rest','fb_c','rest','rest']}` and navigates to `/splits/new`. SplitBuilder's mount-effect surfaces the resume banner; tapping Resume restores all four fields into local state (name input + 🏋️ emoji selected + 3 workouts visible on Step 2). Blank path clears `splitDraft = null` and shows no banner. Back arrow returns to the prior page. No console errors. ✓

### Batch 17g (April 19, 2026) — SplitCanvas + WorkoutEditSheet (the big one)

Steps 7 + 8 of the Split Builder redesign (see `split-builder-redesign-handoff.md` Steps 7 + 8). These two ship together per the handoff's hard-dependency note — Canvas needs the sheet for the workout editor surface, and the sheet is meaningless without the Canvas to call it. Replaces SplitBuilder's 4-step linear wizard with a single canvas where identity, workouts, and rotation are always visible and always editable; saving is always one tap away via a sticky footer.

240. **`EmojiPicker.jsx` (`src/components/EmojiPicker.jsx`).** Shared emoji picker per decision D5: 32 curated emojis in a 6-col grid (Strength / Cardio / Focus / Other buckets interleaved), plus a "paste any emoji" text input at the bottom as the OS fallback. Renders via `createPortal` at z-index 270. Selected state uses ring-2. Per-tap onSelect fires immediately (no Save button — tap = pick). Accepts 1–8 char inputs for multi-codepoint ZWJ sequences.
241. **`SplitCanvas.jsx` (`src/pages/SplitCanvas.jsx`).** Single-page editor that owns identity + workouts + rotation + save-on-click.
    - **Identity row**: large 80×80 emoji button (opens EmojiPicker) + centered name input placeholder `"Untitled split"`.
    - **Workouts section**: collapsible header `Workouts (N)`, workout cards with left-side up/down arrows, emoji + name + preview text (top 3 exercise names + `"+K more"`), right-side ⋮ overflow menu (Edit / Duplicate / Delete), `"+ Add workout"` button that creates a stub and opens the sheet in `isNew` mode. Delete has undo-toast via Batch 17e's `showToast`; auto-prunes rotation references to the deleted workout's id. Duplicate uses the same deep-clone pattern Batch 17e ships — fresh workout id, `(Copy)` suffix, not auto-added to rotation.
    - **Rotation section**: collapsible header `Your Week` + `Cycle / Week` toggle (default Cycle per D6). Cycle view is a horizontal scrollable strip of 64×80 day chips (`D1` label + emoji or RestDayChip). Week view is a 7-col grid (Sun–Sat). Rotation chips open a bottom-sheet `RotationChipMenu` (z-250) to assign a workout or rest day, or remove the slot. Cycle view has inline ← → arrows to reorder; Week view cycles !== 7 show a "switch to Cycle view" hint.
    - **Sticky footer**: accent-colored Save button full-width. Create mode shows an "Activate this split on save" toggle (default on); edit mode omits it. Disabled state includes a helper hint listing missing fields (name / workouts / rotation).
    - **Back arrow**: if `isDirty`, opens `DiscardUnsavedModal` (z-280) with "Keep editing" / "Discard"; else navigates immediately. Save path calls `clearSplitDraft()` so the resume banner never fires on a split that was just saved.
    - **Overflow menu (edit mode only)**: Export (downloads a `bambam-split-export` JSON blob from current canvas state) + Delete split (native confirm → `deleteSplit` + clearDraft + navigate). Delete is hidden for built-in splits.
    - **Draft banner**: matches Batch 17a's pattern but pre-loads the draft into local state BEFORE showing the banner (the wizard required a Resume tap to commit; Canvas shows the draft immediately and the banner is an informational "Keep / Discard" chip). Orphaned and stale drafts auto-clear on mount per the 7-day rule.
242. **`WorkoutEditSheet.jsx` (`src/components/WorkoutEditSheet.jsx`).** Bottom-sheet editor for a single workout at z-270.
    - **Identity row**: 56×56 emoji button + workout name input.
    - **Section blocks**: collapsible, editable label input, up/down reorder buttons, × delete (with native confirm if non-empty). Each exercise row: up/down reorder, name, RecPill, × remove. `+ Add exercise` opens a nested ExercisePicker (z-275).
    - **ExercisePicker (inlined)**: full-screen overlay with search, muscle-group tabs, library-backed list, "Or add your own" bottom input with fuzzy-match dedup (reuses `findSimilarExercises` at 0.85 threshold) and CreateExerciseModal fallback. Structurally identical to SplitBuilder's picker — Step 10 will extract and improve both.
    - **Rec editor**: simple string-only editor for Step 7; structured REC (`{sets, reps, note}`) lands in Step 9. 20-char cap preserved. Saving with an empty value promotes the exercise back to a bare string to avoid noise in the rec data.
    - **Save / Close / Cancel**: Save enabled when name is non-empty. Cancel / Close / backdrop-tap → if dirty, opens a nested DiscardConfirm (z-280); else closes. Header + Save footer both pinned, middle content scrolls at up-to-92vh sheet height.
243. **`RestDayChip` reuse.** Used in the rotation strip (28px for cycle chips, 26px for week grid), in the RotationChipMenu's rest-day option, in the "Add rest" quick-add button (18px). Full D3 application to Dashboard's calendar stays in Batch 17f's follow-up slot.
244. **Route wiring (`App.jsx`).** `/splits/new` and `/splits/edit/:id` now render SplitCanvas. SplitBuilder.jsx remains in the repo as a component for the 72h stability window Step 12 calls for, but no route points at it.
245. **Verified live in preview** (mobile 375×812, debug-backup.json localStorage). `/splits/new` renders empty canvas with "New Split" heading, `Untitled split` placeholder, `WORKOUTS (0)` + "No workouts yet" empty state, YOUR WEEK section with Cycle default + Add rest quick chip, sticky footer with `Activate on save` toggle on + `Save & Activate` disabled with helper `Add a name · Add at least one workout · Add to rotation`. `/splits/edit/split_bam` loads BamBam's Blueprint with 💎 emoji + `BamBam's Blueprint` name + 5 workouts (Push, Quads, Pull, Shoulders & Arms, Hams — each showing top-3 exercise preview text) + 7 rotation chips. Tapping `+ Add workout` pushes a `New Workout` stub into the list AND opens WorkoutEditSheet with `Add workout` header and the stub's name pre-filled. Back arrow from a dirty state opens the DiscardUnsavedModal. No console errors.

### Batch 17h (April 19, 2026) — Structured REC rework

Step 9 of the Split Builder redesign (see `split-builder-redesign-handoff.md` Step 9, decision D7 — additive). Replaces free-text string RECs with the structured shape `{ sets?: number, reps?: string | number, note?: string }` while preserving full backwards compatibility with legacy strings. WorkoutEditSheet gets a proper sets/reps/note editor with live preview; BbLogger's in-session inline editor stays as a speed-first string input but displays via the shared formatter.

246. **`formatRec(rec)` in `helpers.js`.** Canonical display formatter that accepts any supported shape: `null` / `undefined` / `''` → `null`; legacy string → trimmed passthrough; `{ sets, reps, note }` → `"{sets}×{reps}"` + optional `" · {note}"` separator, with fallbacks for partial data (`"{N} sets"` when reps missing, `"{reps} reps"` when sets missing, bare `note` when both are empty, singular `"1 set"` for sets=1). Non-positive sets + non-numeric-string + non-object input all bounce to `null`. Node sanity-check run: 8/8 cases match spec.
247. **`RecPill.jsx` shared display component.** Renders `rec` via `formatRec` in all three call sites. Empty state: muted `📋 Rec` chip. Set state: filled blue `📋 {text}` pill with 10rem truncation. Sized via `size` prop (`sm` default, `xs` for tight rows). Replaces the in-file `RecPillButton` stub WorkoutEditSheet shipped with 17g and lives alongside BbLogger's existing inline pill (which gets `formatRec` integration but keeps its own inline-input editor path).
248. **`RecEditor.jsx` structured editor (bottom sheet).** Opens at z-275 above WorkoutEditSheet. Two-column sets/reps grid + full-width note row + live preview line using `formatRec(buildRec())`. Save returns the compact object (only fields the user filled in); Clear returns `null`. Normalizes input so legacy strings passed in as `current` pre-fill `note` per D7 — no auto-parsing of `"3x10"` style strings into structured form. Esc + backdrop-click dismiss.
249. **`WorkoutEditSheet` wired through the new editor.** The string-only inline stub from 17g is deleted. `editingRec` state simplifies to `{ sectionIdx, exIdx }` (no draft); the sheet reads the current rec from sections at render time and passes it into `RecEditor`. `updateExerciseRec` now accepts any shape `formatRec` understands — null clears (promotes exercise back to a bare string), anything non-empty passes through as-is. Old 20-char string cap dropped.
250. **BbLogger display wired through `formatRec` (`BbLogger.jsx`).** The inline rec pill in the exercise card header now renders `formatRec(exercise.rec) || 'Rec'` instead of raw string access — so structured values set via Canvas render correctly mid-session. Empty / pill styling still keyed on `formatRec` truthy-ness. The in-session edit path stays string-based for speed: when the user taps the pill, the inline input pre-fills with `typeof rec === 'string' ? rec : formatRec(rec) || ''` so they see a coherent starting point, and save continues to write a trimmed string via the existing `onUpdate({...exercise, rec: val})` path. BbLogger never writes the structured shape itself — that's a future surface.
251. **Backwards compatibility.** Per D7 there's no migration. Legacy strings stay strings in the store; new structured values get written side-by-side. `formatRec` + `RecPill` don't care which shape they're handed. `SplitBuilder`'s legacy `RecInline` component and auto-persist block still read/write strings verbatim — safe because `formatRec` handles both, and SplitBuilder isn't reachable via route after 17g.
252. **Verified** via `node -e` sanity against `formatRec` for all 8 edge cases (null / empty / legacy / full structured / partial / sets-only / reps-only / sets=0). Build passes (`npx vite build` — 717.52 KB bundle, +3 KB for RecEditor + RecPill). Canvas-surface verification deferred to the upcoming user test — editor flow is a straight text-input surface with live preview, very low regression risk.

### Batch 17i + 17j + 17k (April 19, 2026) — Shared ExercisePicker, predictive auto-fill + Skip-for-now, retire SplitBuilder

Steps 10 + 11 + 12 of the Split Builder redesign shipped together in a single worktree because they share a surface area (the exercise creation path) and Step 12 is a subtractive cleanup. No schema changes. No persist bump. End-to-end preview-verified before commit.

253. **`src/components/ExercisePicker.jsx` — shared picker extracted from WorkoutEditSheet's inline 17g version.** Pulled the ~220 line inline function out into its own module so future surfaces (a redesigned BbLogger Add panel, a library browser) can share the same implementation. Two improvements on top of the 17g behavior: (a) a new **"Recent in this split"** tab that front-loads library entries already used in sibling workouts — rendered only when `recentInSplit` has entries, and the initial selected tab when it does; (b) a **"Search all muscles"** checkbox (default ON) that controls whether typing in the search box ranges over the whole library or just the active tab's muscle-group slice. Empty-state copy branches: "No exercises from other workouts in this split yet." for the Recent tab, "No exercises match." everywhere else. z-index 275. Also exposes `onSkip` callback to `CreateExerciseModal` (Step 11, see below).

254. **`SplitCanvas.jsx` computes `recentInSplit` and threads it into `WorkoutEditSheet`.** New `useMemo` keyed on `(editingWorkoutId, workouts, exerciseLibrary)` walks every workout except the currently-editing one, collects each exercise's name, resolves against a pre-built `(normalized name → library id)` map covering canonical names + aliases (same pattern as the v3 migration and 16l's ExerciseLibraryManager workout filter), and de-duplicates by library id. Subscribes to `exerciseLibrary` from the store and imports `normalizeExerciseName` from helpers. `WorkoutEditSheet` accepts the new `recentInSplit = []` prop and passes it straight to `<ExercisePicker>`. For BamBam's Blueprint editing Push, the Recent tab surfaces ~15 entries from Quads / Pull / Shoulders & Arms / Hams — verified live.

255. **`WorkoutEditSheet.jsx` rewired to use the extracted picker.** Dropped the inlined `ExercisePicker` function (~220 lines) plus its now-unused imports (`useMemo`, `findSimilarExercises`, `EXERCISE_LIBRARY`, `MUSCLE_GROUPS`, `CreateExerciseModal`) — the shared component handles all of those internally. File shrank from 602 to ~447 lines. Public API adds `recentInSplit = []`; everything else unchanged.

256. **`predictExerciseMeta(name)` in `helpers.js` (Step 11).** Keyword-based best-effort guess that scans a 60-entry `EXERCISE_KEYWORD_MAP` in order and returns `{ primaryMuscles, equipment }` on the first match, or `null` otherwise. Map is ordered specific-first (e.g. "leg press" before bare "press", "lat pulldown" before bare "row"), so compound terms bind before the generic fallback rules. Covers chest / back / legs / shoulders / arms / core / glutes / calves / traps with common variants (RDL, pushup, pull-up, face pull, skull crusher, Russian twist, etc.). Sanity-checked against 15 representative inputs (12 positives + 3 negatives) — all pass.

257. **`CreateExerciseModal.jsx` — 300ms debounced auto-predict + Skip-for-now button.** New `useEffect([name, open, primaryMuscles.length, equipment])` runs `predictExerciseMeta` after a 300ms pause and fills the primaryMuscles + equipment chips if they're still empty. Two refs (`musclesTouched`, `equipmentTouched`) latch to true the first time the user taps any chip in that category — from that point on, auto-fill stops overriding. Refs reset when the modal reopens with a fresh name. New `onSkip` prop is optional: when provided, a subtle "Skip for now — tag later" button appears under the primary Save button and fires `onSkip({ name })`. Verified live: typing "Cable Face Pull" after the debounce auto-selects `Shoulders` + `Cable`.

258. **`ExercisePicker`'s `handleCreateSkip` path (Step 11).** When the user taps Skip in the modal, the picker calls `addExerciseToLibrary({ name, primaryMuscles: [], equipment: 'Other', defaultUnilateral: false, needsTagging: true })`, closes the modal, and calls `onAdd(name)` so the exercise appears in the workout immediately. The entry shows up in `/backfill` and the `Untagged` filter on `/exercises` for later tagging.

259. **`addExerciseToLibrary` validation relaxed for `needsTagging: true` (`useStore.js`).** Previously rejected empty `primaryMuscles` + missing `equipment` unconditionally per spec §3.2.1. Now accepts those shapes when `needsTagging === true`, producing the same `{primaryMuscles: [], equipment: 'Other', needsTagging: true}` shape the v3 migration writes for legacy sessions. Fully-tagged creates still go through the strict path — Skip is the only bypass.

260. **Step 12 — `src/pages/SplitBuilder.jsx` deleted.** The legacy 4-step wizard is retired. `App.jsx` drops the `import SplitBuilder from './pages/SplitBuilder'` line and rewrites the route comment to describe the post-retirement world. A handful of historical comment references to the word "SplitBuilder" in SplitCanvas / ChooseStartingPoint / WorkoutEditSheet / useStore are rewritten to point at SplitCanvas or "the legacy wizard" so `grep -R "SplitBuilder" src/` returns no hits — the strict Step 12 gate.

261. **Build + live verification.** `npx vite build --outDir /tmp/test-build` passes (722.97 KB bundle, +5 KB gzipped vs 17h — accounted for by ExercisePicker extraction + predictExerciseMeta map + Skip path). Preview pass on mobile 375×812 covered: (a) `/splits/edit/split_bam` → open Push workout → add exercise → Recent tab front-loaded with 15 sibling-workout entries; (b) All tab with search "leg" returns 10 hits across muscle groups; (c) flipping "Search all muscles" OFF on the Chest tab with "leg" search returns 0 as expected; (d) typing "Cable Face Pull" in the modal auto-selects Shoulders + Cable after the debounce; (e) Skip-for-now creates a `needsTagging: true` library entry with `equipment: 'Other'` and empty muscles; (f) back-navigation through `/splits` and `/dashboard` produces zero console errors. `grep -R "SplitBuilder" src/` returns nothing.

### Batch 18a (April 19, 2026) — Polish hotfix: exercise data loss + sheet scroll containment

Step 1 of the Split Builder polish pass (see `split-builder-polish-handoff.md` Part 1). The visual redesign pipeline (18b–18e) is blocked on this hotfix — users editing a workout mid-flight could lose exercises silently if 18b–18e landed first. Ships alone.

262. **`normalizeExerciseEntry(ex)` helper in `helpers.js`.** Lossless exercise-shape normalizer used by both `SplitCanvas.jsx` and `WorkoutEditSheet.jsx`. Accepts any legacy or current shape (string / `{name}` / `{name, rec}` / `{exercise}` legacy fallback / null / primitives) and returns the smallest renderable shape — bare string when no rec, `{name, rec}` when rec is non-empty, or `null` only for truly nameless entries. Dev-mode `console.warn` fires on any drop so a buggy migration can't hide behind silent filtering. 16/16 synthetic edge cases pass via `node -e`.

263. **Root-cause fix for the disappearing-exercise bug.** Batch 17g's `WorkoutEditSheet.jsx:33–45` + `SplitCanvas.jsx:54–66` used `.filter(Boolean)` on a ternary that returned `null` for any entry not matching `typeof ex === 'string' || ex?.name`. That silently dropped any legacy `{exercise: '…'}` shape, any `{name, rec: <weird shape>}` that tripped the Batch 17h `formatRec` migration, and any malformed result of an auto-persist-to-split write path in `BbLogger.jsx`. User report: "Flat Bench Press missing from BamBam's Blueprint → Push → Primary even though the Canvas preview says it's there" traces exactly to this drop path. Both call sites now delegate to `normalizeExerciseEntry` and preserve every recoverable entry.

264. **`migration-18a-sanity.mjs` at repo root.** Node ESM. Loads `debug-backup.json`, walks every split's workouts' sections' exercises, reports a per-shape histogram (158 entries: 128 strings + 30 `{name}` objects in the test backup — no malformed entries in the reference data), and asserts zero entries 18a drops that 17g preserved. Mirrors the existing `migration-sanity.mjs` / `migration-v3-sanity.mjs` / `anomaly-sanity.mjs` pattern.

265. **iOS scroll containment on WorkoutEditSheet.** Scroll region (`flex-1 overflow-y-auto`) gets inline `overscrollBehavior: 'contain'` + `WebkitOverflowScrolling: 'touch'` so momentum scroll doesn't leak into the Canvas behind. Backdrop gets `touch-none` to prevent gesture-on-backdrop from scrolling the page beneath. Drag handle (the decorative `w-10 h-1` bar) gets `pointer-events-none` on its wrapper so a stray swipe doesn't hijack touches on the area below. No pull-to-close implementation yet — that stays a future pass per the spec.

266. **Live preview verification.** Seeded BamBam's Blueprint → Push → Primary with 4 entries including a `{exercise: 'Legacy Decline Row'}` malformed shape that 17g would have dropped. All 4 entries render in the sheet (DOM positions captured at 290 / 328 / 366 / 404 px). Dev-mode `console.warn` is silent on clean data; fires correctly on synthetic nameless entries (`{rec: '3x10'}` and `{name: '', rec: '3x10'}` tests). Real-data backup shows 0 recovered entries (no malformation in the test dataset) and 0 regressions. Restored Primary to its canonical 3-entry state before commit.

267. **Build.** `npx vite build --outDir /tmp/test-build` → 723.17 KB bundle (+0.2 KB vs 17k, all in the helper). Gzipped size unchanged at 194.88 KB.

### Batch 18b (April 19, 2026) — SplitManager redesign

Step 2 of the Split Builder polish pass (see `split-manager-handoff.md` — authoritative; supersedes the 18b section of `split-builder-polish-handoff.md`). Redesigns `/splits` against the approved preview at `split-builder-polish-preview.html`. No store changes, no persist bump, no route changes — pure UI rewrite + 4 new pure helpers.

268. **`SplitCard` rewrite (`SplitManager.jsx`).** Single 38×38 brand emoji tile (was 3 emojis — tile + rotation preview row + per-workout pills). Removed the workout-name list on each card (was wrapping mid-phrase on typical names like "Push 2"). Title is 18px/700/-0.02em with `white-space: nowrap; text-overflow: ellipsis` so it truncates instead of wrapping. Meta line at 12.5px pluralizes ("1 session" / "N sessions") and uses tabular-nums. Card geometry pinned per the preview: `border-radius: 18px`, `padding: 16px 18px 14px 18px`, `margin-bottom: 10px`.

269. **Two-row status/lifespan block.** Row 1 — usage stat (left, always: `47 sessions · last today` / `Not yet used`) + status/CTA (right, always: `● Active` pill for the active card, `Set active ›` for all others). Row 2 — provenance (`Started March 22, 2026` when the split has sessions, `Created April 12, 2026` for never-used splits). Column scan works cleanly down the list: usage-always-left, status-or-CTA-always-right.

270. **Subtle active treatment.** Accent-gradient background wash (`linear-gradient(135deg, {accent}14 0%, {accent}00 55%)`) + 3px left accent bar (positioned `absolute top-3 bottom-3`, via rendered `<div>` since Tailwind can't render `::before` with a CSS variable color) + accent-rimmed 38×38 tile (`{accent}2e → {accent}0a` gradient, border `{accent}5c`) + accent-tinted border-top separator (`{accent}26`) + outer ring (`0 0 0 1px {accent}2e`). `ActivePill` uses Tailwind `animate-ping` for the dot's ring-expansion pulse (NOT `animate-pulse` — that's an opacity fade). Replaces Batch 17c's filled-accent card which shouted too loudly.

271. **Helpers added (`helpers.js`).** Four pure functions, no store coupling:
    - `getSplitSessionCount(sessions, split)` → bb-mode session count where `session.type` matches any `split.workouts[].id`.
    - `getSplitLastUsedDate(sessions, split)` → most-recent such session's ISO date, or null.
    - `formatRelativeDate(iso)` → `last today` / `yesterday` / `N days ago` / `last week` / `N weeks ago` / `N months ago` / `over a year ago`. Uses local-timezone day boundaries (Batch 16k pattern).
    - `formatStartDate(isoOrDateStr)` → `"March 22, 2026"`. Accepts both ISO timestamps and date-only strings (`'2026-03-22'`), parsing date-only as local midnight (`'…T00:00:00'`) to avoid rendering "March 21" in negative timezones.

272. **Topbar `+` promoted to filled accent button.** Was a transparent icon from Batch 17c; now a `bg-item` + `border-subtle` button with the glyph itself rendered in `theme.hex`. Now the sole split-creation entry point — the dashed `+ New split` bottom CTA is deleted.

273. **Redundant heading removed.** Dropped the large `<h2>My Splits</h2>` under the topbar (topbar already renders the title at 16px/600). The subtitle shrinks to `text-[11.5px]` + `whitespace-nowrap` so the whole message `"Your active split drives the rotation on the dashboard."` fits on one line at a 380px viewport. Persistent "built-in split tip" card from 17c also removed — list stays clean.

274. **Card tap → edit (was → activate).** Per the new spec (handoff section 6), tapping anywhere on the card body navigates to `/splits/edit/:id` (SplitCanvas). Activation now has its own explicit affordance — the right-side `Set active ›` button on inactive cards. Matches user intent: when you tap a split with its own CTA, you're more likely intending to dig in than to swap the active-split context.

275. **Overflow menu preserved.** Same `OverflowMenu` portal pattern from 17c (z-60, outside-click + Escape dismiss, viewport-edge clamp) + same 5 items (Set Active / Edit / Duplicate / Export / Delete) conditional on active-state + built-in-state. The ⋯ button itself is repositioned inside the new card's top-right but the menu behavior is unchanged. Duplicate still fires `showToast` with 5s undo via Batch 17e.

276. **Live preview verification** (mobile 375×812, debug-backup.json). Seven split cards render with correct brand tiles, titles, meta lines, usage stats, provenance dates. BamBam's Blueprint renders with `23 sessions · 4 days ago` + `Started March 22, 2026`. Brazil Body Plan (never used) renders `Not yet used` + `Created March 22, 2026`. Title truncation confirmed on `BamBam's Blueprint (C…)`. Scenarios pass: (a) tap inactive card → `/splits/edit/{id}`; (b) tap `Set active ›` → activates without navigating; (c) ⋯ on active built-in → `Edit / Duplicate / Export` (3 items); (d) ⋯ on inactive non-built-in → all 5 items; (e) topbar `+` → `/splits/new/start`; (f) accent swap to red → active pill + left bar + topbar `+` all turn red. No console errors.

277. **Build.** `npx vite build --outDir /tmp/test-build` → 727.36 KB bundle (+4.2 KB vs 18a, accounted for by the 4 helpers + the inline-styled active-card treatment). Gzipped 196.18 KB.

### Batch 18c (April 19, 2026) — SplitCanvas redesign

Step 3 of the Split Builder polish pass (see `split-builder-polish-handoff.md` Batch 18c). Reclaims above-the-fold real estate, removes the workout-card control stacking, guards the Save button from the red-accent collision, and preserves everything Batch 17g got right (single-canvas, draft-restore, always-visible rotation).

278. **Compact identity hero (`SplitCanvas.jsx`).** Emoji tile + name input move from 80×80 stacked-centered to 64×64 inline-horizontal. Tile gets a small pencil glyph overlay (-1, -1 bottom-right) as a tap-to-edit cue. `text-2xl` centered name becomes `text-xl` left-aligned. Saves ~80px of vertical real estate; still reads as the hero row without dominating the fold.

279. **WorkoutCard — compact row (`SplitCanvas.jsx`).** Up/down chevron column (24×48 of controls) removed entirely. Reorder now lives in the ⋯ menu (Move Up / Move Down, filtered out for first/last via null `onSelect`). A decorative `<DragHandle />` glyph on the left signals reorderability — the real drag-and-drop plumbing lands later. Card `p-3 → p-4`, border `rounded-xl → rounded-2xl`, emoji cell wrapped in a 40×40 `bg-item` tile, title bumped to `text-base`.

280. **Preview text simplified.** Second line was a first-3-names list like `Pec Dec · Incline DB Press · Flat Bench Press · +7 more` that wrapped/truncated inconsistently; now reads `13 exercises` (singular `1 exercise`). Pluralization and tabular-nums handled inline.

281. **`DragHandle.jsx` shared component (`src/components/DragHandle.jsx`).** 10×18 two-column dot SVG in `text-c-faint`, `pointer-events-none` so it doesn't steal taps from the surrounding button. 18e will re-export from the same path alongside `RowOverflowMenu` — no rename planned. WorkoutEditSheet picks it up in 18d for section headers and exercise rows.

282. **Save button never renders red (`SplitCanvas.jsx`).** If `settings.accentColor === 'red'`, a local `saveTheme = getTheme('emerald')` takes over for the Save button + activate-on-save toggle track. Accent-tinted hero tile, active-pill, and left-bar treatments on SplitManager (18b) deliberately keep red because they're thematic, not primary CTAs. Batch 18e extracts this into `getSaveTheme()` in `theme.js`.

283. **CYCLE/WEEK toggle own row.** SectionHeader's inline `extraRight` slot for `Your Week` is vacated; the `RotationViewToggle` now renders on its own right-aligned row below the header. Header gets its breathing room back and the toggle becomes a discoverable first-class control.

284. **"+ Assign" replaces `?` (`RotationWeekGrid` unassigned slot).** Empty week-grid cells previously rendered an ambiguous `?` glyph. Now render a `+` glyph with a `9px` "Assign" label under it — users know this cell is a target, not a mystery.

285. **Add-to-rotation labeled block.** The inline pill row below the rotation strip is wrapped in a labeled section: `ADD TO ROTATION` uppercase heading (`text-[11px] font-bold text-c-muted`) + pill row beneath. Also: rest-day pill copy `Add rest → Rest day` — the heading already says "add", no need to repeat it per pill.

286. **Activate-on-save toggle elevated to chrome card (`SplitCanvas.jsx`).** In create mode, the activate toggle was a subtle `text-c-secondary` switch row above the Save button. Now lifted into a shared `bg-card border-subtle rounded-2xl` card that wraps BOTH the toggle and the Save button, so activation reads as part of the save action rather than an afterthought. Toggle label simplified from `"Activate this split on save"` → `"Activate on save"`. Helper hint (`"Add a name · Add at least one workout · Add to rotation"`) still renders below the card when `!canSave`.

287. **Live preview verification** (mobile 375×812, debug-backup.json). Edit BamBam's Blueprint: compact hero renders with pencil glyph, 5 WorkoutCards show drag handle + 10×10 emoji tile + "N exercises" count + ⋯ (first card = `Edit/Duplicate/Move down/Delete`, middle = `Edit/Duplicate/Move up/Move down/Delete`, last = `Edit/Duplicate/Move up/Delete`). YOUR WEEK section header has no inline toggle; toggle renders on own right-aligned row. ADD TO ROTATION labeled block with pills. Red accent → Save button `bg-emerald-500` (`rgb(16,185,129)`). Create mode: Activate on save toggle + Save & Activate button share one chrome card. No console errors.

288. **Build.** `npx vite build --outDir /tmp/test-build` → 728.35 KB bundle (+1 KB vs 18b). Gzipped 196.59 KB.

### Batch 18d (April 19, 2026) — WorkoutEditSheet redesign

Step 4 of the Split Builder polish pass (see `split-builder-polish-handoff.md` Batch 18d). Halves the control count per row, folds reorder + delete into row-level ⋯ menus, and collapses each section card to a single visual surface. Keeps the full editability (section labels, emoji, rec editor, ExercisePicker) and existing z-stack unchanged.

289. **Compact SectionBlock header (`WorkoutEditSheet.jsx`).** Was five controls + input: `↑ / ↓ / label / ⌄ / ×`. Now: `DragHandle / label input / collapse chevron / ⋯`. The ⋯ opens a popover (`RowOverflowMenu` below) with Move up / Move down / Delete — Move items filter out at list boundaries via null `onSelect`. Inline × delete is gone.

290. **Compact exercise row.** Was `↑ / ↓ / name / RecPill / ×`. Now: `DragHandle / name-button / RecPill / ⋯`. Tapping the name button opens RecEditor (same target as tapping the RecPill itself), so the tap target is generous. ⋯ menu has Move up / Move down / Remove with boundary filtering. Row padding bumped from `py-1.5` to `py-2` for a roomier touch target now that the chevron column is gone.

291. **`RowOverflowMenu` inline component (`WorkoutEditSheet.jsx`).** Generic ⋯ popover used by both section headers and exercise rows. Outside-click + Escape dismiss, viewport-aware via `getBoundingClientRect`-less absolute positioning (fits within its parent row's flex layout), z-20 local. `items` prop filters out `{ onSelect: null }` so boundary Move items don't render at all. 18e promotes this to a shared `src/components/RowOverflowMenu.jsx` with the same API — no rename planned.

292. **One border per section card.** Outer `border border-subtle` + header's `border-b border-subtle` both removed. Section card is now `bg-card rounded-xl overflow-hidden`; the header's `bg-item` tint provides visual division between the header row and the body. The scrollable middle region gains a `bg-base` tint so each card reads against a subtly different background — stack reads clean without tripled border surfaces.

293. **"+ Add exercise" matches dashed-border CTA style.** Was `px-2 py-2 text-left` hover-only. Now a full-width dashed-border button matching "+ Add section" and "+ Add workout" across the app.

294. **Save button red-fallback (`WorkoutEditSheet.jsx`).** Same pattern as Batch 18c's SplitCanvas: `saveTheme = getTheme(accentColor === 'red' ? 'emerald' : accentColor)`. 18e extracts this to `getSaveTheme()` in `theme.js`.

295. **DragHandle reuse.** The shared `src/components/DragHandle.jsx` from 18c is imported here and used in section headers + exercise rows. Decorative only — the real drag-drop plumbing lands in a future batch.

296. **Live preview verification** (mobile 375×812, debug-backup.json). Edit BamBam's Blueprint → Push → the sheet renders:
    - Each section card shows `DragHandle / label / ⌄ / ⋯` only.
    - Each exercise row shows `DragHandle / name / Rec / ⋯`.
    - Pec Dec's ⋯ (first exercise, first section) → `Move down / Remove` (no Move up, no Delete at exercise level).
    - Choose 1's ⋯ (middle section) → `Move up / Move down / Delete`.
    - Tapping `Pec Dec` name opens RecEditor at z-275 (heading `Coach's prescription`).
    - "+ Add exercise" renders with dashed border matching "+ Add section".
    - Red accent → `Save workout` button `bg-emerald-500` (`rgb(16,185,129)`).
    - Console clean.

297. **Build.** `npx vite build --outDir /tmp/test-build` → 728.65 KB bundle (+0.3 KB vs 18c). Gzipped 196.77 KB.

### Batch 18e (April 19, 2026) — Shared split-builder primitives

Step 5 (final) of the Split Builder polish pass (see `split-builder-polish-handoff.md` Batch 18e). Pure refactor — extracts the reusable pieces Batches 18b–18d duplicated. No user-visible changes beyond a tighter bundle and the guarantee that any future Save button / ⋯ menu gets the same treatment for free.

298. **`src/components/RowOverflowMenu.jsx` shared component.** Consolidates three near-identical inline ⋯-menu implementations:
    - `WorkoutCardMenu` in `SplitCanvas.jsx` (~60 lines).
    - Section-header ⋯ popover in `WorkoutEditSheet.jsx` (~35 lines).
    - Exercise-row ⋯ popover in `WorkoutEditSheet.jsx` (same shape).
    All three now render via `<RowOverflowMenu items={…} ariaLabel={…} anchorClass={…} />`. Items with non-function `onSelect` are filtered out so boundary Move up/down items vanish cleanly (no more `isFirst ? null : …` null-guarding in every call site — the component handles it). Outside-click (mousedown + touchstart) + Escape dismiss, timer-deferred attach so the opening click doesn't immediately close. z-20 local (row-level positioning; page-level menus like SplitManager's still own their z-60 portal pattern).

299. **`getSaveTheme(accentColor)` helper in `theme.js`.** Replaces the inline `accentColor === 'red' ? 'emerald' : accentColor` branch that SplitCanvas (18c) and WorkoutEditSheet (18d) each duplicated. Both now call `getSaveTheme(settings.accentColor)` directly. Canvas's Save + activate-on-save toggle track and the sheet's Save workout button share one source of truth.

300. **Dedup results.** `WorkoutEditSheet.jsx` drops 75 lines (inline menu component + its `useRef` import). `SplitCanvas.jsx` drops 85 lines (`WorkoutCardMenu` component + its state/effect + the inline trigger button inside `WorkoutCard`). Net: `-160 source lines / +95 shared-primitive lines = -65 net`. Bundle 727.14 KB (-1.5 KB vs 18d). Gzipped 196.53 KB.

301. **Live preview verification** (mobile 375×812, debug-backup.json).
    - Canvas: tap Push ⋯ → `Edit / Duplicate / Move down / Delete` (first card, no Move up). Shared component behaves identically to the 18c-retired `WorkoutCardMenu`.
    - Sheet: open Push → Pec Dec ⋯ → `Move down / Remove` (first exercise). Section ⋯ menu renders with `Move up / Move down / Delete` on middle sections. Identical to 18d's inline menu.
    - Red accent: Canvas footer Save button `rgb(16,185,129)` (emerald). Sheet Save workout button `rgb(16,185,129)` (emerald). Both via the shared `getSaveTheme()`.
    - Console clean. No regressions.

302. **Shipped primitives inventory.** The Split Builder redesign now owns three shared primitives that future work can reuse without copy-paste:
    - `src/components/DragHandle.jsx` (18c) — decorative drag-handle glyph.
    - `src/components/RowOverflowMenu.jsx` (18e) — row-level ⋯ popover with null-filtered items + standard dismiss behaviors.
    - `src/theme.js` → `getSaveTheme()` (18e) — red→emerald safe-color routing for commit buttons.
    Also in the supporting library: `src/components/RestDayChip.jsx` (17f), `src/components/EmojiPicker.jsx` (17g), `src/components/Toast.jsx` (17e), `src/components/RecPill.jsx` + `RecEditor.jsx` (17h), `src/components/ExercisePicker.jsx` (17i). Polish loop complete.

### Batch 18f (April 19, 2026) — WorkoutEditSheet hotfixes (spacing + menu clipping + DragHandle removal)

Three live-feedback hotfixes on the sheet. User feedback: "you still can't see all of the exercises inside of each section … the only way to do that is to collapse another section", "when you click the three dots you also cannot view the options — it hides them depending on where you're clicking", and "I don't understand what the six pin even represents — makes me think I can hold and drag".

303. **`RowOverflowMenu` portals the panel to `document.body`** (`src/components/RowOverflowMenu.jsx`). Previously the panel was `absolute right-0 top-11` inside the row's `.relative.shrink-0` wrapper — which got clipped by each section card's `overflow-hidden` (needed for the rounded corners). Rewritten to `createPortal(panel, document.body)` with `position: fixed` + computed `top / left` from the button's `getBoundingClientRect()` at open. Viewport-edge clamp at 8px horizontally. **Auto-flip above the anchor** when the menu would overflow the viewport bottom (long menus near the bottom of the sheet stay on screen). z-index 285 — above WorkoutEditSheet (270) and RecEditor (275), below discard modal (280) and toast (290). Trigger button stays in the row layout so the row's flex alignment is preserved.

304. **Sections default to collapsed when they have ≥1 exercise** (`WorkoutEditSheet.jsx` `SectionBlock`). Previously all sections rendered expanded, which meant a workout with three full sections pushed "+ Add exercise" / rare exercises below the fold — and the only way to reach them was to manually collapse sibling sections first. Now `useState(() => (section.exercises || []).length > 0)`: pre-populated sections collapse on mount, empty sections stay expanded so the "+ Add exercise" CTA is immediately reachable in the new-workout flow. Tap the chevron (or the whole header visually) to expand.

305. **Exercise-count badge in section header** (`WorkoutEditSheet.jsx`). A small `11px text-c-muted tabular-nums` badge (`bg-card`, rounded 6px) renders between the label input and the collapse chevron whenever `exercises.length > 0`. Users see the section contents at a glance without expanding — covers the "how much is hidden?" question collapse introduces. The collapse-button `aria-label` is now dynamic: `"Expand section (3 exercises)"` when collapsed (singular `"1 exercise"` for count=1), plain `"Collapse section"` when open — screen-reader-friendly.

306. **DragHandle removed from WorkoutEditSheet + SplitCanvas WorkoutCard.** User: "I don't understand what the 6-pin represents … makes me think I can hold and drag." They're right — the component was decorative pre-reveal for drag-drop that wasn't wired. Reordering was always through the ⋯ menu's Move up / Move down. Since the handle suggested an interaction that didn't exist, it's removed from both surfaces. `src/components/DragHandle.jsx` stays on disk for when real DnD lands. Exercise row left-padding bumped `pl-2 → pl-3` to compensate for the handle's former visual indent.

307. **Live preview verification** (mobile 375×812, debug-backup.json). Open Push workout: all three sections collapse with badges `3 / 3 / 7` and aria-labels spelling out counts. No drag handles in sheet (0/0 SVG matches for the `10×18` glyph), none in Canvas WorkoutCards either. Tap Pec Dec's ⋯ → portal menu renders at `z:285`, top=353, bottom=443 (viewport 812) — fully visible, not clipped, items `Move down / Remove`. No console errors.

308. **Build.** `npx vite build --outDir /tmp/test-build` → 727.32 KB bundle (+0.2 KB vs 18e, accounted for by the portal + flip logic). Gzipped 196.74 KB.

### Batch 19 (April 19, 2026) — Equipment instance (AI coaching step 7)

Step 7 of the AI Coaching Recommender plan per spec §3.4. Per-session optional machine tag on each LoggedExercise plus a toolbar chip for picking from prior values or typing a custom name. Trend fits and anomaly detectors scope by instance when set, with an "instance history <3 sessions → fall back to unscoped" safety so switching machines never cold-starts the recommender into silence. Additive schema (no persist bump); Batch 16q's swing-detector prompt ("Same machine?") already primed users for this feature.

309. **`getExerciseHistory(sessions, id, name?, equipmentInstance?)`** (`helpers.js`). Optional fourth argument — when a non-empty string is passed, filters each session's matching LoggedExercise by case-insensitive `equipmentInstance` equality. Each history item also gains `equipmentInstance: string | null` so downstream detectors can inspect machine context. Empty / null / whitespace-only input short-circuits back to the pre-Batch-19 behavior (all sessions).

310. **`getInstancesForExercise(sessions, id, name?)`** (`helpers.js`). Returns distinct non-empty `equipmentInstance` strings for the given exercise, most-recent first, case-insensitively deduped while preserving the original casing of the first (newest) occurrence. Drives the picker's prior-values list.

311. **`EquipmentInstancePopover` (`BbLogger.jsx`).** Portal-rendered popover at z-220 (same layer as `PlateConfigPopover`, below RecommendationSheet). Shows the current value + historical instances as tappable pills, a free-text input that commits on Enter or via an Add button (40-char cap), and a Clear option when a value is set. The currently-set value is surfaced even if it's not yet in session history, so the picker always reflects state correctly. Outside-click + Escape dismiss. Viewport-edge clamp at left:8 so the 240px panel stays on screen even when the chip is far right on mobile.

312. **Machine chip in the toolbar row (`BbLogger.jsx`).** New cyan-tinted chip `[Machine {name}]` appended after the Tip chip. Empty-state is a muted dashed-border placeholder labeled `Machine`. The toolbar row is now `flex flex-wrap items-center gap-2` so the six chips gracefully wrap to a second line on mobile 375px when the combined width overflows (Plates/Uni/Last/PR fit row 1; Tip + Machine wrap to row 2). `max-w-[10rem]` + `truncate` on the chip prevents long machine names from blowing up the layout.

313. **Seed from last session + persist end-to-end (`BbLogger.jsx`).** `templateExercises` and the "Added" extras list both pick up `lastExDataByName[name]?.equipmentInstance` so a fresh session pre-fills from the most recent past tag. `buildExerciseData` writes a trimmed 40-char `equipmentInstance` onto the LoggedExercise only when non-empty (keeps the saved shape minimal for untagged exercises). `activeSession` serialization carries the field automatically through the existing exercise-object round-trip.

314. **Instance-scoped recommender history with fallback (`BbLogger.jsx`).** `recHistory` now reads the current exercise's `equipmentInstance` and, when set, scopes `getExerciseHistory` to matching machines. If the scoped history has fewer than 3 sessions AND the unscoped history has more, falls back to unscoped so first-time-on-this-machine doesn't cold-start the engine. Anomaly detection runs over the same `recHistory` — meaning a swap to a different machine produces scoped history whose prev+last are on the SAME machine once enough data accumulates, naturally suppressing the 16q swing detector's cross-machine false positives (the original spec §3.4 goal).

315. **`equipment-instance-sanity.mjs` (worktree root).** Node ESM — 31/31 pass. Validates (1) instance-filtered `getExerciseHistory` on synthetic data (case-insensitive matching, history-item instance echo, null/empty/whitespace short-circuit to unscoped); (2) `getInstancesForExercise` dedupe and whitespace trimming; (3) scoped vs unscoped histories producing divergent recommender prescriptions (6 Hoist sessions with clear uptrend + 6 Cybex sessions flat → Hoist rec ≥225 lbs, Cybex rec ≈290 lbs); (4) `debug-backup.json` baseline — 24 real sessions, 46 distinct exerciseIds, zero pre-19 `equipmentInstance` tags, pre-19 and post-19 call signatures return identical history. Mirrors the existing migration/recommender/fatigue/anomaly sanity script pattern.

316. **Verified live in preview** (mobile 375×812, debug-backup.json seeded). Scenarios pass: (a) Machine chip renders dashed-border placeholder when empty; (b) tap opens the popover at the correct fixed position with viewport-edge clamp; (c) typing "Hoist" + Add commits, chip flips to filled cyan state, `activeSession.exercises[Pec Dec].equipmentInstance === "Hoist"` in localStorage; (d) reload preserves the chip state via `activeSession`; (e) Clear button returns to dashed empty state and persists `""`; (f) with synthetic Hoist + Cybex prior sessions, a FRESH session seeds Pec Dec to Hoist (newest), picker options show `[Hoist (selected)] [Cybex] [Add] [Clear]` chronologically; (g) tap Cybex switches the chip without closing-then-reopening; (h) 6-chip toolbar gracefully wraps to row 2 on 375px — no horizontal overflow. Zero console errors across every scenario.

317. **Build.** `npx vite build --outDir /tmp/test-build` → 731.93 KB bundle (+4.6 KB vs 18f, accounted for by the popover + two helpers + instance-scoping branch in BbLogger). Gzipped 197.79 KB.

### Batch 19a (April 19, 2026) — Machine chip UX polish

Two live-feedback fixes on Batch 19's Machine chip.

318. **Chip label no longer repeats "Machine"** (`BbLogger.jsx`). The filled-state chip showed `Machine Hoist` — redundant once a value is set. Rewritten to render just the value (`Hoist`) when tagged; the `Machine` label is reserved for the empty dashed-border state where it serves as the field affordance.

319. **Chip visibility gated on library equipment type** (`BbLogger.jsx`). New `showMachineChip = libraryEntry?.equipment === 'Machine'` guard wraps the chip + popover. Exercises whose library entry is Barbell / Dumbbell / Bodyweight / Kettlebell / Cable / Other — or exercises with no resolvable library entry — hide the chip entirely so the toolbar doesn't waste space on a prompt that doesn't apply. The tagging work a user does in `/exercises` or `/backfill` now carries into the logger automatically: classify Pec Dec as Machine and the chip appears; classify Lateral DB Raises as Dumbbell and it stays out of the way. Cable was deliberately excluded this round — user explicitly called out Machine as the relevant class; can be added later if Cable Fly / Cable Row gym-to-gym variation becomes worth tracking.

320. **Verified live in preview** (mobile 375×812, debug-backup.json with synthetic Hoist seed). (a) Pec Dec (Machine) renders chip, pre-seeded to `Hoist` from prior session; chip text reads `Hoist` (not `Machine Hoist`). (b) Incline DB Press (Dumbbell) toolbar shows only `Plates / Uni / Last / Tip` — Machine chip absent. (c) Flat Bench Press (Barbell) and Dips (Bodyweight) also hide it. (d) Cable Fly (Cable) hides it too. (e) Any Plate-loaded Press (Machine, no prior tag) renders dashed-border empty-state `Machine` chip. No console errors.

321. **Build.** `npx vite build --outDir /tmp/test-build` → 732.11 KB bundle (+0.2 KB vs 19, all in the gating branch). Gzipped 197.88 KB.

### Batch 19b (April 19, 2026) — Rest timer: hide cog, long-press for settings

User feedback on the floating rest timer: the always-visible settings cog was clutter and the timer's `scale(1.5)` on running made it feel "uncontrollable" as the visual jumped into a different space than where it was anchored.

322. **Cog button removed** (`src/components/RestTimer.jsx`). The inline gear button to the left of the timer is gone. Settings are now accessible via long-press on the timer circle itself.

323. **1-second long-press opens the settings panel.** Pointer/touch/mouse down starts a 1000ms timer; release cancels. A >10px drag also cancels so the existing outer drag-to-reposition behavior still wins when the user is moving the timer. When the long-press fires, the settings panel toggles open and a 15ms haptic vibration confirms the hold (when supported). The subsequent browser-synthesized `click` after release is guarded with a `longPressFired` ref so the timer doesn't also start/stop.

324. **No text-selection / iOS callout on long-press.** Added `userSelect: none`, `WebkitUserSelect: none`, `WebkitTouchCallout: none`, `WebkitTapHighlightColor: transparent`, plus `onContextMenu: preventDefault` to the timer button. iOS won't show the copy/share callout menu; Android won't highlight the label text; desktop won't show the right-click context menu.

325. **Removed the `scale(1.5)` on running.** The timer no longer visually grows when active. It sits stably at the same position before / during / after the rest period, so the tap target never "moves." The progress ring + color change (idle→blue→amber→green) still signal state clearly without needing the scale.

326. **Settings panel accessible while running.** Dropped the `&& !isRunning` guard so long-press opens the duration editor mid-rest too (previously only visible when idle). Harmless — changing the stored duration doesn't retroactively alter an already-running countdown; it just applies to the next start.

327. **`aria-label` updated.** `"Start rest timer (long-press for settings)"` / `"Pause rest timer (long-press for settings)"` — screen readers announce both actions.

328. **Verified live in preview** (mobile 375×812). Cog button absent. Long-press (simulated touch start → 1100ms wait → touchend) opens the settings panel without starting the timer. Subsequent short-tap starts the timer. `user-select: none`, `transform: none`, `webkit-tap-highlight-color: rgba(0,0,0,0)` all confirmed via computed-style inspection. Screenshot shows running timer at fixed position with settings panel opened to its left, no overlap with session header or exercise cards. Zero console errors.

329. **Build.** `npx vite build --outDir /tmp/test-build` → same bundle size (tiny refactor, no new dependencies).

### Batch 19c (April 19, 2026) — Rest timer expansion restored + auto-start rest default on

Two quick follow-ups on 19b: user wanted the running-state scale back (it disambiguates resting vs idle at a glance), and wanted the auto-start-rest-after-a-working-set behavior on by default instead of gated behind a settings toggle.

330. **`scale(1.5)` on running restored** (`src/components/RestTimer.jsx`). Wrapped the button in a dedicated transform container with `transform-origin: top right` so the circle grows down+left into empty space rather than off-screen at top:70 / right:16. Transition stays at `0.3s ease`. All long-press + text-select guards from 19b remain on the underlying button, so the hold-to-open-settings flow still works while scaled.

331. **`autoStartRest` default flipped `false → true`** (`src/store/useStore.js`). Fresh installs now auto-start the rest timer when the user completes a working set (weight + reps both present) and moves to a new set via the numpad's Next-on-reps action or the `+ Add Set` button. The existing `addSet` guard in `BbLogger.jsx:819` (`settings.autoStartRest && lastSet?.type === 'working' && (lastSet.reps || lastSet.weight)`) is unchanged — just the default value. Users who want it off can still toggle via HamburgerMenu → Settings → Workout Defaults. **Note for existing users:** the persist merge preserves any value already saved to localStorage, so long-time users whose setting was explicitly set to false (or matched the old default) will keep false until they toggle it on once in settings.

332. **Verified live in preview** (mobile 375×812). Idle state: timer transform is `matrix(1,0,0,1,0,0)` (scale 1). Tap starts the timer → transform transitions to `matrix(1.5,0,0,1.5,0,0)` after the 0.3s ease. Auto-start path: seeded Pec Dec with weight:185 / reps:10 on set 0; tapped `+ Add Set` → `restEndTimestamp` flips from null to a future epoch ms (~90s out) immediately. Screenshot confirms the blue running timer at 1.5× scale in the upper-right, no cog, Pec Dec showing the filled 185×10 set with trophy + the new empty row below. Zero console errors.

333. **Build.** Same bundle size (one value flip + one transform wrapper — nothing measurable).

### Batch 20b (April 19, 2026) — Gym-scoped recommender + Machine chip gym seed + auto-tag prompt (AI coaching step 8 substep 3)

Second visible UI surface from Step 8, plus the engine wiring that Step 7 left on the table: the recommender and the Machine chip now both scope by the session's gymId, so training at different locations produces correctly-scoped prescriptions and the right prior-machine seed. New GymTagPrompt banner captures the §3.5.4 "Tag X as available at Y?" decision inline at the top of the expanded exercise card. All three pieces rely on the Batch 20 data layer.

352. **`recHistory` progressive-fallback gym + instance scoping (`BbLogger.jsx` ExerciseItem).** The useMemo that feeds the recommender + anomaly detectors + sparkline now tries four tiers in descending specificity: (1) `instance + gym` (most specific — same machine at same gym), (2) `gym alone` (same gym, any machine — covers intra-gym machine swaps), (3) `instance alone` (same machine at any gym — covers identically named machines across gyms), (4) unscoped (the pre-Batch-19 safety net). Each tier must clear the ≥3-session bar to be used; otherwise it falls to the next. Verified live: at TR with 3 Cybex sessions, the sheet reports "last 3 sessions" with Peak 293 and pushes 220×10; switching the SessionGymPill to VASA (4 Hoist sessions) re-renders "last 4 sessions", Peak 240, pushes 185×10 — same exercise, different prescription per gym.

353. **Gym-preferring `lastExDataByName` seed (`BbLogger.jsx`).** Two-pass build: first pass collects per-exercise seeds from same-gym sessions only, second pass fills any missing names from the broader history. Drives the Machine chip's initial value + the unilateral / plateLoaded init flags. Reads `seedGymId = savedSession?.gymId || settings.defaultGymId || null` — resumed sessions use the saved gym exactly, fresh starts fall back to `defaultGymId` (what the readiness overlay will default to), so the Machine chip seed reflects what the user is about to pick without the TDZ hazard of reading the yet-to-be-declared `gymId` useState. Verified live: fresh session at VASA (default gym) seeds Pec Dec to Hoist; fresh session at TR seeds it to Cybex.

354. **`getInstancesForExercise` gym-scoped with fallback (`BbLogger.jsx` ExerciseItem).** Machine picker options now come from `getInstancesForExercise(allSessions, id, name, currentGymId)`. Fallback to the unscoped list when the gym-scoped result is empty, so first-time-at-a-gym surfaces historical picks as a starting point rather than nothing.

355. **`dismissedGymPrompts` + `dismissGymPrompt(key, gymId)` store action (`useStore.js`).** Shape: `{ [`${exerciseId}:${gymId}`]: sessionId }`. Mirrors `dismissedAnomalies` — session-scoped, self-invalidates on next session's startTimestamp change. This is the persistence hook for the prompt's "Not this time" branch; "Always skip" persists via `Exercise.skipGymTagPrompt` instead (Batch 20).

356. **`GymTagPrompt` component (`src/pages/log/Recommendation.jsx`).** Accent-tinted banner with three action buttons: "Yes, tag it" (primary, accent-filled), "Not this time" (secondary, `bg-item`), "Always skip" (tertiary, muted). Rendered inline inside an expanded exercise card, above the AnomalyBanner so tagging decisions come first and correctly-scoped history reduces false-positive anomaly signals. Copy: `Tag {exerciseName} as available at {gymLabel}?`. Co-located with `AnomalyBanner` in `Recommendation.jsx` for style consistency.

357. **BbLogger wiring — 3 new props + 3 new store subscriptions.** ExerciseItem accepts `currentGymId` + `currentGymLabel` and threads them into recHistory, machineOptions, and the GymTagPrompt render gate. Also subscribes to `addExerciseGymTag` / `addSkipGymTagPrompt` / `dismissGymPrompt` so each of the three prompt actions fires the right store action. The prompt's render gate composes: `settings.enableAiCoaching && libraryEntry && currentGymId && shouldPromptGymTag(libraryEntry, currentGymId) && !dismissedThisSession`. Uses `libraryEntry.id` as the prompt key (not `exercise.exerciseId`) because template-seeded exercises don't carry exerciseId until save time.

358. **Verified live in preview** (mobile 375×812). Seven scenarios pass with zero new console errors after the initial seedGymId fix: (a) fresh session at VASA → Pec Dec seeds to Hoist; (b) fresh session at TR → seeds to Cybex; (c) GymTagPrompt renders in expanded Pec Dec card with "Tag Pec Dec as available at VASA?"; (d) Yes, tag it → `exerciseLibrary['ex_pec_dec'].sessionGymTags = ['gym_vasa']`, prompt disappears; (e) Not this time → `settings.dismissedGymPrompts['ex_pec_dec:gym_vasa']` stamped with activeSession.startTimestamp, prompt disappears, no tag added; (f) Always skip → `skipGymTagPrompt = ['gym_vasa']` on library entry (persistent), prompt disappears; (g) mid-session gym swap (TR → VASA via SessionGymPill) re-scopes the recommender live — sheet flipped from "last 3 sessions, Peak 293, Push 220×10" to "last 4 sessions, Peak 240, Push 185×10" in the same flow. Screenshot confirmed clean layout at VASA with purple accent.

359. **Build.** `npx vite build --outDir /tmp/test-build` → 743.18 KB bundle / 200.39 KB gzipped (+2.6 KB vs 20a, accounted for by the GymTagPrompt component + the progressive-fallback recHistory + gym-preferring seed pass).

### Batch 20c (April 19, 2026) — Picker gym filter (AI coaching step 8 substep 4)

Third visible UI surface for Step 8 per spec §9.7 (Option A) + §3.5.3. Adds an opt-in "Only available at {gym}" toggle to the session's Add Exercise panel. Hard filter, default OFF so nothing is hidden until the user opts in. Untagged library entries pass through as universally available per §3.5.3 ("empty or missing sessionGymTags = available everywhere"). Scoped to the in-session picker surface only — `ExercisePicker` (WorkoutEditSheet / SplitCanvas edit flow) is intentionally untouched because there is no gym context there. Adding the toggle to a surface without a gymId would be dead code.

360. **`isExerciseAvailableAtGym` import in BbLogger (`BbLogger.jsx`).** Appended to the existing multi-import from `../../utils/helpers`. No other consumers yet — ExercisePicker (the shared component) stays gym-agnostic.

361. **`AddExercisePanel` signature extended** (`BbLogger.jsx`). Two new props with defaults: `currentGymId = null`, `currentGymLabel = null`. Backwards compatible — any caller that doesn't thread them gets a null → toggle hidden → pre-Batch-20c behavior.

362. **Suggestions useMemo returns `{suggestions, hiddenCount}`** (`BbLogger.jsx`). Candidates are first built via the existing starter-12 / fuzzy-match logic (unchanged), then — only when toggle ON AND a gymId is set — filtered via `isExerciseAvailableAtGym`. `hiddenCount = candidates.length - filtered.length`. No-op when either gate is false, so the untouched path preserves the old perf/memory shape exactly.

363. **Toggle chip row below search input** (`BbLogger.jsx`). Rendered only when `currentGymId` is non-null so a gym-less session doesn't see a dead checkbox. Checkbox bound to `onlyThisGym` useState (default false per spec §9.7 Option A). Label copy: `Only available at {currentGymLabel || 'this gym'}` with the gym name wrapped in `<strong>` for emphasis. A right-aligned `{hiddenCount} hidden` counter appears only when toggle ON AND at least one candidate was filtered out — muted text + tabular-nums so counts jump without layout shift. `accentColor: theme.hex` inline style on the checkbox so it matches the user's palette.

364. **Create path ("+ Add [typed]") is NOT filtered** (`BbLogger.jsx`). Left the `{query.trim() && <button>+ Add "{query}"</button>}` branch outside the memo so tapping it continues to hit `handleAddTyped` regardless of filter state. The filter is a picker filter, not a create filter — new names never resolve against sessionGymTags.

365. **Render-site prop threading** (`BbLogger.jsx`). The single `<AddExercisePanel ... />` call site at the panels-and-modals block now passes `currentGymId={gymId}` + `currentGymLabel={currentGymLabel}`. Both values already existed on the parent scope from Batch 16n / 20b — no new computation needed.

366. **Verified live in preview** (mobile 375×812, fresh install seeded with VASA/TR gyms + Pec Dec tagged gym_vasa + Tricep Pushdown tagged gym_tr + Nordic Curl tagged gym_tr). Six scenarios pass with zero console errors: (a) no gym set → toggle hidden entirely (Add Exercise panel children are `H3 / INPUT / DIV / BUTTON` only, no `<label>`); (b) gym VASA set + toggle OFF → all 11 starter suggestions visible with `"Only available at VASA"` label + unchecked checkbox; (c) toggle ON with current tags → 10 results (Tricep Pushdown drops off) + `"1 hidden"` counter appears; (d) Pec Dec (tagged VASA) stays visible while Tricep Pushdown (tagged TR only) disappears — confirms §3.5.3 rule; (e) search "curl" + toggle ON → 7 of 8 results (Nordic Curl tagged TR drops off), `"1 hidden"` counter visible, `+ Add "curl"` create button still present above the suggestion list; (f) create path verified untouched — the query-driven `+ Add` button never filters. Screenshot confirms clean layout with the new label row sitting between the search input and the suggestion list.

367. **Build.** `npx vite build --outDir /tmp/test-build` → 744.15 KB bundle / 200.73 KB gzipped (+0.97 KB vs 20b, all in the filter branch + label JSX).

### Batch 20d (April 19, 2026) — Gym CRUD Settings UI + orphan cleanup (AI coaching step 8 complete, v1 DONE)

Final Step 8 substep. Closes out v1 of the AI Coaching Recommender per `coaching-recommender-spec-v3.pdf`. A collapsible "My Gyms" section lives under Settings → Profile Settings in `HamburgerMenu.jsx`. Before this shipped, gyms could only be created inline from the readiness overlay or the Batch 20a SessionGymPill — there was no way to rename a typo, promote a different gym to default without a visit to the logger, or delete a gym that stopped being relevant. The section renders the gym list with live session and tag counts, an inline rename path, a Set-default shortcut, and a confirm-guarded delete that strips orphan references from the exercise library so nothing points at a removed gymId.

368. **`renameGym(id, newLabel)` store action (`useStore.js`).** Case-insensitive dedupe against sibling gyms (case-preserving no-op when the label already matches — tapping Rename without changing anything doesn't spuriously reject). Returns `true` if the rename committed, `false` when rejected (missing id, empty label after trim, or collision). The `set(state => ...)` block reads the current list inside the updater, so back-to-back renames in the same render don't race against each other.

369. **`removeGym(id)` amended for orphan cleanup (`useStore.js`).** The existing action already dropped the gym from `settings.gyms` + reassigned `defaultGymId` to the first remaining gym when the deleted one was default. Now also walks `exerciseLibrary` and strips the removed gymId from every entry's `sessionGymTags` AND `skipGymTagPrompt` arrays. When the resulting array is empty, the field is deleted entirely (keeps the serialized shape minimal for entries that had a single tag matching the deleted gym). Historical `session.gymId` values are intentionally left untouched — they're a record of past truth, and rewriting them would silently change what exercises count under which gym when reviewed in History. No store split — a second `removeGymWithCleanup` action would have risked a caller doing the first without the second, and the cleanup is non-optional for correctness.

370. **HamburgerMenu "My Gyms" section (`HamburgerMenu.jsx`).** Collapsible heading under Profile Settings (starts collapsed — Batch 16r "less is more" pattern). Heading shows `My Gyms · N` with a chevron; tap toggles. When expanded:
    - Per-gym card with inline label + `Default` badge (accent-tinted) for the active gym.
    - Two compact info lines: `{sessionsHit} session{s}` left-aligned, `· {taggedHit} tagged` appended when nonzero.
    - Actions row (right-aligned): `Set default` (when not already default), `Rename`, `Delete`. Buttons are small text links at `text-xs font-semibold` so they fit cleanly at 375px — an earlier single-row layout had the Delete button clip past the viewport on gyms with all three actions visible, so the actions sit on their own line below the counts.
    - Inline `Rename` path: tapping swaps the label for an autofocused input; Enter/blur commits via `renameGym`, Escape reverts. Duplicate-label collision surfaces a native `alert()` so the user isn't left guessing why the rename silently didn't take.
    - `Delete` action builds a detail-rich confirm message: `Delete "{label}"? {label} has {N} past sessions and {M} tagged exercises. History stays, but exercise tags will be cleared.` Parts drop gracefully when either count is zero (pure-never-used gyms show just `Delete "{label}"?`).
    - `+ Add a gym…` form at the bottom — identical case-insensitive dedupe path as the Batch 16n readiness overlay + Batch 20a pill. 40-char cap on both Rename and Add inputs.

371. **Verified live in preview** (mobile 375×812, fresh install seeded with VASA + TR + Home Gym, defaultGymId=gym_vasa, Pec Dec tagged both gyms, Tricep Pushdown tagged TR only, Cable Fly with skipGymTagPrompt=['gym_home'], 3 synthetic sessions at VASA + 2 at TR + 0 at Home). Six scenarios pass with zero console errors:
    (a) My Gyms heading renders `My Gyms · 3` with chevron; tapping expands + chevron flips.
    (b) VASA row shows `3 sessions · 1 tagged` + Default badge; TR shows `2 sessions · 2 tagged` + Set default/Rename/Delete row; Home Gym shows `0 sessions` + Set default/Rename/Delete row.
    (c) Rename VASA → `VASA Orem` via inline input + Enter: store + UI both update to `VASA Orem`, editing mode exits.
    (d) Rename TR → `VASA Orem` (duplicate attempt): `renameGym` returns false, alert fires `"Another gym already uses that name."`, store stays `{VASA Orem, TR, Home Gym}`.
    (e) Tap Set default on TR: `defaultGymId` flips to gym_tr, Default badge moves to TR, VASA Orem row gains a Set-default button. Add new gym "Gold's Gym" via the inline form: store grows to 4 gyms, renderedLabels include `Gold's Gym`, header count bumps `My Gyms · 4`.
    (f) Delete Home Gym (stubbed confirm → true): row disappears, `Cable Fly.skipGymTagPrompt` is undefined after cleanup (empty array deleted), gym count back to 3. Delete VASA Orem (now not default — 3 past sessions + 1 tagged exercise): confirm message reads `Delete "VASA Orem"? VASA Orem has 3 past sessions and 1 tagged exercise. History stays, but exercise tags will be cleared.` After accept, `Pec Dec.sessionGymTags` flips from `['gym_vasa','gym_tr']` → `['gym_tr']`, Tricep Pushdown tags unchanged, 3 sessions with `gymId: 'gym_vasa'` still in the sessions array (historical truth preserved), remaining gyms `[TR, Gold's Gym]`, defaultGymId still `gym_tr`.

372. **Build.** `npx vite build --outDir /tmp/test-build` → 748.61 KB bundle / 202.11 KB gzipped (+4.46 KB vs 20c, all in the HamburgerMenu section + the renameGym action + the orphan-cleanup branch on removeGym).

**v1 DONE.** Every spec item from `coaching-recommender-spec-v3.pdf` that was in-scope for v1 has shipped: §2 (Layer 1 e1RM / Layer 2 %1RM / Layer 3 nudge / auto-deload), §2.5 readiness check-in, §3.2 exercise identity via exerciseLibrary, §3.3 fuzzy dedup, §3.4 equipment instance scoping, §3.5 gym tagging (data layer + session pill + recommender gym-scoping + auto-tag prompt + picker filter + CRUD Settings), §3.7 RPE plumbing (engine-side, UI reverted per user's failure-always pattern), §3.8 warmup/working auto-classify, §4 fatigue multipliers (grade × cardio × rest × gap), §4.5 anomaly detectors (plateau / regression / swing), §9.1/§9.2/§9.3/§9.4/§9.6/§9.7 all UI surfaces. Out-of-scope per plan Part 4: back-off sets, RPE re-introduction, detector threshold tuning, §8.x growth tracks.

### Batch 20a (April 19, 2026) — Session gym pill in BbLogger header (AI coaching step 8 substep 2)

First visible UI surface from Step 8. Makes the gym-driven AI inferences (Machine chip auto-fill from Batch 19, incoming recommender-history scoping in 20b) visible to the user per their UX principle: "make AI inferences visible — don't hide auto-behavior." Hard requirement surfaced in the Step 8 briefing.

346. **`SessionGymPill` component (`src/pages/log/SessionGymPill.jsx`).** Compact accent-tinted chip rendered below the workout title in BbLogger's sticky header. Two states: **set** — filled with `theme.hex` tint + border + color, building glyph + gym label + ▾; **empty** (gyms configured but session has no gymId) — dashed-border `bg-item` placeholder labeled "Set gym". **Hidden** (returns null) when `settings.gyms.length === 0` — nothing to pick from, don't pester users. Chip is aria-labeled `Change gym (currently {label})` / `Set gym` so screen readers announce its function.

347. **Portal popover at z-240.** Tapping the chip opens a 260px-wide panel rendered via `createPortal` to `document.body` with `position: fixed`. Anchored to `chipRef.current.getBoundingClientRect()` at open time, 6px below, viewport-clamped at 8px from the horizontal edges. Outside-click (mousedown + touchstart with 0ms setTimeout attach) + Escape dismiss. z-index 240 — below RecommendationSheet (250), above page content. Panel content: heading `Where are you lifting?`, gym list with ✓ on the selected one, `Add a new gym…` input (40-char cap) + Add button, and `Clear gym for this session` link-style button when a gym is set.

348. **Three store hooks + pass-through props.** Component subscribes to `settings.gyms`, `addGym`, `setDefaultGymId`. Receives `gymId`, `onChange(id | null)`, `theme` as props so the logger (or any future caller) owns the state. Selecting a gym calls `onChange(id)` + `setDefaultGymId(id)` so next session defaults to the same pick. Clear button calls `onChange(null)` without touching defaultGymId — session is explicitly gym-less for now, but next session still seeds to the last real pick. Add button auto-selects the newly-created gym and sets it as default.

349. **BbLogger wiring (`src/pages/log/BbLogger.jsx`).** New import + usage below the h1 title, gated on `sessionStarted` — pre-start the ReadinessCheckIn overlay already owns gym selection, so the pill would be redundant chrome behind the overlay. `setGymId` is the existing state setter from Batch 16n; the existing `saveActiveSession` useEffect already writes `gymId` into localStorage under `activeSession`, so persistence rides on existing plumbing. Pill rendered on its own row beneath the title (not inline) to avoid the floating RestTimer circle overlapping the chip in the top-right corner.

350. **Verified live in preview** (mobile 375×812, blue accent, fresh install). Eight scenarios pass: (a) no gyms configured → pill hidden, no chip in header; (b) two gyms + gymId='gym_vasa' → blue pill reading `🏢 VASA ▾` with 82×26px bounds, aria-label `Change gym (currently VASA)`; (c) tap pill → portal popover at top:134 / left:20 / 260×188, within the 375px viewport, contents `[VASA✓] [TR] [Add a new gym…] [Add] [Clear gym for this session]`; (d) tap TR in popover → pill flips to `TR`, `activeSession.gymId === 'gym_tr'`, `defaultGymId === 'gym_tr'`, popover auto-dismisses; (e) add-new flow: type "Home Gym" + Add → new gym in `settings.gyms`, gymId → new id, pill reads "Home Gym"; (f) Clear button → `activeSession.gymId = null`, defaultGymId preserved, pill reverts to dashed-border "Set gym" empty state with muted text; (g) reload mid-session preserves `gymId` via the existing `saveActiveSession` pipeline; (h) zero console errors across every scenario. Screenshot confirms chip sits cleanly below the workout title / "Resumed from saved session" line with no overlap with the scaled-up RestTimer circle.

351. **Build.** `npx vite build --outDir /tmp/test-build` → 740.63 KB bundle / 199.72 KB gzipped (+4.3 KB vs Batch 20, all in the new component + the handful of wiring lines in BbLogger).

### Batch 20 (April 19, 2026) — Gym tagging data layer (AI coaching step 8 substep 1)

First substep of Step 8 — the final v1 milestone. Spec §3.5 + §9.7. Additive data layer only; no UI surface lands yet. Paves the way for 20a (session gym pill in BbLogger header), 20b (Machine chip gym-scoped auto-fill + auto-tag-on-use prompt), 20c (picker filter), 20d (Settings UI for gym CRUD). `settings.gyms` + `session.gymId` already exist from Batch 16n; this batch adds the per-Exercise axis the spec calls for (§3.5.1 Level 1) plus the §3.5.6 history-scoping variants.

339. **Two new optional fields on Exercise records (`useStore.js`).** `sessionGymTags?: string[]` (§3.5.1 Level 1 — gymIds where this exercise IS available) and `skipGymTagPrompt?: string[]` (§3.5.4 — gymIds where the user tapped "Always skip" so the auto-tag prompt stays quiet). Both stored as arrays of gymIds (not labels), matching how session.gymId already works, so renaming a gym via Settings won't orphan tags. Empty or missing means "universally available / unspecified" per §3.5.3. The `addExerciseToLibrary` passthrough already carried `sessionGymTags` (Batch 17j left a ghost hook in place); mirrors it for `skipGymTagPrompt` and fixes both to survive only when non-empty so new exercises stay minimal.

340. **Four store actions for the two fields (`useStore.js`).** `addExerciseGymTag(exerciseId, gymId)` (dedup by gymId), `removeExerciseGymTag(exerciseId, gymId)` (drops the field entirely when the array empties out so the shape stays minimal), `addSkipGymTagPrompt(exerciseId, gymId)`, `removeSkipGymTagPrompt(exerciseId, gymId)` (symmetric — lets a user undo an Always-skip decision in Settings). All are no-ops on missing/invalid inputs so the call sites in 20b–20d don't need defensive guards.

341. **Three pure gym-tag helpers (`helpers.js`).** `isExerciseAvailableAtGym(exercise, gymId)` — returns true when gymId is null, when sessionGymTags is empty/missing, or when the array includes gymId (§3.5.3 "available here" rule with the empty-means-universal fallback). `shouldSkipGymTagPrompt(exercise, gymId)` — pure read of the skip list. `shouldPromptGymTag(exercise, gymId)` — composed boolean that drives 20b's prompt: fires when a gym is set, not already tagged, and not opted-out. All three defensive against null/undefined/non-array inputs so the picker filter (20c) can feed them raw library rows.

342. **`getExerciseHistory` accepts `gymId` as 5th arg (`helpers.js`).** Adds to the existing `equipmentInstance` 4th-arg pattern. When a non-empty gymId is passed, only sessions whose `session.gymId` matches contribute. Sessions without a gymId (pre-16n data, or skipped-readiness sessions) do NOT match a scoped query per §3.5.6 — "unspecified" stays unspecified. Instance + gym compose with AND so "Hoist @ VASA" restricts correctly when the user has the same machine name at two gyms for distinct models. Each history item now echoes `gymId` alongside the existing `equipmentInstance` echo. Callers do the <3-session fallback themselves (same pattern Batch 19 established for machines); the helper stays pure.

343. **`getInstancesForExercise` accepts `gymId` as 4th arg (`helpers.js`).** Drives 20b's Machine-chip picker: the prior-instances list at VASA shows only instances used AT VASA, at TR shows only TR instances. Unscoped behavior unchanged.

344. **`gym-tags-sanity.mjs` at worktree root.** 49/49 pass. Covers: (1) `isExerciseAvailableAtGym` across 9 cases (no gymId, null/undefined exercise, missing/empty/non-array sessionGymTags, matching tag, multiple tags, tagged-elsewhere-only); (2) `shouldSkipGymTagPrompt` 6 cases; (3) `shouldPromptGymTag` composition 8 cases (the auto-tag decision table); (4) `getExerciseHistory` gymId scoping against 6 synthetic sessions across 2 gyms + 1 legacy unspecified — confirms history-item gymId echo, exact-match filter, legacy-session exclusion from scoped queries, empty-string / whitespace-only gymId short-circuits to unscoped; (5) `getInstancesForExercise` gym scoping; (6) composed instance+gym AND intersection including case-insensitive instance match; (7) `debug-backup.json` baseline — 0 real sessions have a gymId yet, unscoped calls match pre-Batch-20 behavior exactly, any scoped call returns empty. Mirrors the existing `equipment-instance-sanity.mjs` / `fatigue-sanity.mjs` / `anomaly-sanity.mjs` / `readiness-sanity.mjs` / `migration-*` patterns.

345. **No UI surface this batch, no preview verification.** Change is engine-only — nothing renders differently in the browser. Batch 20a wires the session gym pill into the BbLogger header as the first visible surface; that's when preview walkthrough returns. Build passes (`npx vite build --outDir /tmp/test-build` → 736.31 KB / gzipped 198.80 KB, +4.2 KB vs 19d).

### Batch 19d (April 19, 2026) — WorkoutEditSheet: Change Section + scroll fix + header spacing

Three targeted fixes in the workout editor sheet after live-feedback on Batch 18f.

334. **"Change section" action** (`src/components/WorkoutEditSheet.jsx`). New item in each exercise row's ⋯ menu, ordered `Move up / Move down / Change section / Remove`. Only renders when the workout has more than one section. Tapping it opens a new `ChangeSectionModal` (z-278, above the sheet at 270 and RecEditor at 275, below DiscardConfirm at 280). The modal lists every section as a selectable pill — the current section is disabled with a `current` badge, others show their exercise count. User picks a target, taps Confirm; the exercise moves to the END of the target section. New `moveExerciseToSection(fromSectionIdx, exIdx, toSectionIdx)` helper powers the commit.

335. **Sheet scroll on long sections** (same file). Two bugs stacked. First, the scrollable flex child was missing `min-h-0` — the classic flex-overflow gotcha where flex-1 refuses to shrink below its content's natural height, so `overflow-y-auto` never engages. Second, the SectionBlock outer div needed `shrink-0` — without it, the flex column was compressing each section card, and the card's own `overflow-hidden` (needed for rounded corners) then clipped the content instead of letting it bubble up to the scroll region. Both together: long sections now properly scroll inside the sheet. Verified at 375×500 with If You Have Time expanded (content 780px, scroll region 229px, scrollTop=150 succeeds).

336. **Collapsed section header spacing** (same file). The old layout had the count badge floating between the flex-1 input and the right-hand controls, visually detached from the label. Restructured: label input + count are now wrapped in a single `<label>` element (with `cursor-text` so tapping anywhere in the label area focuses the input), count renders as a subtle `· 3` middot suffix in `text-c-muted` hugging the label text. Chevron + ⋯ cluster on the right with `gap-1.5`. Row padding bumped `py-2.5 → py-3` for breathing room. Chevron shrunk `w-8 → w-7` to free a few pixels for longer labels. Collapsed rows now read cleanly as `Primary · 3` with trailing `⌄ ⋮`, not `Primary _______ [3] > ⋮`.

337. **Verified live in preview** (mobile 375×812 + 375×500 for overflow). Pec Dec's ⋯ menu on Primary (first row) shows `Move down / Change section / Remove` — no Move up, no Delete at exercise level. Change section → modal opens with Primary disabled (`current` badge) and Choose 1 / If You Have Time as options with their counts. Pick If You Have Time → Confirm → Pec Dec moves to the end of If You Have Time (count 7 → 8), Primary drops to 2 exercises, modal dismisses, sheet still open. Scroll check at 375×500: `scrollHeight: 780`, `clientHeight: 229`, `scrollable: true`, `scrollTop=150` sets cleanly. Collapsed headers render `Primary · 3` / `Choose 1 · 3` / `If You Have Time · 7` with inline middot counts. Zero console errors.

338. **Build.** `npx vite build --outDir /tmp/test-build` → tiny bump for the new modal component + moveExerciseToSection helper.

### Batch 21 (April 21, 2026) — Revert auto-classify of set type (spec §3.8 retracted)

User report: with `Default first set = Working`, set 1 correctly opens as "Work" but subsequent sets silently appear as "Warm". The setting read path (`firstSetType` at `BbLogger.jsx:784`) and `addSet` logic are correct — set 2 is born with `type: 'working'`. Root cause was the auto-classify block shipped in Batch 16p (#195, spec §3.8), which silently reclassified any non-drop set's type on weight entry: `<60% of recent top e1RM → warmup`, `>80% → working`, middle band kept the current type. Spec §3.8 is retracted after user review.

Rationale for full removal (both branches, not just the demotion):

1. **Warmups are temporal, not weight-based.** A lower-weight set following a working set is a back-off, AMRAP finisher, or deload — never a warmup. Warmups come first. The auto-classify judged each weight entry in isolation and ignored training structure.
2. **It poisoned the recommender.** `getExerciseHistory` in `helpers.js` deliberately excludes warmups when computing the per-session top set. A set that should have been 'working' but got silently demoted to 'warmup' was then silently removed from the recommender's e1RM history, skewing the trend fit, the confidence %, and the prescription.
3. **Silent override violates the "make AI inferences visible" principle.** Users got no toast, no highlight, no undo affordance.

373. **Removed auto-classify block entirely** (`src/pages/log/BbLogger.jsx`, previously at lines 839–852). Both directions — `<60% → warmup` demotion AND `>80% → working` promotion. `updateSet` now mutates only the drop-set branch; a set's type only ever changes via explicit Warm/Work/Drop tap or the first-set `firstSetType` seed at creation time.
374. **Removed `recHistoryRef` + its assignment** (`src/pages/log/BbLogger.jsx`). The ref was introduced in 16p solely to let `updateSet`'s closure read the latest `recHistory` without a TDZ error. With the auto-classify block gone, nothing consumed the ref. Three call sites deleted (declaration at `:791`, the auto-classify block, and the assignment at `:970`). `recHistory` itself still powers the RecommendationSheet / AnomalyBanner / sparkline paths unchanged — only the closure-read pathway is gone.
375. **No schema, migration, or settings changes.** Zustand persist version stays at 4. `settings.defaultFirstSetType` keeps its first-set-only semantic (the setting's name accurately reflects its scope — `addSet` at `:808–827` hardcodes `'working'` for set 2+). Historical sessions' saved `type` values are untouched; only new weight entries no longer silently reclassify.
376. **Build.** `npx vite build --outDir /tmp/test-build` → 748.25 KB bundle / 201.99 KB gzipped (−0.36 KB vs 20d, all in the deleted block + ref + comments).

Post-v1 roadmap follow-up per user: **drop-set bundling.** User observed that drop sets should be modeled as continuations of the preceding working set, not standalone rows. A working set at 185×10 followed by two drops to 135×8 and 95×6 is conceptually ONE logical set with two drop stages, not three. Significant data-model change affecting storage shape, BbLogger UI, volume / PR calculations (`getExercisePRs`, `isSetPR`, `perSideLoad`), history display, session comparison, share card, recommender input (`getExerciseHistory` top-set logic), and a new persist migration. Deserves its own plan pass — added to the Post-v1 roadmap below.

### Batch 22 (April 21, 2026) — Drop-set bundling data layer + v5 migration

First of three batches implementing drop-set bundling (see `drop-set-bundling.md` plan). Data layer only — no UI changes yet. Under the new model, a working set at 185×10 followed by drops to 135×8 and 95×6 stores as ONE top-level `{type:'working', weight:185, reps:10, drops:[{weight:135, reps:8}, {weight:95, reps:6}]}` entry, not three separate `sets[]` rows. Batches 23 + 24 land in the same feature branch and ship together so users never see intermediate-state inconsistency.

User-locked decisions for this change:
1. Set count = 1 per bundled group (drops don't inflate set counts).
2. Drops contribute to volume (recomputed from the nested shape).
3. PRs key off the working primary only — drop stages never qualify as PRs.
4. "+ Drop stage" button lives only inside an existing working-set card (no orphan drops via UI).
5. Retype-with-drops blocked — the Warm/Work/Drop cycler disables when `drops.length > 0`.

377. **`migrateSessionsToV5(sessions)` in `helpers.js`.** Two-pass migration. Pass 1 walks each exercise's flat `sets[]` left-to-right: working sets push to output and become the new parent; `type:'drop'` entries have their `type` + `isNewPR` fields stripped and attach to the preceding working's `drops[]` array; warmups push unchanged and reset the parent pointer (drops after a warmup start a new group and get promoted). Orphan drops (no preceding working in the same exercise) are **promoted** to `type:'working'` and become their own parent. Pass 2 recomputes `isNewPR` chronologically keyed by `exerciseId || name`, walking **only** working primaries under the new decision 3. Warmups with stale `isNewPR` from pre-v3 chronological walks get cleared. Idempotent: re-running on bundled data is a no-op because the walk only matches `type:'drop'` entries at the top level (bundled drops are nested and invisible to the walk).

378. **`calcSessionVolume` updated (`helpers.js`).** Walks `set.weight * set.reps` for the primary, then adds `Σ drop.weight * drop.reps` across `set.drops[]` when present. For pre-bundled / pre-Batch-22 data with no `drops` field, the inner reducer returns 0, so volume totals match the flat-shape calc exactly (decision 2 verified by sanity script).

379. **`getExercisePRs` skips non-working sets (`helpers.js`).** New `if (set.type !== 'working') return` guard early-exits warmups and (under the bundled shape) never enters drop stages anyway since they're nested. Matches decision 3 — PRs are working-primary-only.

380. **`getExerciseHistory` iterates working sets only (`helpers.js`).** The pre-Batch-22 `type === 'working' || type === 'drop'` filter is now just `type === 'working'`. Drop stages live inside `set.drops[]` under the bundled shape and never compete for top-e1RM. Under the new model, the recommender's trend fits on strength data only; drops are fatigue work and correctly excluded.

381. **Persist version bumped `4 → 5` (`useStore.js`).** New migrate block calls `migrateSessionsToV5(persistedState.sessions)` when `version < 5`. Chain runs v1→v2→v3→v4→v5 for pre-v5 stores; no-op on fresh installs (default version is 5). `importData` also runs the v5 migration on imported sessions so pre-v5 backups land in the current schema.

382. **`migration-v5-sanity.mjs` at worktree root.** 10/10 pass against `debug-backup.json` (24 sessions, 353 working sets, 31 drop entries). Reports: (a) all 31 flat drops bundled into 21 parent working sets; (b) 0 top-level drops remain; (c) 0 orphan promotions (user's real data has no drop-first sequences); (d) aggregate volume preserved to the lb-rep (471,755 before and after — decision 2 verified); (e) total `isNewPR` flags drop 134 → 110, representing 24 pre-v3 warmup PR flags being correctly cleared under the stricter working-only rule; (f) idempotency proven via re-run; (g) synthetic 5-set test case (`drop → drop → warmup → working → drop`) correctly produces 3 top-level entries: one promoted working (bundling the orphan drop chain), one warmup, one working with one drop nested. Mirrors the existing `migration-sanity.mjs` / `migration-v3-sanity.mjs` / `migration-18a-sanity.mjs` / `readiness-sanity.mjs` / `anomaly-sanity.mjs` / `gym-tags-sanity.mjs` pattern.

383. **No UI changes this batch.** BbLogger still renders flat rows, ghost rows stay flat, History/ShareCard/Progress still show flat drops. Sessions saved post-Batch-22 through the pre-update BbLogger would continue to write flat drops until Batch 23's buildExerciseData rewrite. This is why Batches 22 + 23 + 24 ship as one merge — the intermediate state where persisted data is bundled but new writes are flat would create visible drift. Feature branch `claude/drop-set-bundling` holds all three commits for a single fast-forward merge.

384. **Build.** `npx vite build --outDir /tmp/test-build` → 749.82 KB bundle / 202.36 KB gzipped (+1.57 KB / +0.25 KB vs Batch 21 — all in the new migration function).

### Batch 23 (April 21, 2026) — BbLogger bundled-set rendering + handlers

Second of three drop-set-bundling batches. The session logger now renders the bundled data shape end-to-end: each top-level `warmup` or `working` set is the primary row, and any working set's `drops[]` renders as compact orange-tinted indented stages beneath, with a "+ Drop stage" CTA that creates a new drop inside the parent. Still in feature branch `claude/drop-set-bundling` alongside Batch 22 + the upcoming Batch 24 — the three commit as one merge to main so users never see a mid-refactor state.

385. **Row-level type cycler drops the 'drop' option** (`BbLogger.jsx` `SET_TYPES` + `SetTypeBtn`). The cycler now steps warmup ↔ working only. Drop is no longer a type the user can produce via the chip — it's an action (+ Drop stage), not a state. Legacy `type:'drop'` values still render defensively as orange "Drop" chips and cycle back to working so a user can escape the state, but post-v5-migration no top-level set has that type anyway. New `disabled` prop on `SetTypeBtn` greys the button with a "Remove drop stages to change type" tooltip (decision 5 — retype-with-drops is blocked).

386. **`DropStageRow` component (`BbLogger.jsx`, below `SetRow`).** Compact weight/reps row for each drop stage nested inside a working's `drops[]`. "↳ Drop" label on the left (orange-tinted, same `w-14` as `SetTypeBtn` so column widths align with the primary). Weight input (`w-20`) + reps input (`w-16`) + × remove button. No type cycler, no PR trophy (decision 3), no plate-mode path (decision — drops are direct-weight entries even when the parent is plate-loaded). Numpad integration uses `weight-drop-{ex}-{i}-{j}` / `reps-drop-{ex}-{i}-{j}` field keys so focus state doesn't collide with primary row fields.

387. **Drop-stage handlers (`BbLogger.jsx`).** Three new actions on `ExerciseItem`:
    - `addDropStage(parentIdx)` — appends `{reps:'', weight:''}` to `parent.drops[]`. No-op unless parent is a working set.
    - `updateDropStage(parentIdx, dropIdx, newDrop)` — strips any `type` / `isNewPR` / `plates` / `barWeight` / `plateMultiplier` fields the caller sends (defensive — drops don't carry those).
    - `deleteDropStage(parentIdx, dropIdx)` — removes the stage; if the parent's `drops[]` empties out, the field is deleted entirely so the serialized shape stays minimal.
    `deleteSet` also gets a native-confirm guard: deleting a working primary with ≥1 drop stage prompts `"Delete this set and its N drop stage(s)?"` to prevent silent data loss. Warmups and empty workings delete silently.

388. **`addSet` semantics cleaned up** (`BbLogger.jsx`). The pre-Batch-23 branch `lastSet?.type === 'drop' ? 'drop' : …` is gone — `addSet` now creates only warmup or working primaries, never drops. If the previous set was a warmup, the new set is also a warmup (user is still in their warmup sequence); otherwise it's working. Maintains the existing "first set honors `settings.defaultFirstSetType`" behavior from Batch 1.

389. **`updateSet` simplified.** The drop-set plate-clearing branch (previously `if (newSet.type === 'drop' && oldSet.type !== 'drop')`) is removed. Under the bundled model, you never cycle into 'drop' via the type button (drops come from the "+ Drop stage" CTA, which creates a plateless nested entry directly). Clean straight-through updater now.

390. **Exercise set rendering loop rewritten** (`BbLogger.jsx` ExerciseItem). Replaces the flat `.map(set, i)` with a per-set block that renders:
    - Primary row via `SetRow` / `PlateSetRow` (for warmup or working).
    - If the primary is working + has `drops[]`: an indented block (`ml-3 pl-3 border-l-2 border-orange-500/40`) of `DropStageRow`s beneath.
    - If the primary is working AND has weight + reps filled: a dashed-border `+ Drop stage` button (indented to match the drop block).
    The CTA gate prevents orphan drops attached to blank parents (decision 4). `cyclerDisabled` prop is threaded into `SetRow` / `PlateSetRow` when the working has drops, so the Warm/Work chip can't flip while drops are attached (decision 5).

391. **`buildExerciseData` emits the bundled shape** (`BbLogger.jsx`). Serializes each top-level set with `type` / `reps` / `weight` / `rawWeight` / `isNewPR` as before, plus a new `drops: [...]` array when the working has nested stages. Each saved drop stage carries `reps` + `weight` + `rawWeight` (unilateral exercises correctly double the per-side drop load too, matching primary behavior) — but NO `type`, NO `isNewPR`, NO `plates`. `isNewPR` on primaries is gated to `type === 'working'` only (decision 3); warmups and any legacy drops never light up as PRs.

392. **Aggregation updates inside BbLogger.**
    - Session-comparison per-exercise volume (`exerciseVolume`) descends into `set.drops[]` alongside the primary (decision 2).
    - Share card `totalVolume` walks primary + nested drops.
    - Share card `totalSets` filters to `type === 'working'` (decision 1).
    - Finish-modal `loggedSets` counter tracks only filled working primaries.
    - `hasPR` / per-set trophy guards (`SetRow`, `PlateSetRow`) require `type === 'working'` before calling `isSetPR` (decision 3) — previously a warmup with a heavy weight could trip the trophy on first-ever logs (`maxWeight===0 → return true` branch in `isSetPR`).

393. **Ghost rows still render flat.** `PrevSetRow` in BbLogger (the "Last Time" block) and History / Share Card / Dashboard preview still iterate past sessions' top-level `sets[]` only — drop stages from prior sessions are invisible until Batch 24 rewrites those render paths. Batch 24 adds the condensed `185×10 → 135×8 → 95×6` chain format to `PrevSetRow` + the bundled rendering in History session detail + Share Card's exercise list. All three batches merge together so the user never sees this gap on main.

394. **Build.** `npx vite build --outDir /tmp/test-build` → 754.45 KB bundle / 203.45 KB gzipped (+4.63 KB / +1.09 KB vs Batch 22 — all in the `DropStageRow` component, drop-stage handlers, bundled rendering loop, and the `cyclerDisabled` / `drops` serialization paths).

### Batch 24 (April 21, 2026) — Drop-set bundling: display surfaces (ghost rows, History, ShareCard, Progress, Dashboard)

Third and final drop-set-bundling batch. Every read-only surface that renders past-session data now understands the bundled shape: the BbLogger "Last Time" ghost row shows drops as a condensed `→`-chained line beneath the primary, History's session-detail modal indents drop stages under each working set, and Share Card / Progress / Dashboard aggregates walk nested drops for volume while counting working primaries only for set displays. All three batches land as a single fast-forward merge to main so users never see a pre-Batch-24 intermediate where persisted data is bundled but display surfaces still expect flat shape.

395. **`PrevSetRow` chain format (`BbLogger.jsx`).** Ghost row now renders the bundled shape natively: the primary row layout is unchanged, but a working-with-drops entry gets a second line beneath — `↳ 135×8 → 95×6` in orange, indented to align with the weight column, truncated at viewport edge. Drops use `rawWeight ?? weight` so unilateral exercises display per-side numbers. Warmups and working-without-drops render as before. Legacy `type:'drop'` top-level entries (defensive — shouldn't exist post-migration) still get a "Drop" chip and no chain.

396. **History session-detail bundled rendering (`History.jsx`).** The per-exercise set list at `:320` now renders each top-level set as today (type chip + `N reps × W lbs` + PR trophy), but adds an indented `↳ {w}×{r} → {w}×{r}` chain in orange beneath any working set with drops. `ml-[54px]` aligns the chain with the reps column so primary + drops read as one logical group. Empty `drops[]` falls through to nothing rendered — pre-bundled sessions look identical to before.

397. **`buildShareData` aggregations (`History.jsx`).** `totalVolume` walks primary + nested drops (decision 2). `totalSets` filters to `type === 'working' && (reps || weight)` (decision 1). `totalPRs` is unchanged since drops don't carry `isNewPR` under the bundled shape (decision 3 — naturally working-only via the data model). Session-card `setCount` at `:493` similarly filters to working primaries.

398. **ShareCard `workingSetCount` (`ShareCard.jsx:368`).** Old filter was `type === 'working' || type === 'drop'` (flat-shape counting). New filter is `type === 'working'` — drops nested inside primaries don't contribute to the stat bar's SETS tile. `getTopSet` at `:121` was already working-only from pre-Batch-22; no change needed.

399. **Progress aggregations (`Progress.jsx`).**
    - `sessionVolume` descends into `set.drops[]` for the bar chart.
    - `sessionSetCount` filters to `type === 'working'` (previously `ex.sets.length` — which under bundled shape correctly excludes drops since they're nested, but also counted warmups; working-only matches the decision 1 rule).
    - Radar-chart muscle-group volume at `:263` includes drops.
    - `PRTimeline` walker at `:355` unchanged — iterates top-level sets and filters `set.isNewPR`, which under bundled shape is always on a working primary.

400. **Dashboard volume calcs (`Dashboard.jsx`).** Both `getWeekVolume` (weekly stat card) and `totalVolume` (all-time total volume stat card) descend into `set.drops[]` so pre-migration volume totals are preserved exactly (decision 2). `lastWorking` lookup at `:417` was already `type === 'working'` — no change. Weekly/monthly calendar preview at `:1546` uses `setsToShow` which filters `weight || reps`; under bundled shape that correctly iterates primaries only.

401. **Centralized helpers already updated in Batch 22.** `calcSessionVolume` / `getExercisePRs` / `getExerciseHistory` / `getAchievements` were rewritten in Batch 22 and needed no further Batch 24 changes. The per-page aggregations touched this batch are the ones that don't go through those helpers.

402. **Verification — end-to-end consistency at the bundle boundary.** After the three-batch merge:
    - (a) Pre-existing sessions migrate on load: flat `type:'drop'` entries bundle into their parent working's `drops[]`.
    - (b) New sessions logged post-merge emit the bundled shape directly (Batch 23's `buildExerciseData`).
    - (c) Every display surface (logger ghost rows, history detail, share card, progress chart, dashboard stats) reads the bundled shape consistently.
    - (d) Volume numbers are preserved (decision 2 — Batch 22 sanity script confirmed 471,755 lb-reps before and after on `debug-backup.json`).
    - (e) Share card SETS and History session-card counts drop for users who drop-set (decision 1 — "one logical set per bundled group"); working-only counts are the new source of truth.
    - (f) PR flags on drop rows are gone from History — the migration cleared 24 stale warmup-flagged PRs as a bonus cleanup (Batch 22 sanity).

403. **Post-v1 roadmap — drop-set bundling shipped.** Updated Post-v1 list below — the "Drop-set bundling (flagged Batch 21)" entry is removed from pending since it's live.

404. **Build.** `npx vite build --outDir /tmp/test-build` → 755.57 KB bundle / 203.77 KB gzipped (+1.12 KB / +0.32 KB vs Batch 23, all in the five display-surface walkers descending into drops).

### Batch 25 (April 21, 2026) — Timezone fix: evening entries stay on the local day

User report from North America users: logging a workout at 5 PM (paraphrase — the actual symptom triggers at ~7-8 PM Eastern when UTC rolls over) was being counted toward the next day in cardio-attachment checks, Progress heatmap + week filters + PR timeline, and split "Created" dates. Batch 16k fixed this pattern on the Dashboard calendar strip (via a private `isoToLocalDateStr` helper), but BbLogger, CardioLogger, Progress, and useStore.js never got the same treatment — they continued to call `new Date().toISOString().split('T')[0]` to derive "today's date," which returns the UTC date. For any browser west of UTC logging past ~7-8 PM local, UTC has already rolled over and the returned date is tomorrow.

User suggested "set the default time zone to Central" as a fallback. That would have been a partial fix at best: a Pacific user's 5 PM would be interpreted as Central's 7 PM (still correct); an Eastern user's 11 PM would be interpreted as Central's midnight (still wrong). The app already has access to each user's real local timezone via the browser's `Date` API — it just wasn't using it consistently. So the correct fix is to replace every `toISOString().split('T')[0]` with a helper that derives the date from `getFullYear()/getMonth()/getDate()` (all of which return LOCAL values). That works for every US timezone and every timezone worldwide with no need for a hardcoded fallback.

405. **`toLocalDateStr(input)` exported from `helpers.js`.** Flexible-signature canonical "what local date is this?" helper. Accepts a `Date` object, an ISO string, a date-only `YYYY-MM-DD` string, or `undefined` (defaults to right now). Returns `YYYY-MM-DD` in the browser's LOCAL timezone, or `null` for unparseable input. Date-only strings are normalized through `'T00:00:00'` so they parse as local midnight (avoiding the UTC-midnight drift `new Date('2026-04-07')` introduces in negative-UTC offsets). Replaces the pre-Batch-25 private `toLocalDateStr(d)` — same name, now exported with a richer signature. Sanity-checked via 7/7 node cases covering every input shape + the headline case (a Date at 8:30 PM local in UTC-5 returns the local date, not the UTC-tomorrow date).

406. **BbLogger.jsx — 2 sites updated.** `new Date().toISOString().split('T')[0]` → `toLocalDateStr()`:
    - Inline-cardio save path (the "Log Now" flow from the Finish modal): `cardio.date` is now a LOCAL date string.
    - `todayStr` / `todayCardio` filter used by the Finish modal to find today's unattached cardio — now uses local date, and the comparison is robust to either flat date strings OR full ISO via `(s.date || '').slice(0, 10)`.

407. **CardioLogger.jsx — 5 sites updated.** Cardio save path writes `date: toLocalDateStr()`. The two "check for today's workout" paths (`handleConfirm` auto-attach check + `getTodayWorkoutName` display) both compare via `toLocalDateStr(s.date)` so attach logic finds same-day workouts regardless of which timezone the user is in. The `cardio.date` storage format remains a date-only `YYYY-MM-DD` string (unchanged from pre-Batch-25), just now derived locally.

408. **Progress.jsx — 4 sites updated.** Weekly-load-chart session filter (`getWeekBounds` comparison), PR-timeline grouping key (`${exerciseName}|${localDate}`), and consistency-heatmap byDate bucket all switched from UTC-derived to local-derived date strings. Private `toDateStr(d)` helper at `:27` now delegates to the shared `toLocalDateStr` so Progress and helpers stay in sync.

409. **useStore.js — 3 sites updated.** Split `createdAt` writes in `cloneSplit`, `duplicateSplit`, and `addSplit` paths all use `toLocalDateStr()` instead of `toISOString().split('T')[0]`. Previously a split created at 10 PM Pacific on April 21 would show `Created April 22, 2026` on the SplitManager card; post-fix it correctly reads `Created April 21, 2026`.

410. **What was deliberately not touched.** Export backup filename (`workout-backup-YYYY-MM-DD.json` at `useStore.js:761`) keeps its UTC-derived filename — changing it would alter users' backup-file naming conventions without meaningful benefit (export is a snapshot, not a user-facing day-grouping). `activeSession.date` + `session.date` keep their full-ISO `toISOString()` storage format for sessions — that's the canonical timestamp, and reader code now converts via `toLocalDateStr(s.date)` at display / grouping time. Dashboard.jsx's private `isoToLocalDateStr(iso)` helper from Batch 16k is left alone (does the same thing as the shared helper for ISO inputs; refactor-only change, not a behavior fix).

411. **Note on existing historical data.** Cardio sessions + splits created pre-Batch-25 with a `createdAt`/`date` that's a UTC date string (evening entry in US timezone → tomorrow's date) remain as-is. The fix corrects ONLY new writes going forward. No data migration — the legacy dates are ambiguous (we don't know the user's timezone at the moment of save), so any post-hoc correction would be guessing. Users can delete + re-log if a specific entry is visibly on the wrong day; otherwise the inconsistency just ages out naturally.

412. **Sanity.** `node -e` test of `toLocalDateStr`: 7/7 pass. Cases: `undefined` → today; `null` → today; Date at 8:30 PM local (UTC-5) → local date (not UTC-tomorrow); ISO string → correct local date; date-only string → same string; invalid string → null; number → null.

413. **Build.** `npx vite build --outDir /tmp/test-build` → 755.33 KB bundle / 203.79 KB gzipped (−0.24 KB / +0.02 KB vs Batch 24 — helper replaces inline string ops, roughly a wash).

### Batch 26 (April 21, 2026) — Traps + Abs added to MUSCLE_GROUPS

User-requested tiny addition. Previously 12 muscle groups; now 14.

414. **`MUSCLE_GROUPS` in `src/data/exerciseLibrary.js`.** Added `'Traps'` (after `'Back'`) and `'Abs'` (before `'Core'`). Full list now: `Chest, Back, Traps, Shoulders, Quads, Hamstrings, Glutes, Biceps, Triceps, Abs, Core, Calves, Forearms, Full Body`. `Core` stays as the broader category (obliques + stabilizers + lower back); `Abs` is the narrower abdominal-specific tag for crunches, leg raises, etc. — users can now tag the distinction. Single source of truth, so the four pickers that consume it (`Backfill.jsx`, `ExerciseEditSheet.jsx`, `ExercisePicker.jsx`, `CreateExerciseModal.jsx`) all pick up the new options automatically.

415. **Bonus fix: `predictExerciseMeta` "shrug" prediction now validates.** The keyword map at `helpers.js:1617` already predicted `['Traps']` for exercises matching "shrug", but Traps wasn't in `MUSCLE_GROUPS` — so `addExerciseToLibrary`'s §3.2.1 validation silently rejected the prediction and the user ended up manually picking a different group. With Traps now valid, the auto-fill chip actually sticks on save. `predictExerciseMeta`'s "crunch / sit-up / leg raise / plank / russian twist / ab wheel" predictions continue to map to `'Core'` — not changed this batch; users who want those tagged as `Abs` can switch manually.

416. **No code changes beyond the array.** No build-size change (+0 KB — the strings are tiny). Existing library entries that currently use `muscleGroup: 'Back'` for trap-adjacent exercises (e.g., shrugs) stay as-is; users can re-tag via `/exercises` if they want the finer distinction. No migration needed.

### Batch 27 (April 21, 2026) — Machine split into Selectorized / Plate-loaded + v6 library migration

User request: "have selectorized machine and plate-loaded machine as the two machine options." Previously a single `'Machine'` equipment type covered both Hoist / Cybex selectorized stacks AND Hammer Strength / Smith / hack-squat plate-loaded rigs, which tangled the Machine-instance data together (recommender couldn't tell apart a selectorized Pec Dec at VASA from a plate-loaded one at TR even when they were physically different machines). Split into two specifics with a v5→v6 persist migration that remaps legacy `'Machine'` to `'Selectorized Machine'` (safer default at commercial gyms). Built-in library updated in place with best-guess specifics; user-specific adjustments via `/exercises`.

417. **`EQUIPMENT_TYPES` (`src/data/exerciseLibrary.js`).** Replaced `'Machine'` with `'Selectorized Machine'` + `'Plate-loaded Machine'`. Full list now: `Barbell, Dumbbell, Selectorized Machine, Plate-loaded Machine, Cable, Bodyweight, Kettlebell, Other`. Single source of truth, so the four pickers that consume it (Backfill, ExerciseEditSheet, ExercisePicker, CreateExerciseModal) all pick up the split automatically.

418. **Built-in library entries updated** (`data/exerciseLibrary.js`). 27 `equipment: 'Machine'` entries repartitioned:
    - **Plate-loaded** (6): Incline Smith Machine Press, Any Plate-loaded Press, Squats or Smith Machine Squat, Hack Squats, Leg Press, Belt Squat, Donkey Calf Raise. Rationale: Smith machines load plates; most hack-squat / leg-press / belt-squat rigs at commercial gyms are Hammer Strength–style plate-loaded.
    - **Selectorized** (21): Pec Dec, Chest Supported Wide Row, Reverse Pec Dec, Leg Extensions, Leg Curls, Seated Leg Curl, Lying Leg Curl, Adductors, Abductors, Calf Raises, Standing Calf Raise, Seated Calf Raise. Default for isolation / support machines — Hoist / Cybex / Life Fitness style pin-select stacks.
    Users whose gym has the opposite style (e.g., plate-loaded Pec Dec at a Hammer Strength gym) can re-tag in `/exercises` → open → pick the other option. One-tap fix.

419. **`predictExerciseMeta` keyword map updates** (`helpers.js`). Auto-fill predictions for machine-family keywords now use specific types: `leg press / hack squat` → Plate-loaded Machine; `leg extension / leg curl / chest press / pec dec / pec deck / rear delt / calf raise` → Selectorized Machine. Users typing a new machine exercise into the Create modal get the specific chip pre-selected after the 300ms debounce — no dead legacy `'Machine'` values leaking through to the strict §3.2.1 addExerciseToLibrary validator.

420. **`isMachineEquipment(equip)` helper (`helpers.js`).** Pure predicate returning true for `'Selectorized Machine'`, `'Plate-loaded Machine'`, or the legacy `'Machine'`. Used by the BbLogger Machine-chip visibility gate (Batch 19a) so the chip surfaces on all three. Keeps the legacy branch for defensive correctness even though v6 migration clears it on load — if a session's cached library entry somehow has `'Machine'`, the chip still renders instead of silently hiding.

421. **`migrateLibraryToV6(library)` helper (`helpers.js`).** Walks each exercise entry, rewrites `equipment: 'Machine'` → `equipment: 'Selectorized Machine'` (safer default — selectorized is more common at commercial gyms than plate-loaded). Returns same reference when nothing changed (O(1) idempotency marker). No-op on non-array inputs. Sanity-checked via 14/14 node cases: `isMachineEquipment` across 7 value shapes; migration correctly rewrites legacy, preserves specifics + non-machine equipment, returns same ref when idempotent, and handles null/empty defensively.

422. **Persist version bumped `5 → 6` (`useStore.js`).** New migrate block calls `migrateLibraryToV6(persistedState.exerciseLibrary)` when `version < 6`. Additive — doesn't touch sessions or settings. Pre-v6 users' libraries (including the 27 built-in machine entries that persisted from a prior install) all shift from `'Machine'` → `'Selectorized Machine'` on first v6 load. Users adjust plate-loaded entries manually via `/exercises`.

423. **BbLogger Machine-chip gate updated (`BbLogger.jsx:1132`).** Pre-Batch-27: `libraryEntry?.equipment === 'Machine'` (fails for new specific values). Post-Batch-27: `isMachineEquipment(libraryEntry?.equipment)` (works for both new values + legacy). Chip-visibility behavior end-to-end: a Hoist Pec Dec (Selectorized) + Plate-loaded Leg Press both show the Machine chip; Barbells / Dumbbells / Bodyweights still hide it. Cable still deliberately excluded (Batch 19a rationale).

424. **What was deliberately not touched.** Historical session data (`session.data.exercises[].equipmentInstance`) stays as user-typed free-text — that's a per-session machine-name tag ("Hoist" / "Cybex"), which is orthogonal to the library's equipment-type classification. `settings.*` unchanged. Sessions array unchanged. No migration-sanity script for this batch — the migration is a one-line string swap with full 14/14 node test coverage.

425. **Build.** `npx vite build --outDir /tmp/test-build` → 755.56 KB bundle / 203.82 KB gzipped (negligible +0 KB change — two helpers + one enum split + 27 raw-data string replacements all net to a wash).

### Batch 28 (April 21, 2026) — Session logger polish sweep

Live-feedback round after Monday's Quads workout at Lanhammer. Seven discrete items — the biggest being `hiddenAtGyms` (data-layer deny-list for per-gym exercise visibility) and the Coach's Call "Use it" button that one-taps a prescription into the next empty working set (plate-mode aware via a greedy plate packer).

426. **Gym pill neutral styling + "Resumed from saved session" subheader killed** (`SessionGymPill.jsx`, `BbLogger.jsx`). Filled-state gym chip now uses `rgba(0,0,0,0.35)` bg + `rgba(255,255,255,0.3)` border + `rgba(255,255,255,0.95)` text — contrasts against any accent header gradient (was using `${theme.hex}1a`+`${theme.hex}` which disappeared against the matching-accent header). Subheader "Resumed from saved session" removed entirely (not useful); unused `isResumed` state dropped. Gym pill margin tightened `mt-1.5 → mt-1` since the subheader no longer spaces it out.

427. **Drop stage CTA polish + hide after next working set** (`BbLogger.jsx`, `space-y-1.5`). Styling softened `text-orange-400 font-semibold → text-xs italic font-medium text-orange-400/70` + border opacity `/40 → /30` — reads as a subtle opt-in, not a primary affordance. New gate `!hasLaterWorking` (set index j > i exists with type='working') hides the CTA on prior working sets once the user advances — no stale "+ Drop stage" offering retroactive drops on already-finished sets.

428. **Set 2+ locked to Work** (`SetTypeBtn` + `addSet` + `SetRow`/`PlateSetRow`). Warmups only make sense on the first set; subsequent sets displayed-and-stored as working regardless. New `lockedToWorking` prop on SetTypeBtn forces effective value to 'working', disables click handler, renders `cursor-default` (no opacity dimming — looks like a normal Work chip, just non-interactive). `addSet` for setIndex > 0 now always writes `type: 'working'` (dropped the warmup-inheritance branch) so stored data matches the forced display.

429. **Drop stage rows at ~2/3 height of primary** (`DropStageRow`). Compact row treatment reinforces bundling visually: `h-10 → h-7` on the ↳ Drop label + weight/reps inputs + × button; `text-base → text-sm` on input values; `rounded-lg → rounded-md`; `py-2 → py-0`; × icon `w-4 h-4 → w-3.5 h-3.5`. Verified live: primary 40px / drop 28px (ratio 0.70).

430. **Focus mode header collapse + hardening** (`BbLogger.jsx`, `index.css`). When the numpad is open, the title row (emoji + workout name + gym pill) collapses to 0px via CSS max-height + opacity transition (0.2s ease-out). Saves ~64px of vertical real estate for set rows during data entry. Back arrow in focus mode becomes "Exit focus mode" — taps close the numpad instead of navigating away (prevents accidental session-exit); second tap does the normal navigate-back. aria-label flips dynamically. Old "Tap to show all exercises" zone rewritten as a compact accent-tinted "Show all exercises" pill (inline-flex, hugs content, centered; was full-bleed + stacked). Timer centering fixed via `flex items-center` + `leading-none` on the font-mono span (was sitting 4px low). Back button shrunk `w-8 h-8 → w-7 h-7` to match pause button. `closeNumpad` now blurs `document.activeElement` so tapping the SAME input after Show all re-fires onFocus and reopens the numpad (previously a no-op because React only re-fires onFocus on blur→focus edges).

431. **Hide for this gym** (data + UI + logger filter). New `Exercise.hiddenAtGyms?: string[]` field + store actions `addHiddenAtGym(exerciseId, gymId)` / `removeHiddenAtGym(exerciseId, gymId)` (drops field when array empties so shape stays minimal). New pure predicate `isExerciseHiddenAtGym(exercise, gymId)` in `helpers.js`. GymTagPrompt's third button renamed "Always skip" → **"Hide for this gym"** with red styling (`rgba(248,113,113)` text + border, transparent bg) to signal destructiveness. On tap: native confirm → if accepted, dual-writes `addHiddenAtGym` + `addSkipGymTagPrompt`. Logger's `templateExercises` + merged-extras loops filter out exercises where `isExerciseHiddenAtGym(libraryEntry, seedGymId)` — exercise disappears from the workout list at that gym on next session mount, stays visible at other gyms (prompt still fires there normally). History unchanged — past sessions that included the now-hidden exercise still show it in the History detail modal. Un-hide UI flagged for Batch 29.

432. **"Use it" button in Tip card** (`RecommendationSheet` + `BbLogger.jsx` + `helpers.js` `recommendPlatesForWeight`). New emerald ghost-pill matching the Tip chip's style (✨ sparkle icon + `bg-emerald-500/15 border border-emerald-500/40 text-emerald-300`) rendered in the sheet header. On tap: closes sheet → finds first working set with empty weight (or `addSet` if none) → populates weight with `prescription.weight` → focuses reps input via `setRepsRefs.current[idx].focus()`. In plate mode, the new `recommendPlatesForWeight(target, bar, multiplier)` helper does a greedy plate-fit across standard Olympic plates [45, 35, 25, 10, 5, 2.5] and rounds DOWN to achievable totals so the user never accidentally overshoots. Verified: 145 lbs @ 45 bar 2× → `{45:1, 5:1}`; 230 @ 45 2× → `{45:2, 2.5:1}`. Close button × also upgraded from unicode glyph to SVG icon (centers cleanly in the 32×32 pill regardless of font baseline). Trace-animation on the target set row was built + discarded per user feedback ("thin and slow... actually just remove it") — Use it works plain-text now.

433. **Gym tag prompt visual parity** (`Recommendation.jsx`). All three GymTagPrompt buttons were unified to outlined ghost-pills (accent-text + transparent bg) briefly, then reverted per user feedback ("Yes, tag it looks already selected that way — prior version was better"). Final state: "Yes, tag it" stays solid accent-filled (primary), "Not this time" neutral ghost, "Hide for this gym" red ghost.

434. **markDone reliable scroll-to-top** (`BbLogger.jsx:1093`). Double-requestAnimationFrame before `window.scrollTo({top:0, behavior:'smooth'})` so the scroll fires AFTER React commits the done-state change + the re-sort that moves completed exercises to the bottom. Pre-Batch-28 behavior sometimes landed the scroll in the wrong place because `scrollTo` fired synchronously against the pre-commit DOM. First rAF waits for commit queue flush; second waits for paint.

435. **Files modified summary.** `src/pages/log/SessionGymPill.jsx` (item 1); `src/pages/log/BbLogger.jsx` (items 1/2a/2b/2c/3/4/5 — biggest diff); `src/pages/log/Recommendation.jsx` (item 3 prompt + item 4 Use it button); `src/store/useStore.js` (item 3 actions); `src/utils/helpers.js` (item 3 predicate + item 4 plate packer); `src/index.css` (transient trace animation, added + removed).

436. **Build.** `npx vite build --outDir /tmp/test-build` → passes clean.

### Batch 29 (April 24, 2026) — Session logger hotfixes

Three hotfixes from live session logger use: plate bar weight not sticking session-to-session, unilateral doubled weights leaking into post-session display surfaces, and the Coach's Call sheet's "Use it" button visually colliding with the 2xl weight text on mobile. Ships alone so the more substantial rep-range inference (Batches 30 + 31) gets a clean revert point.

437. **Plate-mode bar weight persists session to session** (`BbLogger.jsx`). Both `templateExercises` init and the `defaultExercises` extras path now seed the bar from the last session's first set that recorded one: `(lastExDataByName[name]?.sets || []).find(s => s?.barWeight != null)?.barWeight ?? 45`. Previously both init sites hardcoded 45 regardless of history, so the popover silently reset every session and forced the user to re-pick 25 / None each time. **Writes both `barWeight` AND `barDefault`** on the exercise object: `PlateConfigPopover` (line 1440) reads `ex.barDefault`; `ex.barWeight` at the exercise level turned out to be vestigial. Writing both defensively keeps every downstream consumer happy regardless of which field it reads.

438. **Unilateral display uses per-side weight across post-session surfaces** (`History.jsx`, `ShareCard.jsx`). For unilateral exercises, `set.weight` holds the doubled (total) volume value and `set.rawWeight` holds the per-side input. The logger's ghost rows, inline "Last:" hints, PR chip, and recommender sheet already use `perSideLoad(set)` correctly (Batch 9 / 14 / `getExerciseHistory` return shape). This batch swaps the remaining leaks:
    - `History.jsx:344` session detail set row: `{s.reps} reps × {s.weight} lbs` → `{s.reps} reps × {perSideLoad(s)} lbs`.
    - `History.jsx:349` drop chain inside session detail: now reads `perSideLoad(d)` per drop so unilateral parents chain as `↳ 60×8 → 45×6` instead of `↳ 120×8 → 90×6`.
    - `ShareCard.jsx getTopSet`: comparison still reads `s.weight` (volume-consistent within a single exercise since all sets share the same unilateral state); display swaps from `best.weight` to `perSideLoad(best)` so the share card's exercise-list line matches what the user inputted.
    Volume math everywhere (History `buildShareData`, Progress, Dashboard weekly / total, session comparison) unchanged — `set.weight` is the correct doubled value for volume totals per Batch 22 decision 2.

439. **Coach's Call sheet header restructured to two rows** (`Recommendation.jsx:254–301`). Pre-29 layout was a single flex row: `[Recommended top set: 180 × 10]  [✨ Use it] [×]`. At 375px the 2xl weight value and the emerald Use it pill competed for the middle band and visually collided. Restructured:
    - Row 1: label + 2xl weight×reps, full width, `flex-wrap` so extreme cases gracefully spill to row 2.
    - Row 2: `Last session's top set: …` subtitle left (truncated if needed) + `[✨ Use it] [×]` right-aligned via `justify-between`.
    Close button stays reachable regardless of prescription state; the no-prescription state renders "No prescription yet" + close button on a single row with no empty row above.

440. **Build.** `npx vite build --outDir /tmp/test-build` → 760.19 KB bundle / 205.15 KB gzipped (+4.63 KB / +1.33 KB vs Batch 28, accounted for by the three comments + `perSideLoad` imports in History + ShareCard + the two-row restructure JSX).

### Batch 30 + 31 (April 24, 2026) — Rep-range inference + advisory UI

Solves the user-reported "coach silently deloaded me from 90×8 → 80×10 even though I'm progressing at +5%/wk" problem. Pre-Batch-30 the recommender used a uniform `targetReps=10` across every exercise, and auto-deloaded after two consecutive sessions missed by ≥2 reps. User's observation: "targetReps=10 doesn't match how I train — for compound lifts 6–8 is a strong set, for calves 12–15 is normal." Plan: per-exercise [min, max] range, INFERRED from the user's own recent rep counts by default (with an editable override surface). Engine becomes advisory instead of prescriptive: the auto-deload silent override is gone; when the user drops below their own declared floor twice in a row, a soft banner surfaces a one-increment-down weight suggestion next to the normal Push/Maintain chips instead of rewriting them. Both batches ship together via `claude/rep-range-inference` feature branch so users never see a mid-refactor state where persisted data has ranges but the UI doesn't.

User-locked design decisions (from plan conversation):
1. Engine INFERS range from the user's last 6 top-set rep counts by default — zero-friction, reflects actual training pattern.
2. User override via `ExerciseEditSheet` (plus quick Edit→ from inside the Recommendation sheet) flips `repRangeUserSet: true`.
3. `Your range: 6–9 reps (inferred)` vs `(you set this)` suffix makes the source visible per "AI inferences should be visible."
4. Below-floor-streak detection keeps the push prescription at current-strength-level; advisory is a SOFT suggestion, not a silent override.
5. Deload step is one `loadIncrement` down, not 10% off last weight. Respects per-exercise increment (DB → 5 lb step, BB → 10 lb step).

441. **`classifyRepRange(name, equipment, primaryMuscles)` (`helpers.js`).** Cold-start default when history has < 4 sessions. 50-entry keyword map, ordered specific-first. Returns [min, max]: compound barbell (squat, deadlift, bench, barbell row, overhead press) → [5, 8]; DB / machine press + row + pulldown → [6, 10]; isolation (curl, extension, fly, pec dec, pushdown, hip thrust) → [8, 12]; side / rear delts + calf raise + forearms + core (crunch, leg raise, plank) → [10, 15]. Equipment fallback: Barbell → [5, 8]. Muscle fallback: Calves / Forearms → [10, 15]. Default [8, 12].

442. **`inferRepRange(history, classificationDefault)` (`helpers.js`).** Pulls [min, max] from the user's last 6 top-set reps: min = worst rep count in window, max = best + 1 (gives a ceiling to push toward). Falls back to classificationDefault when < 4 sessions. Clamped to [3, 25]. For Overhead DB Extension with real backup history `[6, 7, 8, 8, 8, 8]` yields `[6, 9]` — close to user's stated intent of `[6, 10]`.

443. **`migrateLibraryToV7(library)` (`helpers.js`).** Idempotent v6 → v7 library migration. Re-seeds every entry's `defaultRepRange` per `classifyRepRange` (pre-v7 had uniform `[8, 12]` across built-ins), and adds `repRangeUserSet: false` on every entry. Entries with `repRangeUserSet: true` keep their range intact — they're user overrides. Returns same reference when no changes (matches v6 idempotency marker).

444. **Persist version bumped `6 → 7` (`useStore.js`).** New migrate block + chain — `if (version < 7) library = migrateLibraryToV7(library)`. `importData` also runs v6 + v7 on imported libraries so pre-v7 backups land in the current schema. `buildBuiltInLibrary` now calls `classifyRepRange(raw.name, raw.equipment, [raw.muscleGroup])` instead of hardcoding `[8, 12]`.

445. **`recommendNextLoad` signature change + decision-rule rewrite (`helpers.js`).** Replaces `targetReps = 10` with `repRange = [min, max]`. Backwards-compat: if a legacy caller still passes `targetReps`, derives `[targetReps - 2, targetReps]`. New range-aware decision rule:
    - `reps >= max` → push (Layer 3 nudge, existing math unchanged; Layer 2 floor at `maxReps`).
    - `min ≤ reps < max` → hold weight, "in range" reasoning.
    - `reps < min` (single session) → hold weight, "below floor once, fine" copy.
    - `reps < min` for 2 consecutive sessions → `meta.belowFloorStreak = 2` + `meta.suggestedDeloadWeight = last.weight - loadIncrement`. Prescription itself stays at current-strength-level. NO silent override.
    - `mode === 'deload'` (user-declared) → `last.weight - loadIncrement` rounded to loadIncrement. Softer than the pre-Batch-30 65%-of-e1RM formula, respects per-exercise loadIncrement.
    Meta additions: `repRange: [min, max]`, `belowFloorStreak`, `suggestedDeloadWeight`.

446. **`rep-range-sanity.mjs` at worktree root.** 60/60 pass. Covers: classifyRepRange across 22 exercise names + equipment + muscle fallbacks; inferRepRange edge cases (< 4 sessions, exactly 4, clamp at 25, reps:0 filter, null history); Overhead DB Extension user scenario (range inferred to [6, 9], push rec holds weight, no silent deload); below-floor-streak detection on 2 consecutive sub-min sessions + single-session handling; in-range hold; loadIncrement-aware deload for 2.5 / 5 / 10 lb; migrateLibraryToV7 idempotency + user-override preservation; real-backup sanity spot-check.

447. **BbLogger `recRepRange` memo (`BbLogger.jsx`).** Replaces `recTargetReps`. Resolution order:
    1. `libraryEntry.defaultRepRange` when `libraryEntry.repRangeUserSet === true` (user override).
    2. `inferRepRange(recHistory, classified)` where `classified = classifyRepRange(name, equipment, muscles)`.
    3. Classification default (via inferRepRange fallback) when history is thin.
    Threaded through `recommendNextLoad({history, repRange: recRepRange, ...})` and into `RecommendationSheet` as `repRange` + `repRangeUserSet` props.

448. **`ExerciseEditSheet` gains two number steppers (`src/components/ExerciseEditSheet.jsx`).** "Top-set rep range" section between Load increment and Unilateral. `Progress when reps ≥ [max]` + `Back off when reps < [min]`, each with − / + stepper buttons (tap-only, no keyboard needed). Values clamp [1, 30]; min nudges up with max when max drops below it (and vice versa). Save commits both values + flips `repRangeUserSet: true` + clears `needsTagging: false` (pre-existing). Pre-fills from the current range regardless of whether it's user-set or inferred-and-stamped by v7.

449. **"Your range" row in Recommendation sheet (`Recommendation.jsx`).** New row between the sparkline key and mode chips. Format: `Your range: 6–9 reps (inferred) [Edit →]` vs `Your range: 6–10 reps (you set this) [Edit →]`. `Edit →` button closes the recommendation sheet and fires `onEditLibraryEntry(libraryEntry)` from BbLogger, which opens `ExerciseEditSheet` stacked above at its native z-260. Hidden when `minReps`/`maxReps` aren't resolvable. New props: `repRange`, `repRangeUserSet`, `libraryEntry`, `onEditRange`.

450. **`BelowFloorAdvisory` inline in Recommendation sheet (`Recommendation.jsx`).** Renders between the Your-range row and mode chips when `recs.push.meta.belowFloorStreak === 2` and not dismissed this session. Amber tint (`rgba(245, 158, 11)` at 8% bg + 35% border). Copy: `**Two sessions below your {min}-rep floor.** Today could be a lighter reset day. [Use lighter weight: {suggestedDeloadWeight} lbs] [×]`. Use-lighter button calls `onApply({ weight: suggestedDeloadWeight })` (reuses the existing Use-it path from Batch 28) + dismisses the advisory. × button dismisses only. Push + Maintain chips keep their normal current-strength-level values.

451. **`settings.dismissedBelowFloorAdvisories` + `dismissBelowFloorAdvisory(exerciseKey)` (`useStore.js`).** Session-scoped dismissal map keyed by `exerciseId || name`, mirrors `dismissedAnomalies`. Stamps `activeSession.startTimestamp` against the key; next session's startTimestamp change auto-invalidates stale entries and the advisory returns if the streak still fires. No un-dismiss UI — the streak breaks naturally when the user hits at or above floor.

452. **Growth label disambiguation (`Recommendation.jsx`).** The sparkline key's `Growth: +7.0%/wk` renders as `Weekly growth: +7.0%` post-Batch-31. User reported parsing `+2.0%/wk` as a "+2 lbs" weight delta in one of the data points (Rear Delts had a ~1.7%/wk rate that rounded to +2.0%). Explicit "Weekly growth:" prefix removes the ambiguity.

453. **Files modified summary.** `src/utils/helpers.js` (classifier, inferrer, migration, engine rewrite); `src/store/useStore.js` (v7 persist, buildBuiltInLibrary classifier, importData v6+v7 run, dismissedBelowFloorAdvisories slice + action); `src/pages/log/BbLogger.jsx` (recRepRange memo, ExerciseItem thread-throughs, parent-level ExerciseEditSheet render, below-floor dismissal state); `src/pages/log/Recommendation.jsx` (Your-range row, advisory banner, Growth → Weekly growth, new props); `src/components/ExerciseEditSheet.jsx` (rep range steppers + save flip).

454. **Build.** `npx vite build --outDir /tmp/test-build` → 769.24 KB bundle / 206.99 KB gzipped (+9.05 KB / +1.84 KB vs Batch 29, accounted for by the 50-entry classifier map + inferRepRange + v7 migration + range-aware recommender branch + the Your-range + advisory + stepper UI additions).

### Batch 32 (April 24, 2026) — Paste Outline + retroactive drop-type cycler

Two live-feedback features on the session logger. Both live entirely inside `src/pages/log/BbLogger.jsx` — no engine / helpers / persist / schema changes. Ship together since they overlap the drop-stage UI surface.

455. **Paste Outline** (`BbLogger.jsx` ExerciseItem + ghost-row block). Copies last session's set structure into today as an editable scaffold. Button appears below the ghost rows in the "Last" block when toggled on, only when last session had at least one working set. Merge rules: (a) never overwrite a today set whose weight or reps is filled; (b) fill empty slots from last's matching set; (c) append sets when last has more than today; (d) final count = `max(today.sets.length, last.sets.length)`. **Reps are intentionally left blank** on the pasted set (and any nested drop stages) — paste is a scaffold for structure + weight + plate config, not a prediction of today's rep performance. Drops deep-copied to the new set's `drops[]` (per Batch 23 shape — weight only, no plates / barWeight / type / isNewPR / plateMultiplier; drop reps also blank). Plate mode: copies plates + barWeight verbatim, recomputes `weight` via `calcTotal(plates, barWeight, exercise.plateMultiplier)` so the displayed total matches today's multiplier if it differs from last's. Non-plate mode strips stale plate fields. Uni handling: pastes `perSideLoad(lastSet)` into `set.weight` (per-side numeric), `buildExerciseData` re-doubles at save time via `exercise.unilateral` — consistent with the rest of the in-memory session shape.

456. **Model A retroactive drop-type cycler** (`SetTypeBtn` + `SetRow` + `PlateSetRow` + ExerciseItem). New `onConvertToDrop` prop on `SetTypeBtn`. When provided at a locked-to-Work position (set index ≥ 1, previous set is a working primary, this set has no drops attached), the chip becomes interactive again — tapping fires the callback which structurally extracts the set from top-level `sets[]` and attaches it as a drop stage under the preceding working set. Label stays "Work" because the click *performs* the conversion — the set leaves `sets[]` entirely and re-renders as a `DropStageRow` under the parent. Preserves weight + reps; strips type / isNewPR / plates / barWeight / plateMultiplier per the Batch 23 drops[] shape. Numpad focus moves to the new drop's weight field so the user continues typing without interruption. Orphan guard: when previous set is a warmup or absent, the prop is null → SetTypeBtn falls back to non-interactive "Work" chip (existing locked behavior). `cyclerDisabled` (working + has drops) still wins — disabled chip stays non-interactive regardless. `addSet` + `updateSet` unchanged; the "+ Drop stage" CTA at `:1699–1707` also unchanged (still gated `!hasLaterWorking`). Interaction matrix:
    - Set 0: Warm ↔ Work existing cycler.
    - Set ≥ 1 + prev working + no drops: tap converts to drop.
    - Set ≥ 1 + prev working + has drops: `cyclerDisabled` → disabled.
    - Set ≥ 1 + prev warmup / missing: non-interactive Work chip.

457. **Files modified summary.** `src/pages/log/BbLogger.jsx` — only file touched. Four discrete surfaces: `SetTypeBtn` gains `onConvertToDrop` branch (:88); `SetRow` + `PlateSetRow` thread the prop through to SetTypeBtn (:450, :630, :700, :730); `ExerciseItem` gains `pasteOutline()` (after `handleApplyRecommendation` at :1058) and `convertSetToDrop()` (after `deleteDropStage` at :1112); the rendering loop computes `canConvertToDrop` per set (:1786) and the ghost-row block renders the `📋 Paste Outline →` button (:1760).

458. **Build.** `npx vite build --outDir /tmp/test-build` → 771.57 KB bundle / 207.74 KB gzipped (+2.33 KB / +0.75 KB vs Batch 31, accounted for by the two handlers + the button + the prop-threading branches).

### Batch 35 (April 24, 2026) — Readiness check-in polish: prominent gym + exercise order

User feedback on the pre-session check-in overlay: "Choosing the gym that you're at should be more prominent, not sort of nested below" + new feature request "Exercise Order" with two options (Last session / Default). Both land in one batch — purely UI + a local reorder helper in BbLogger. No schema / persist / store changes.

465. **Gym promoted to a labeled top-of-form row** (`src/pages/log/ReadinessCheckIn.jsx`). Was a small 12px pill nested below the three readiness rows — easy to miss per the user's report. Now a full-width button sitting directly beneath the workout title, with an uppercase `GYM` label above it (matching the other row labels). Filled state: `bg-white/10 border-white/15` with the gym label rendered at `text-base font-semibold` on the left and a muted `change` affordance on the right. Empty state: dashed `border-white/20 border-dashed` pill reading `Where are you lifting?` with a muted `pick` affordance. The existing portal-style popover pattern stays — same gym list, same `Add gym name…` inline input, same case-insensitive dedupe via `addGym`. Only the trigger's prominence changed.

466. **Exercise Order row** (`ReadinessCheckIn.jsx`). New 2-button grid row at the bottom of the readiness stack, between `Today's goal` and `Start Session`: `[Last session] [Default]`. Default value is `'default'` so untouched sessions behave identically to pre-Batch-35. Payload now carries `orderMode` on both the answered path (`onStart({energy, sleep, goal, gymId, orderMode})`) and the Skip path (`onStart({readiness: null, gymId, orderMode})`) — exercise order is an independent preference from readiness data, so a user can skip the check-in while still picking a non-default order.

467. **`ReadinessRow` column-count adapts to options length** (`ReadinessCheckIn.jsx`). 2-option rows (Exercise Order) use `grid-cols-2`; 3-option rows (Energy/Sleep/Goal) keep `grid-cols-3`. Single primitive handles both layouts so the visual rhythm across all four rows stays consistent.

468. **`reorderByLastSessionCompletion(exercises, lastSession)` helper** (`src/pages/log/BbLogger.jsx`, module-scope). Pure function. Buckets `exercises` by `group` (section label) while preserving first-seen group order from the template, then sorts WITHIN each group by `completedAt` ascending — earliest-completed last session wins top slot within its section. Exercises with no `completedAt` on last session sink to the bottom of their section (stable sort preserves any tiebreaker order for them). No-op when `exercises` is empty, when there's no `lastSession`, or when the last session had no `completedAt` timestamps. Section order from today's template is never rearranged; only within-section order shifts.

469. **`handleStartSession` applies `orderMode`** (`BbLogger.jsx`). When `payload.orderMode === 'lastSession'`, looks up the prior session via the already-imported `getLastBbSession(sessions, type)` and calls `setExercises(prev => reorderByLastSessionCompletion(prev, prior))`. Only applies at fresh start — resumed sessions skip the whole overlay (guarded at `!sessionStarted`) so their saved order stays intact. `'default'` is a complete no-op. Comment on the handler now documents both the answered + Skip payload shapes.

470. **Verified live in preview** (mobile 375×812, debug-backup.json). Layout: overlay renders with stacked rows in order GYM → ENERGY → SLEEP → TODAY'S GOAL → EXERCISE ORDER → Start Session → Skip/Go back, all left-aligned labels in uppercase 10px text. Gym button reads `Training Room change` in large semibold white text; tapping opens the existing popover with `Training Room ✓` + `Lanhammer` + `Add gym name…` row. With `orderMode = 'default'`: Legs 2 template renders in canonical order `Seated Leg Curl / Single Leg RDL / Leg Press` (Primary), `Lying Leg Curl / Leg Extensions` (Choose 1), `Calf Raises` (If You Have Time). After synthetically reordering last session's `completedAt` to `Leg Press → Single Leg RDL → Seated Leg Curl` + `Leg Extensions → Lying Leg Curl` and picking `Last session` + Start: today's template correctly renders `Leg Press / Single Leg RDL / Seated Leg Curl` (Primary) and `Leg Extensions / Lying Leg Curl` (Choose 1). Section headers and calf-only If-You-Have-Time section unchanged. Zero console errors.

471. **Build.** `npx vite build --outDir /tmp/test-build` → 774.69 KB bundle / 208.59 KB gzipped (+2.55 KB / +0.75 KB vs Batch 34, accounted for by the gym row markup + Exercise Order row + reorder helper + the handleStartSession branch).

### Batch 34 (April 24, 2026) — Plate bar-change hotfix

User report: "the plate loaded bar weight does seem persistent, but what happens is sometimes a user will log a set and then realize that the plate loaded bar setting is wrong and change it. It doesn't apply that change to the volume calculated in the already programmed set." Reproduced: when a set exists in plate mode but has no `plates` breakdown (e.g. set was logged pre-plate-mode and plate mode was toggled on after, or an edge-case init state), the popover's `onBarChange` / `onMultChange` skipped that set entirely via `if (!s.plates) return s`. The display meanwhile was happily showing `calcTotal(emptyPlates(), exercise.barDefault, mult)` = just the current bar, while storage kept the old `weight` string. Result: display "25 lbs" with stored weight "225" — the bar change visibly updated the UI but the saved volume number didn't move. User saw the display change; didn't realize the stored value hadn't.

459. **`onBarChange` always stamps every set** (`BbLogger.jsx` PlateConfigPopover at `:1622`). Replaces `if (!s.plates) return s` with an always-apply path: seeds `plates: emptyPlates()` when absent, stamps `barWeight: newBar`, recomputes `weight: String(calcTotal(plates, newBar, mult))`, stamps `plateMultiplier: mult`. After the change, every set has a consistent `plates + barWeight + weight + plateMultiplier` quadruple and display / storage / saved volume all agree.

460. **`onMultChange` mirrors the pattern** (`:1638`). Same always-apply: `plates: s.plates ?? emptyPlates()`, `barWeight: s.barWeight ?? exercise.barDefault ?? 45`, recomputes weight under the new multiplier.

461. **Inline `onToggleMultiplier` at SetRow render site** (`:1842`). Same always-apply treatment; previously duplicated the same `if (!s.plates) return s` skip.

462. **Tradeoff accepted.** For a set that somehow entered plate mode with a typed weight like `'225'` but no plate breakdown, the first bar-or-mult change recomputes `weight` to just-the-bar (losing the 225). Pre-fix the 225 was already visually lost (display showed bar-only via emptyPlates fallback) — this fix just syncs storage with what was already on screen. User can tap the plate picker to re-seed the real breakdown anytime.

463. **Verified live** (preview, mobile 375×812). Scenario 2 seed: Leg Press w/ plate mode on, set `{weight:'225', reps:'10'}` + no plates. Opened popover → tapped 25 lb → stored set became `{weight:'25', plates: emptyPlates(), barWeight:25, plateMultiplier:2, reps:'10', type:'working'}` — display "Total 25 lbs" matches stored. Scenario 3 (set with plates 45×1): bar 45→25 still correctly updates stored weight 135→115 via the existing path. Both paths now uniform.

464. **Build.** `npx vite build --outDir /tmp/test-build` → 772.14 KB bundle / 207.84 KB gzipped (+0.57 KB / +0.10 KB vs Batch 32, accounted for by three tiny handler rewrites + comments).

### Batch 36 (April 24, 2026) — Superset feature

User-requested mid-session superset feature. Tap the new SS chip on any expanded exercise to pair it with up to 2 partners; cycle through them set-by-set in focus mode; rest timer fires only at round boundaries. Memory of past pairings surfaces via the chip's illuminated state on future sessions, with a one-tap Re-pair shortcut nested inside the same chip's sheet. Decision recap from the planning round: chip label "SS"; everything nested behind the chip (no extra UI surfaces); rest timer per round; Done mid-superset drops current exercise from cycle but keeps remaining partners going.

472. **Two new optional fields on `session.data.exercises[i]` (additive, no persist bump).** `supersetGroupId?: string` — `'sg_${timestamp}'`, identifies the superset grouping; present on all 2–3 grouped exercises. `supersetOrder?: number` — 0/1/2, cycle order with 0 = trigger. UI-only flag `supersetActive?: boolean` — true while cycle is live; **stripped before save** in `buildExerciseData`. Round-trips through `saveActiveSession` ([BbLogger.jsx:2985](src/pages/log/BbLogger.jsx)) automatically — same pattern as Batch 19's `equipmentInstance`. Pre-Batch-36 sessions simply lack the fields.

473. **`getMostRecentSupersetPartners(sessions, exerciseIdOrName)` in `helpers.js`.** Scans bb-mode sessions newest-first, matches by `exerciseId` (with name fallback for pre-v3 safety), returns `{partners: string[], date: isoString}` for the most recent session where the exercise was in a `supersetGroupId` with at least one other member. Null when no prior superset history. Drives the chip's illuminated state + the Re-pair shortcut copy.

474. **`SupersetSheet.jsx` (new component, `src/components/`).** Portal at z-245 (between RecommendationSheet 250 and PlateConfigPopover 220). Three variants:
    - **Initiate** (no prior history, not active): partner picker grouped by section, max 2 selectable, third attempt silently blocked + "Superset capped at 3" hint, Begin disabled until ≥1 selected.
    - **Re-pair** (prior history, not active): "Last time (Apr 23) you paired Pec Dec with Incline DB Press + Cable Fly" + Re-pair primary CTA + Customize secondary (expands picker). Re-pair greys when any prior partner is missing from today's workout, with "Missing: …" copy.
    - **Active** (live cycle): cycle order display + End superset CTA.
    Backdrop tap + Escape dismiss. All hooks called unconditionally before the early `if (!open) return null` to satisfy Rules of Hooks.

475. **Cross-exercise field-ref map at parent BbLogger** (`exerciseFieldRefs = useRef({})`). Shape: `{[exerciseId]: {weight: HTMLInputElement[], reps: HTMLInputElement[]}}`. `registerFieldRef(exId, setIdx, field, el)` callback threaded into ExerciseItem → SetRow/PlateSetRow's `weightRef` / `repsRef` props, composed with the existing local-ref assignment so both layers stay populated. Auto-nulls on unmount via React's callback-ref convention. Powers programmatic cross-card focus that the cycle handler triggers.

476. **`exercisesRef` at parent BbLogger** updated each render. Superset handlers read fresh state synchronously via `exercisesRef.current` rather than through a `setExercises` updater closure — the closure-mutated locals (target id, set index) aren't visible to the surrounding rAF/focus scheduling under React 18 batched dispatch, so this pattern decouples reads from writes.

477. **`handleBeginSuperset(triggerExId, partnerIds)` at parent BbLogger.** Generates a fresh `groupId`, validates partners (max 2, must exist in current state, not the trigger), tags trigger+partners with `supersetGroupId` + ascending `supersetOrder` + `supersetActive: true`, and immediately focuses the trigger's next empty-set weight input via `focusTargetSet`. If the trigger's tail set has values, appends a fresh empty one; otherwise reuses the existing empty tail.

478. **`handleSupersetCycle(currentExId)` at parent BbLogger.** Reads cur state via ref, sorts active non-done members by `supersetOrder`, finds current's slot, computes `nextIdx = (currentIdx + 1) % members.length`. Round-boundary detected when `nextIdx === 0` → fires rest timer if `autoStartRest`. If current isn't in members (just-marked-done case), advances to lowest-order remaining member as a graceful fallback. Reuses the target's empty tail set when present; otherwise appends. `<2 members` branch auto-ends the cycle (clears `supersetActive` on all members; keeps `supersetGroupId` for history).

479. **`handleEndSuperset(groupId)` at parent BbLogger.** Single `setExercises` mapping that flips `supersetActive: false` on every member of the group while preserving `supersetGroupId` + `supersetOrder` so the saved session retains the grouping for future-session memory.

480. **`handleSupersetDone(currentExId)` at parent BbLogger.** Marks the exercise done + completedAt; if it was in an active superset and ≥1 partner remains active+!done, rAF-defers a `handleSupersetCycle` call to advance focus to the next member. If no partners remain, clears `supersetActive` on all group members.

481. **`focusTargetSet(exId, setIdx, plateLoaded)` helper at parent BbLogger.** Multi-tick focus retry — rAF-rAF (post-commit) plus setTimeout 0/50/120ms fallbacks. React 18's concurrent commits split rendering across multiple frames; without the longer fallback the new SetRow's ref hadn't been registered yet by the time rAF fires. Once registered, `el.focus()` succeeds; subsequent attempts no-op via the `landed` ref.

482. **`buildEmptySetFor(exercise)` module-level factory** (`BbLogger.jsx`). Always `type: 'working'` per Batch 28 (set 2+ is always working). Respects plate-loaded state by seeding `plates`/`barWeight`/`plateMultiplier` from the exercise's current config.

483. **ExerciseItem accepts new props.** `allExercises` (for the picker), `onBeginSuperset`/`onEndSuperset`/`onSupersetCycle`/`onSupersetDone` (parent handlers), `registerFieldRef` (parent ref-map registration). Threaded from BbLogger render loop's ExerciseItem call site.

484. **SS chip in toolbar row** (after Machine chip, ~[BbLogger.jsx:1797](src/pages/log/BbLogger.jsx#L1797)). Three states, all indigo-themed:
    - Idle no-history: `bg-item text-c-faint border border-dashed border-white/10`, label `SS`.
    - Idle prior-history (illuminated): `bg-indigo-500/20 border border-indigo-500/40 text-indigo-300`, label `SS`.
    - Active in cycle: `bg-indigo-500/35 border border-indigo-500/60 text-white font-bold`, label `SS ×N` (member count).
    Always visible (always in toolbar row) — no conditional gating. Title attribute reflects the current state.

485. **Override `onAdvance` and `onDone` in superset mode** (ExerciseItem set rendering loop). When `inActiveSuperset` (derived from `supersetGroupId && supersetActive`), `onAdvance` calls `onSupersetCycle?.(exercise.id)` instead of the usual `addSet(true)`; `onDone` calls `onSupersetDone?.(exercise.id)` instead of the usual `stableMarkDone`. The "Mark as Done" CTA at the bottom of the expanded card also routes through `onSupersetDone` in superset mode.

486. **Effective expand + focus-collapse exemption** for superset members (ExerciseItem). `effectiveExpanded = expanded || inActiveSuperset` so partner cards stay mounted with their inputs in DOM (the cycle handler's cross-card focus would otherwise no-op against unmounted refs). `focusCollapsed = numpadOpen && !ownsActiveField && !supersetExempt` — non-superset cards still collapse, but partner members remain rendered so cross-card focus works.

487. **`buildExerciseData` strips `supersetActive`, persists `supersetGroupId` + `supersetOrder`.** Spread the two persistent fields conditionally; the in-session UI flag is intentionally absent from saved sessions so future-session memory lookups work cleanly without a stale "active" hint.

488. **Verified live in preview** (mobile 375×812, debug-backup.json).
    - **Initiate flow**: SS chip on Pec Dec renders dashed-muted "SS"; tapping opens "Initiate superset" sheet with 12 partner options grouped by section. Selecting Incline DB Press + Cable Fly → Begin assigns `supersetGroupId: sg_<ts>`, orders 0/1/2, all `supersetActive: true`. SS chip flips to "SS ×3" active state. Pec Dec's weight input auto-focuses.
    - **Cycle flow**: Setting Pec Dec to 185×10 + invoking onSupersetCycle → focus moves to Incline DB Press's weight input. Setting 100×12 + cycle → Cable Fly. Setting 60×15 + cycle → wraps back to Pec Dec set 2; rest timer fires (round boundary, `restEndTimestamp` flips from null to a future epoch). Round 2 begins.
    - **End flow**: Tapping SS chip on active member opens "Active superset" variant with End button; End flips `supersetActive: false` on all 3, preserves `supersetGroupId` + `supersetOrder` for history.
    - **Re-pair flow**: Injecting a fake completed session with the 3-member superset history → next session's SS chip renders illuminated (`bg-indigo-500/20 border-indigo-500/40 text-indigo-300`) with title "Last paired with Incline DB Press + Cable Fly". Tapping opens "Superset" variant with "Last time (Apr 23) you paired …" + Re-pair primary CTA + Customize secondary.
    - Zero console errors throughout (one Rules-of-Hooks violation caught + fixed during dev: the `if (!open) return null` early return in SupersetSheet now lives below all useState/useEffect/useMemo calls).

489. **Build.** `npx vite build --outDir /tmp/test-build` → 785.61 KB bundle / 212.05 KB gzipped (+10.92 KB / +3.46 KB vs Batch 35, accounted for by SupersetSheet component + parent handlers + chip + helper).

### Batch 37 (April 25, 2026) — Hybrid Training v1 foundation: library schema + 8-station catalog

First batch of the Hybrid Training v1 workstream (B37–B46) — see `hybrid-training-design-v1.md` and `hybrid-training-implementation-plan.md`. Adds the dimension model and type system to library entries WITHOUT changing any UI surface. The library list still renders the same cards; behind the scenes every entry now has a `type` and a `dimensions` array, and 8 HYROX station entries get auto-seeded. Foundation pass — every existing flow continues to work identically; the schema is just richer for B38+ to build on.

490. **`src/data/hyroxStations.js` (new)** — closed catalog of 8 HYROX stations per design doc §3: `sta_skierg`, `sta_sled_push`, `sta_sled_pull`, `sta_burpee_broad`, `sta_row`, `sta_farmers`, `sta_sandbag_lunges`, `sta_wall_balls`. Each carries `id`, `name`, `dimensions[]` (locked per station), `raceStandard` (canonical race-day target — used by B42's Start HYROX overlay). `buildHyroxStationLibraryEntry(station)` exported as the canonical converter — consumed by both `buildBuiltInLibrary` (fresh installs) and `migrateLibraryToV8` (returning users on v7→v8 upgrade). Stations carry `primaryMuscles: ['Full Body']` (truthful for compound HYROX work — the 14-muscle taxonomy doesn't really map) and `equipment: 'Other'` (per-gym variation already handled by Batch 19's equipment instance string). Type=`hyrox-station`; the user can't create a 9th in v1.

491. **`classifyType(name)` in `helpers.js`.** Keyword-based type prediction returning one of the 4 type values: `weight-training | running | hyrox-station | hyrox-round`. Mirrors `classifyRepRange` structure — ordered map, first match wins, defaults to `'weight-training'`. Order matters: composite-round terms ("hyrox round", "run + skierg") fire BEFORE station singletons ("skierg", "sled push") so "Run + SkiErg Round" classifies as hyrox-round even though "skierg" alone is hyrox-station. 22-keyword `TYPE_KEYWORD_MAP` covers the 8-station catalog + composite rounds + running keywords (easy run, treadmill, incline walk, easy bike, jog, etc.).

492. **`defaultDimensionsForType(type)` in `helpers.js`.** Returns dimension preset per type: weight-training → `[weight+reps]`; running → `[distance+time+intensity?]`; hyrox-station fallback → `[time]` (catalog stations have locked dims, this fires only on a custom station — shouldn't happen in v1); hyrox-round → `[]` (round templates use `roundConfig`, not `dimensions`, per B38). Five axes: weight | reps | distance | time | intensity. `required: true` gates set/round completion; `unit` is descriptive — canonical conversion via §11.

493. **`predictExerciseMeta` extended** (`helpers.js`). Now returns `{ primaryMuscles, equipment, type }` (was just `{primaryMuscles, equipment}`). Type-only fallback: when no muscle/equipment keyword fires but `classifyType(name)` returns a non-default type (e.g. "Easy Run" → 'running'), returns `{ primaryMuscles: [], equipment: 'Other', type }` so CreateExerciseModal still gets the type cue. `EXERCISE_KEYWORD_MAP` extended: 9 new HYROX-station entries at the top (specific-first ordering — must match before generic "row"/"press" fallbacks), and every existing entry gains `type: 'weight-training'` for explicitness.

494. **`migrateLibraryToV8(library)` in `helpers.js`.** Idempotent v7→v8 library migration. Pass 1: every existing entry gains `type` (default 'weight-training') and `dimensions` (default per type). Pass 2: seeds the 8 HYROX stations IF their canonical ids aren't already present — a user who manually created an exercise with id `sta_skierg` predates v8 and wins. Returns same array reference when nothing changes (matches `migrateLibraryToV6` / `V7` pattern). Defensive against null/undefined/non-array inputs.

495. **Persist version bumped `7 → 8` (`useStore.js`).** New `if (version < 8)` block appended to the migrate chain. `importData` chains `migrateLibraryToV8` after the v6 + v7 calls so pre-v8 backups land in the current schema. `buildBuiltInLibrary()` now emits `type: 'weight-training'` + `dimensions` on every lift entry, then concatenates the 8 HYROX stations via `HYROX_STATIONS.map(buildHyroxStationLibraryEntry)`. Fresh installs get the full 117-entry library (109 lifts + 8 stations) immediately on first load.

496. **`hybrid-b37-sanity.mjs` at worktree root.** 124/124 pass. Covers: classifyType across 25 cases (4 types + edge cases — empty string, null, non-string defaults to weight-training); defaultDimensionsForType across 5 cases; predictExerciseMeta extension across 10 cases (existing behavior preserved + station/round type-only fallback + DB Shrug Traps regression check from Batch 26); HYROX_STATIONS catalog integrity (8 stations, each with id/name/dimensions/raceStandard, all `sta_*` convention); buildHyroxStationLibraryEntry shape; migrateLibraryToV8 across synthetic v7 input + idempotency (re-run returns same reference) + user-collision case (pre-existing custom `sta_skierg` preserved, only 7 new stations seeded) + defensive cases (null/undefined/non-array). Real-data spot check section gracefully skips when `workout-backup-2026-04-24.json` isn't in the worktree.

497. **No UI surface this batch, no preview verification.** Per design — B37 is the data-layer foundation. UI work lands in B39 (library list type-axis filter + create modal) and B41+ (HYROX section preview, Start overlay, round logger, summary, hybrid finish). Build passes (`npx vite build --outDir /tmp/test-build` → 794.79 KB bundle / 214.28 KB gzipped, +9.18 KB / +2.23 KB vs Batch 36 — accounted for by the 22-entry TYPE_KEYWORD_MAP, the 9 new HYROX entries on EXERCISE_KEYWORD_MAP, the 8-station catalog, and the migrateLibraryToV8 + classifyType + defaultDimensionsForType helpers).

### Batch 38 (April 25, 2026) — Hybrid Training v1 foundation: session schema + unit conversions + v8→v9 migration

Second batch of the Hybrid Training v1 workstream (B37–B46). Adds the dimension-aware session schema and the unit-conversion layer that lets HYROX features read metric values without conversion lag (race-pace coaching, race-weight rehearsal, future unit-toggle UI). Like B37, no UI surface lands this batch — the schema is invisible to existing flows; B41+ surfaces consume it.

498. **Unit-conversion helpers (`helpers.js`).** Four pure functions + two exported constants:
    - `LBS_TO_KG = 0.45359237`, `MILES_TO_METERS = 1609.344`.
    - `lbsToKg(lbs)` / `kgToLbs(kg)` — round to 3-decimal precision per design doc §11.2. Defensive against null / undefined / non-numeric input (returns null).
    - `milesToMeters(mi)` — returns integer meters per §11.2 (no fractional meters in storage).
    - `metersToMiles(m)` — 3-decimal precision so `1609 → 1.000`, `500 → 0.311`.
    Mirrors the existing helper pattern (no class, no shared state, defensive guards on every input).

499. **`migrateSessionsToV9(sessions)` in `helpers.js`.** Walks every session's exercises' sets — top-level + nested drops — and adds derived `weightKg` (and `rawWeightKg` when present) alongside the existing lbs fields. Idempotent: sets that already carry `weightKg` are skipped, so re-running on v9 data is a no-op (returns same array reference if no fields were added). Defensive against null / non-array / malformed inputs. No HYROX rounds (`session.data.exercises[].rounds[]`) exist pre-v9 yet — those get written natively in v9-shape from B43 onward.

500. **`migrateCardioSessionsToV9(cardioSessions)` in `helpers.js`.** Walks cardio sessions; for entries with `distanceUnit === 'miles'` (Running / Walking / Treadmill per CardioLogger's `getDistanceUnit`), adds derived `distanceMiles` + `distanceMeters`. Other units (`'floors'` for Stairmaster; `null` for Bike / custom types) are left as-is — they're not length axes. Idempotent.

501. **LoggedSet + LoggedHyroxRound shapes documented inline (`helpers.js`).** Block comment above the v9 migration documents the dimension-aware schema:
    - **`LoggedSet`** gains optional `weightKg`, `rawWeightKg`, `distanceMiles`, `distanceMeters`, `timeSec`, `intensity`. Pre-v9 sets without these fields render as before (back-compat).
    - **`LoggedHyroxRound`** — new type, lives inside `LoggedExercise.rounds[]` for type=`hyrox-round` library entries. Shape: `{ roundIndex, legs: [{type:'run', distanceMiles, distanceMeters, timeSec, completedAt}, {type:'station', stationId, distanceMeters?, timeSec?, weight?, weightKg?, reps?, completedAt}], restAfterSec, completedAt }`. v1 locks `legs[]` to length 2 (run → station); §4.5 generalizes to N legs for future round structures.
    - **`LoggedExercise`** for type=`hyrox-round` carries `rounds: LoggedHyroxRound[]` + session-level prescription overrides (`prescribedRoundCount`, `prescribedStationId`, `prescribedRunDistanceMeters`).
    No code creates these shapes yet — B43 is the first batch to write a `LoggedHyroxRound`. Documenting now keeps B43+ surfaces aligned.

502. **Persist version bumped `8 → 9` (`useStore.js`).** New `if (version < 9)` block runs `migrateSessionsToV9(persistedState.sessions)` + `migrateCardioSessionsToV9(persistedState.cardioSessions)`. Additive — no fields removed, no schema break for existing UIs. `importData` chains v9 after v5 on imported sessions and after the cardio array assignment so pre-v9 backups land in the current schema.

503. **`hybrid-b38-sanity.mjs` at worktree root.** 50/50 pass. Covers: conversion-constant accuracy (`LBS_TO_KG` to 9 decimals, `MILES_TO_METERS` to 9 decimals); `lbsToKg` / `kgToLbs` across 8 + 4 cases (real values, 0, null/undefined, numeric strings, non-numeric); `milesToMeters` / `metersToMiles` across 7 + 4 cases (1mi, 0.5mi, 5K-distance, SkiErg-500m, edge cases); `migrateSessionsToV9` synthetic v8 data covering warmup/working/drops/unilateral with weight + rawWeight; idempotency (same reference returned); defensive handling (null/undefined/non-array/empty/malformed); `migrateCardioSessionsToV9` with mixed units (miles/floors/null/null-distance); idempotency; real-data spot-check section gracefully skips when backup absent.

504. **No UI surface this batch, no preview verification.** Per design — B38 is the second data-layer foundation pass. The full 117-entry library + the new dimensioned-set schema are now both in place; B39 lights up the first visible UI (library list type-axis filter + create-exercise modal type selector). Build passes (`npx vite build --outDir /tmp/test-build` → 796.64 KB bundle / 214.73 KB gzipped, +1.85 KB / +0.45 KB vs Batch 37 — accounted for by the four conversion helpers + the two v9 migrations).

### Batch 39 (April 25, 2026) — Hybrid Training v1: library list type-axis + CreateExerciseModal + ExerciseEditSheet type-aware

Third batch of the Hybrid Training v1 workstream and the **first user-visible UI** of the workstream. Surfaces the type system from B37 across the three library-management screens. Branched on `claude/hybrid-b39-library-ui` and pushed to GitHub for Vercel preview review — NOT auto-merged to main.

505. **Type-aware display helpers (`helpers.js`).** Four pure functions:
    - `getTypeColor(type)` — brand color per design doc §12.4: weight-training → `#60A5FA` (blue-400), running → `#34D399` (emerald-400), hyrox-station + hyrox-round → `#EAB308` (yellow-500). Defaults to blue for legacy / null inputs.
    - `getTypeLabel(type)` — short uppercase label: WEIGHT / RUN / HYROX (both station + round share HYROX since they're a unit on the user's mental model).
    - `getTypeFilterBucket(type)` — maps the 4 type values onto the 3-axis filter (lift / run / hyrox). Both station + round → 'hyrox'.
    - `formatLastSetSummary(set, type)` — type-aware "last logged" summary text. Weight-training: `185 × 10` (uses `perSideLoad` for unilateral). Running: `1.2 mi · 12:30`. HYROX station: `100 lb · 50m` (depending on which dimensions are present). HYROX round: returns null (rounds[] not in the flat `set` shape — B43 ships the writer). Returns null when no fields are renderable.

506. **`addExerciseToLibrary` extended for type-aware validation (`useStore.js`).** Pre-Batch-39 the action assumed weight-training shape (rejected empty primaryMuscles + missing equipment). Now:
    - Accepts a `type` field; defaults to `'weight-training'` for back-compat.
    - Validates against the 4-type enum.
    - **Rejects `hyrox-station`** — the 8-station catalog is closed in v1 and auto-seeded by the v8 migration. Users can only edit existing entries, not create custom stations.
    - **Requires `roundConfig`** with at minimum a `stationId` OR `rotationPool` for `hyrox-round`. Other shapes are caught at edit time by the form, but we guard at the store too.
    - Skip-tagging path stays for `weight-training` only (running / hyrox-round bypass §3.2.1 muscle/equipment requirements; their dimensions don't apply).
    - Backfills `dimensions` via the existing `defaultDimensionsForType(type)` import when caller doesn't supply one.

507. **`CreateExerciseModal` rewritten with 4-option type selector (`src/components/CreateExerciseModal.jsx`).** Top of form gets a type chip row (Weight / Run / HYROX round / HYROX station), each chip painted in the type's brand color when selected. Form fields swap based on selected type:
    - **Weight**: existing fields unchanged (name, primaryMuscles, equipment, defaultUnilateral) + the existing 300ms predictExerciseMeta auto-fill. Skip-for-now stays exactly as before.
    - **Run**: name + equipment chip row (Treadmill / Outdoor / Bike / Other) + intensity-tracking checkbox + a "Distance and time logged per session" hint. No muscle groups (running entries default to `['Full Body']`).
    - **HYROX round**: name + station-mode toggle (Single station / Rotates from pool) → station chips from the 8-catalog (single-pick OR multi-select pool) → run-leg distance stepper (100m–5000m, 100m increments) → round-count chips (3/4/5/6/8) → rest-between-rounds chips (1:00 / 1:30 / 2:00 / 3:00). Save commits a roundConfig object: `{runDimensions: {distance: {default, unit:'m'}}, stationId | rotationPool, defaultRoundCount, defaultRestSeconds}`.
    - **HYROX station**: read-only catalog block. Lists the 8 pre-seeded stations + pointer to the HYROX filter on `/exercises`. Save button replaced with "Got it" → closes the modal.
    - Auto-predict on name now respects type: when the user hasn't touched the type chip and the predictor returns a non-default type, it pre-selects (e.g. typing "Easy Run" auto-flips type → Run after the 300ms debounce).

508. **`ExerciseEditSheet` extended for type display + roundConfig editor (`src/components/ExerciseEditSheet.jsx`).** Header eyebrow becomes `[TYPE]·SOURCE` — a brand-colored type badge (WEIGHT / RUN / HYROX) next to the existing Built-in / Custom label. Body fields gate on type:
    - **Weight-training**: existing form (muscles, equipment, load increment, rep range, unilateral) — unchanged.
    - **Running**: name + equipment chips (Treadmill / Outdoor / Bike / Other). Muscle groups + load increment + rep range hidden. Distance/time noted as session-only.
    - **HYROX station**: name editable, but a yellow read-only "Locked dimensions" panel below shows each axis (distance / time / weight / etc.) with required/optional + unit. Race standard (e.g. "1000 m") rendered beneath. Stations can't change their catalog dimensions, only their name (gym-specific machine instance variation lives on Batch 19's per-session chip, not the library row).
    - **HYROX round**: full roundConfig editor mirroring the create modal — single/pool toggle + station picker + run-leg stepper + round-count + rest-between-rounds. Save persists the rebuilt roundConfig.
    - Save-button enable rules + the helper hint copy ("Pick at least one muscle group…" etc.) are now type-conditional.

509. **`ExerciseLibraryManager` (`/exercises`) gets type-axis primary filter + row stripes (`src/pages/ExerciseLibraryManager.jsx`).** Major UI refresh:
    - **Type-axis filter chips** at the top: `All N · Lift N · Run N · HYROX N`. Selected chip uses the type's brand color (blue/green/yellow) instead of the user's accent. Hides chips with `0` count except All.
    - **Source-axis chips moved into a `⋯` overflow** next to the new + New / Done buttons. Portal popover with the 4 source filters + counts; outside-click + Escape dismiss.
    - **Per-row left-edge stripe** (3px) painted in the row's type color so the list scans visually by category at a glance.
    - **Inline type tag** right of the exercise name — small uppercase pill (e.g. `HYROX`) with the type's brand-color treatment.
    - **Type-aware "Last logged" summary** — feeds the entry's `type` into the new `formatLastSetSummary` helper. Weight entries continue to show `225 × 8`; HYROX/running entries fall back to nothing rendered until B43+ writes those dimensioned sets.
    - **`+ New` button** in the topbar opens `CreateExerciseModal`. Previously the only entry to the modal was via the BbLogger Add panel — now `/exercises` has its own creation entry.

510. **`hybrid-b39-sanity.mjs` at worktree root.** 45/45 pass. Covers: type color + label + filter-bucket maps across all 4 types + null/unknown defaults; formatLastSetSummary across weight-training / running / hyrox-station / hyrox-round shapes; classifyType + predictExerciseMeta integration spot-checks (Bench Press, Easy Run, SkiErg, HYROX Simulation Round, unknown name). Mirrors the existing hybrid-b37 / b38 sanity patterns.

511. **Branch + Vercel preview.** Pushed `claude/hybrid-b39-library-ui` to GitHub — Vercel auto-deploys a preview URL. **Not auto-merged to main**: this is the first user-checkable UI batch, the user reviews before promoting. Build clean (`npx vite build --outDir /tmp/test-build` → 813.38 KB bundle / 218.90 KB gzipped, +16.74 KB / +4.17 KB vs Batch 38 — accounted for by the rewritten CreateExerciseModal + the type-aware ExerciseEditSheet branches + the new ExerciseLibraryManager type filter + the SourceFilterOverflow portal + the four helpers in helpers.js).

**Batch 39 polish followups (post-review).** Four UI tweaks landed between B39 and B40 based on user feedback before the merge to main:

512. **`Lift` tab → `Weight Training`** (`ExerciseLibraryManager.jsx`). The original "Lift" label competed with the "WEIGHT" row chip — felt like two competing labels for the same concept. Renamed for consistency.
513. **WEIGHT row chip dropped from weight-training rows** (`ExerciseLibraryManager.jsx`). The 3px color stripe + the muscle/equipment line below already convey the type. HYROX and RUN rows keep their tag chip — they're the by-exception types where the explicit flag helps the eye against the unmarked default.
514. **Secondary filter row hidden when type filter ≠ All** (`ExerciseLibraryManager.jsx`). The workout-name + Logged/Never-logged chips became noise when narrowed to HYROX or Run. Now they only render on the All filter; once you narrow by type, the secondary axes collapse.
515. **`TYPE_LABELS['weight-training'] = 'WEIGHT TRAINING'`** (`helpers.js`). The ExerciseEditSheet header badge now reads `WEIGHT TRAINING · BUILT-IN`, matching the renamed tab. The row chip is gated to non-weight-training types post-#513 so the longer label doesn't bloat row layouts.

### Batch 40 (April 25, 2026) — Hybrid Training v1: Brooke JSON v3 + split-import library extension

Fourth batch of the Hybrid Training v1 workstream. Re-encodes Brooke's split as v3 (canonical roundConfig on every hyrox-round) and extends the split-import path to consume `type` fields by spawning library entries on import. Mostly invisible to the user — the visible result is just "Brooke's JSON imports cleanly + the new HYROX-round library entries appear in `/exercises`." The actual HYROX surfaces (preview card, Start overlay, round logger) arrive in B41+.

516. **`brooke-hybrid-split.json` v3** (repo root). Re-authored against the real schema. Three hyrox-round entries gain valid `roundConfig`: Tuesday "HYROX Run + SkiErg Round" → `{stationId: 'sta_skierg', runDistance: 800m, rounds: 4, rest: 120s}`; Friday "HYROX Simulation Round" → rotation pool of 7 stations (everything except Burpee Broad Jump per Brooke's program), runDistance: 1000m, rounds: 4, rest: 90s; Saturday "Wall Balls + 200m Run Round" → `{stationId: 'sta_wall_balls', runDistance: 200m, rounds: 3, rest: 60s}`. The 3 hyrox-station references in Thursday's "Active Rest & Light Skill" day were renamed to canonical catalog names (`Light Farmers Carry` → `Farmers Carry`, `Sled Push Technique` → `Sled Push`, `Sled Pull Technique` → `Sled Pull`) so they resolve to existing catalog entries. The "light only" / "technique focus" flavor moves into rec-note text.

517. **`importLibraryEntryFromSplit(exercise, library)` in `helpers.js`.** Pure decision helper. Returns `{create | existingId | skip | error}` per imported exercise. Catalog-closed `hyrox-station` names that don't match a seeded catalog entry return `error` (per plan §B40 "validate against the catalog… reject malformed"). `hyrox-round` without a roundConfig also errors. `running` entries get a fully-tagged shape (Full Body + equipment fallback). `weight-training` entries that don't match an existing library row get `needsTagging:true` so they land in `/backfill` for the user to finish later. Defensive against null/non-string inputs.

518. **`collectLibraryAdditionsFromSplit(splitData, library)` in `helpers.js`.** Walks every exercise across every workout/section, aggregates per-entry decisions, dedupes within the import payload by normalized name. Returns `{toCreate, errors}` for a single store transaction.

519. **`importSplitWithLibrary(data)` store action (`useStore.js`).** Wraps the three call sites' inline `addSplit({...splitData, isBuiltIn:false})` flow: validates payload shape, runs `collectLibraryAdditionsFromSplit`, fires `addExerciseToLibrary` for each new entry (catching per-entry failures so a single bad exercise doesn't abort the whole import), strips top-level `id`, calls the existing `addSplit`. Returns `{ok, split, libraryAdded, errors}`.

520. **3 import call sites updated** (`SplitManager.jsx`, `ChooseStartingPoint.jsx`, `Welcome.jsx`). Each preserves its site-specific pre-validation (for tailored error messages) but the inner `addSplit` becomes `importSplitWithLibrary(data)` + a `console.warn` of any returned errors. Welcome.jsx threads the result through the existing `setActiveSplit(newSplit.id)` chain via `result.split.id`.

521. **`hybrid-b40-sanity.mjs` at worktree root.** 65/65 pass. Covers: (1) brooke-hybrid-split.json v3 shape — 6 workouts, exact catalog station names, valid roundConfig on each hyrox-round; (2) `importLibraryEntryFromSplit` decision matrix across 8 input shapes; (3) `collectLibraryAdditionsFromSplit` dedup + error aggregation; (4) end-to-end empty-library import (pre-v8 user) — 3 hyrox-station errors + 3 rounds + 4 running + 27 weight-training creates; (5) end-to-end seeded-library import (post-v8 user) — 0 errors, stations resolve to catalog ids, all 3 round configs preserved through the import. Mirrors the existing hybrid-b37 / b38 / b39 sanity patterns.

522. **No UI surface this batch, no preview verification.** Per design — B40 is the JSON + import data layer. UI work for the HYROX flow lands in B41+. Build passes (`npx vite build --outDir /tmp/test-build` → 815.72 KB bundle / 219.67 KB gzipped, +2.34 KB / +0.77 KB vs Batch 39 — accounted for by the two helpers + the importSplitWithLibrary action + the three call-site wirings).

### Batch 41 (April 25, 2026) — Hybrid Training v1: HYROX section preview card in workout view

Fifth batch of the Hybrid Training v1 workstream and the **first user-visible HYROX surface**. Workout view at `/log/bb/:type` learns to render any section labeled "HYROX" (case-insensitive) as a single immersive yellow preview card per design doc §5.2 + §12.4 + mockup 1, instead of the standard exercise-card list. Lift sections continue to render as cards. Branch shipped to `claude/hybrid-b41-section-preview` as a feature branch awaiting user review — Start HYROX button is wired to a placeholder alert pending B42's overlay surface.

523. **`getLastHyroxRoundSession(sessions, exerciseIdOrName)` in `helpers.js`.** Walks completed bb-mode sessions newest-first, returns the most recent session that logged a hyrox-round exercise matching the given id or name, with derived `totalTimeSec` (sum of all leg `timeSec` + `restAfterSec` per round) + `roundCount`. Returns null when no prior session has rounds[]. Forward-compatible — pre-B43 (no rounds[] writer yet) the helper returns null on real-world data, but the synthetic-rounds sanity proves it'll work once B43 ships the writer.

524. **`formatDuration(sec)` in `helpers.js`.** Renders `M:SS` for sub-1-hour durations and `H:MM:SS` for longer. Defensive against null / NaN / negative inputs (returns empty string). Used by the HYROX preview card's last-session summary line + the future round-logger gym clock display.

525. **`HyroxSectionPreview.jsx` (new, `src/pages/log/`).** Renders the HYROX section as a stack of yellow gradient-wash cards — one per hyrox-round exercise in the section. Each card has:
    - Yellow uppercase "HYROX" section label above (matches design doc §12.4 yellow takeover).
    - Yellow gradient-wash card background (`linear-gradient(135deg, rgba(234,179,8,0.12) 0%, rgba(0,0,0,0.4) 65%)`) with a 1px yellow inset shadow.
    - `INTERVALS · N ROUNDS` chip (yellow on yellow-tinted bg) above the round-template name.
    - Three black-tinted prescription tiles in a row: `RUN LEG {distance}{unit}` / `STATION {name|Rotates(N)}` / `REST {M:SS}`.
    - Last-session summary in the top-right (`Last: 16:45 · 4 rounds`) when prior history exists; hidden otherwise.
    - Yellow filled `Start HYROX →` button with black text spanning the card width.
    - Done state (when `exercise.rounds.length >= prescribedRoundCount`): muted dark card with `✓ done · {totalTime} · {delta} vs last`. Tap re-routes (B45 will wire to summary surface). Pre-B43 this state is unreachable in normal flow since no rounds get written yet.
    - Rotation-pool stations show `Rotates (N)` with a 2-station preview subtitle.

526. **BbLogger.jsx render-loop wiring** (`src/pages/log/BbLogger.jsx:4061+`). Surgical insertion at the top of each `renderGroups` iteration: when `group.label.trim().toLowerCase() === 'hyrox'` AND it's not active-superset / completed, render `<HyroxSectionPreview exercises={groupExes} sessions={sessions} onStart={...} />` and return. Otherwise fall through to the existing GroupLabel + card-list render. Start HYROX click currently fires a placeholder alert `"Round logger ships in Batch 42"` — B42 wires the actual `/log/hyrox/:exerciseId/start` overlay route.

527. **`hybrid-b41-sanity.mjs` at worktree root.** 31/31 pass. Covers: `getLastHyroxRoundSession` across 9 input cases (null / empty / non-bb sessions / missing rounds / id match / name fallback / newest-first ordering / total-time accumulation including restAfterSec / roundCount); `formatDuration` across 9 ranges (0 / 45s / 60s / 1005s / 3600s + null/NaN/negative defensive cases + half-second rounding); HYROX section detection predicate across 11 cases (label "HYROX" / "hyrox" / "  HYROX  " / "Hyrox" matches; "Lift" / "Primary" / "HYROX Round" / null label do NOT match; isCompleted + isActiveSuperset block).

528. **Live preview verified** (port 5174, mobile 375×812). Synthetic Brooke-like split imported via the file-input flow with a Tuesday workout containing both a Lift section (3 weight-training exercises) AND a HYROX section (1 hyrox-round entry, SkiErg @ 800m / 4 rounds / 2:00 rest). Result: `/log/bb/tue_test` renders with yellow-uppercase "HYROX" header below the Lift cards, the immersive yellow preview card with INTERVALS · 4 ROUNDS chip, "HYROX Run + SkiErg Round" title, three tiles (RUN LEG `800m` / STATION `SkiErg` / REST `2:00`), and the yellow `Start HYROX →` button. Lift section above renders as standard exercise cards (Cable Lateral Raise / DB Lateral Raise / Reverse Flies). Tap Start HYROX → alert("Round logger ships in Batch 42…"). Zero new console errors (the existing key-via-spread warnings on `<ExerciseItem>` predate B41 and aren't introduced by this change).

529. **Build.** `npx vite build --outDir /tmp/test-build` → 821.18 KB bundle / 221.23 KB gzipped (+5.46 KB / +1.56 KB vs Batch 40, accounted for by the new HyroxSectionPreview component + getLastHyroxRoundSession + formatDuration helpers + the BbLogger render-loop branch).

530. **Branch + handoff.** Pushed `claude/hybrid-b41-section-preview` to GitHub. **Not auto-merged to main** — first user-visible HYROX surface, awaiting your eyes. The Start HYROX button is wired to a placeholder alert; B42 ships the actual Start HYROX overlay (cycling headline + prescription editor + Begin round 1 → B43 round logger).

### Batch 42 (April 25, 2026) — Hybrid Training v1: Start HYROX overlay + 30-headline bank + station picker

Sixth batch of the Hybrid Training v1 workstream and the second user-visible HYROX surface. Replaces the B41 placeholder alert with the real Start HYROX overlay (mockup 2). Pre-populates today's prescription based on the round template's roundConfig defaults; the user can override any of the four rows (rounds / run leg / station / rest) before tapping Begin round 1, which routes to a B43 stub on `/log/hyrox/:exerciseId/round/1/run` carrying the prescription via `location.state`. Branch shipped to `claude/hybrid-b42-start-overlay` — NOT auto-merged to main, awaiting user review.

531. **`src/data/hyroxHeadlines.js` (new)** — closed bank of 30 short cycling headlines per design doc §13.1. Three tiers: serious / locked-in (10 entries — "Lock in.", "Earn it.", "Pace, not panic."), coach voice / motivating (10 — "You against last week.", "Smooth is fast.", "Make her sweat."), playful / light (10 — "Time to suffer fluently.", "Cardio o'clock.", "Sprint now, brunch later."). All entries ≤ 50 chars per design doc §13.3 two-line max. No duplicates. Single source of truth for the overlay's hero text.

532. **`pickHeadline(lastShownIndex)` in `helpers.js`.** Pure function per design doc §13.2. `do { idx = Math.floor(Math.random() * bank.length) } while (idx === lastShownIndex)` — guarantees no consecutive repeat across opens. Returns `{ text, index }` so the caller can persist the index for next-open's avoidance check. Defensive: bank.length=0 returns `{text:'',index:-1}`; bank.length=1 short-circuits before the loop. First-time call (lastShownIndex=-1) treats every index as fair game.

533. **`pickHyroxStationForToday(roundConfig, sessions, exerciseIdOrName)` in `helpers.js`.** Pure function per design doc §5.3 (Option A — user picks at session start, with light freshness bias to surface variety). Single-station rounds (`roundConfig.stationId`) return that id directly. Pool rounds walk newest-first through up to 2 prior sessions of the same template, build a `stationId → most-recent-timestamp-used` map, then return the FIRST pool member NOT used in that recency window. If every pool member was used recently (small pool), falls back to least-recently-used. No history → first pool entry. Name-fallback resolution for pre-v3 sessions (`exerciseIdOrName` matches by `exerciseId` OR by `name`). Defensive against null roundConfig / empty pool / non-array sessions / falsy `stationId` + present `rotationPool` (uses pool path).

534. **`settings.lastHyroxHeadlineIndex` + `setLastHyroxHeadlineIndex(idx)` (`useStore.js`).** Default `-1` (sentinel = none yet). Setter validates numeric input + persists via the existing settings deep-merge. The Start HYROX overlay reads it on mount, calls `pickHeadline`, then writes the picked index back via a `useEffect` (NOT during render — initial implementation called the setter inside `useState`'s lazy initializer and triggered React's "Cannot update a component while rendering" warning when Zustand subscribers in the App tree re-rendered; moved to a mount-only useEffect so the persist fires after commit).

535. **`StartHyroxOverlay.jsx` route component (`src/pages/log/StartHyroxOverlay.jsx`).** Full-page yellow-takeover overlay at `/log/hyrox/:exerciseId/start`. Resolves the round template by id (with name fallback) from `exerciseLibrary`. Renders:
    - Yellow radial glow at top via `radial-gradient(ellipse 70% 40% at 50% 0%, rgba(234,179,8,0.18) 0%, ... 0)` per §12.2.
    - Yellow context chip `HYROX · {round template name}`.
    - 34px / weight 500 / lh 1.05 / letter-spacing -0.02em hero headline with 200ms fade-in animation per §13.3.
    - Last-session bests inline below the chip when prior history exists (`Last: {totalTime} · {N} rounds`).
    - Four tap-to-edit `PrescriptionRow` cards: Rounds (chip 3/4/5/6/8), Run leg (− / + stepper, 100m–5000m, 100m steps), Station (read-only "Single-station round — fixed by template" OR rotation-pool chip selector), Rest between rounds (chip 1:00 / 1:30 / 2:00 / 3:00). Tap a row → expand inline editor below; tap a chip / step → commit the new value and auto-collapse.
    - Yellow `Begin round 1` button at the bottom with `0 8px 30px rgba(234,179,8,0.35)` glow shadow.
    - `Skip HYROX today` text-link below — calls `navigate(-1)`, returns to the workout page (Lift section preserved).
    - Defensive empty-state: when the route's exerciseId can't resolve to a hyrox-round library entry, renders "Round template not found." + Go back button instead of crashing.

536. **`HyroxRoundLoggerStub.jsx` placeholder route component.** Renders at `/log/hyrox/:exerciseId/round/:roundIdx/:leg`. Shows "Round logger ships in Batch 43", a JSON dump of the `location.state.prescription` (proves the prescription threaded through correctly: `exerciseId / roundCount / runDistanceMeters / restSec / stationId`), and a Go back button. B43 replaces this entire component.

537. **App.jsx route registration.** Two new routes added between `/log/bb/:type` and `/cardio`: `/log/hyrox/:exerciseId/start` → `StartHyroxOverlay`; `/log/hyrox/:exerciseId/round/:roundIdx/:leg` → `HyroxRoundLoggerStub`.

538. **BbLogger.jsx onStart wire.** Replaced B41's placeholder alert with `navigate('/log/hyrox/${encodeURIComponent(ex.exerciseId || ex.name)}/start')`. The exerciseId-or-name fallback covers pre-v3 sessions; encodeURIComponent handles spaces and special chars in round-template names. The case-insensitive `group.label.trim().toLowerCase() === 'hyrox'` gate from B41 is preserved verbatim — non-HYROX sections continue to render the standard card list.

539. **Fullscreen-flow predicate extended for HYROX routes.** `BottomNav.jsx`, `HamburgerMenu.jsx` both gain `path.startsWith('/log/hyrox/')` to the existing `isFullscreenFlow` check that already covered `/log/bb/`, `/splits/new*`, `/splits/edit*`. `RestTimer.jsx` also hides on `/log/hyrox/*` per design doc §5.4 — HYROX gets its own gym-clock timer (B43) and a full-screen yellow rest countdown between rounds (B44), so the floating circle would be redundant chrome behind the overlay.

540. **`hybrid-b42-sanity.mjs` at worktree root.** 33/33 pass. Covers: (1) HYROX_HEADLINES bank integrity (30 entries, all non-empty strings ≤ 50 chars, no duplicates, three tier anchors present); (2) pickHeadline 200-trial non-repeat verification + 500-trial coverage check (≥25 of 30 distinct indices hit); (3) pickHyroxStationForToday across single-station / no-history pool / pool with prior history (least-recently-used wins) / small-pool-everyone-used-recently / defensive cases (null roundConfig, empty rotationPool, non-array sessions, falsy stationId-with-pool) / name-fallback resolution; (4) end-to-end seed correctness for both Brooke round templates (Tuesday Run+SkiErg → roundCount=4, runDistance=800m, rest=120s, SkiErg; Friday Simulation → roundCount=4, runDistance=1000m, rest=90s, first-not-recently-used pool member).

541. **Live preview verified** (port 5176, mobile 375×812, synthetic Brooke split with both round templates injected via localStorage). Tuesday Run+SkiErg overlay: yellow chip `HYROX · HYROX RUN + SKIERG ROUND`, headline rotates each open ("Lock in.", "One round at a time.", "Run it back."), 4 prescription rows pre-populated correctly (4 rounds / 800m / SkiErg / 2:00), Begin round 1 routes to `/log/hyrox/HYROX%20Run%20%2B%20SkiErg%20Round/round/1/run` with state `{exerciseId, roundCount: 4, runDistanceMeters: 800, restSec: 90 (overridden from 120 via the chip), stationId: "sta_skierg"}`. Friday Simulation overlay: separate headline ("Make her sweat."), pool sublabel "Rotates from pool (7)" under SkiErg, expanding the Station row reveals all 7 pool members (SkiErg / Sled Push / Sled Pull / Rowing / Farmers Carry / Sandbag Lunges / Wall Balls), tapping Sled Push overrides the prescription. Begin round 1 routes through with `stationId: "sta_sled_push"` correctly persisted in `location.state`. BottomNav + RestTimer + HamburgerMenu correctly hidden on the overlay route. Console clean of new errors — the only remaining errors are the pre-existing `<ExerciseItem>` key-via-spread warnings from BbLogger.jsx that predate B41 and are documented as known caveats.

542. **Build.** `npx vite build --outDir /tmp/test-build` → 832.19 KB bundle / 224.63 KB gzipped (+11.01 KB / +3.40 KB vs Batch 41, accounted for by StartHyroxOverlay component + 30-headline bank + pickHeadline + pickHyroxStationForToday + the route registration + the HyroxRoundLoggerStub placeholder).

### Batch 43 (April 25, 2026) — Hybrid Training v1: round logger + gym-clock timer + intra-leg comparison

Seventh batch of the Hybrid Training v1 workstream and the **third user-visible HYROX surface**. Replaces B42's `HyroxRoundLoggerStub.jsx` placeholder with the real per-leg round logger (mockup 3): gym-clock digital timer per design doc §17 + intra-leg comparison band per §14.1 + Done · Stamp time auto-stamp per §5.4. Branch shipped to `claude/hybrid-b43-round-logger` — NOT auto-merged to main, awaiting user review.

543. **Four new station-anchored helpers in `helpers.js`** (the headline B43 invariant per §14.1: stations are the comparison primitive, not round positions). All pure functions, defensive against null / non-array inputs, used by `HyroxRoundLogger`'s comparison band:
    - `getStationHistory(sessions, stationId, dimensions = {})` — newest-first array of every station leg matching stationId across all sessions and round templates. Optional `{distanceMeters, weight, reps}` filters narrow to exact-dimension matches. Each result echoes `sessionId / sessionDate / exerciseId / exerciseName / roundIndex` so callers can build "vs your last SkiErg at 500m on Tuesday" framings. Sort: newest-first by sessionDate; **descending roundIndex within the same session** so within a single workout the LATER rounds (which happened more recently) rank ahead of earlier ones — required for "most recent leg" semantics.
    - `getRunLegHistory(sessions, distanceMeters)` — same shape, run legs only. Cross-template aggregation by design.
    - `computePaceFromHistory(history)` — average seconds per 100 meters across every prior leg with both `timeSec` AND `distanceMeters`. Dimension-agnostic pace fallback (§6.5). Returns null when history doesn't carry distance (e.g. wall-balls reps-only).
    - `buildIntraLegComparison({legType, stationId, stationName, distanceMeters, weight, reps, currentTimeSec, sessions})` — composes the comparison band data per §14.1. Returns `{mode: 'exact' | 'pace', status: 'ahead' | 'behind' | 'neutral', label, lastTimeSec, deltaSec, paceSecPer100m, paceProjectedTimeSec}` OR null on cold start (§14.4 — band hides). Mode-`exact` fires when today's dimensions match a prior occurrence; mode-`pace` fires as fallback when station has prior history but the dimension combo is novel; null fires when station has zero prior legs.

544. **`src/components/GymClock.jsx` (new)** — gym-clock digital timer per design doc §17. Three rectangular digit boxes (HRS / MIN / SEC) separated by colon glyphs, all wrapped in a black surround with 2px yellow border + inset glow + a label eyebrow. Pure presentational — parent owns the elapsed-seconds state and the 100ms tick. §17.1 spec exactly: `rgba(234,179,8,0.08)` digit-box bg, 56px min-width, 38px monospace `#FEF08A` numerals, 8px font-size yellow-700 label below. Colon: 32px monospace yellow-50% top-aligned. Wrapper: `inset 0 0 60px rgba(234,179,8,0.12)` glow. `mode='rest'` shifts the wash to a more subdued `rgba(234,179,8,0.04)` and the eyebrow defaults to "REST" (B44 will count down with this). `eyebrowOverride` prop lets the parent set "ROUND CLOCK" / "STATION CLOCK" / "RUN CLOCK" / "REST" explicitly.

545. **`src/pages/log/HyroxRoundLogger.jsx` (new — replaces stub)** — the per-leg surface. Routes at `/log/hyrox/:exerciseId/round/:roundIdx/:leg`. Reads `location.state.prescription` from B42's overlay on FIRST mount; subsequent mounts (reload, back-arrow round-trip) read from `activeSession.hyrox`. Initialization happens inside a `useEffect` (NOT during render) so the Zustand setter doesn't fire mid-render and trip React's "Cannot update a component while rendering" warning (lesson learned from B42). Layout per implementation plan B43.1:
    - Header: `← back` + yellow `HYROX · {round template}` chip + `⏸ pause` button. Back arrow auto-pauses so the user can dip out to a Lift exercise without burning round time.
    - Round-progress dots — yellow filled = done, yellow ring = current, muted ring = upcoming. Per-dot aria-label.
    - Round + leg label ("ROUND 2 · STATION") + headline (e.g. "SkiErg") + subtitle ("Race standard" / "1000m · 0.62 mi").
    - Gym clock — rounds clock per §5.4 (continuous within a round, resets only on round-transition).
    - Recent splits row — chips showing this round's run leg + up to 2 prior round totals.
    - Intra-leg comparison band — green/amber/neutral per `buildIntraLegComparison`'s `status`.
    - Green Done · Stamp `{segment time}` button.
    - Skip {leg} secondary text link.
    - Paused-state banner pill at the top (yellow on black).

546. **Dual-clock pattern: round clock + segment clock.** Per design doc §5.4 the timer "keeps running" across legs within a round (visual continuity). Per the same section + plan B43.3, each leg's stamped `timeSec` is segment-specific (run-only / station-only). To satisfy both:
    - `roundStartTimestamp` set when the round begins; **NOT reset on run→station transition**. Resets only on round-transition (station-Done of non-final round).
    - `legStartTimestamp` resets on EVERY leg transition (run→station + round-transition).
    - Visual gym clock reads `roundElapsedSec = (now - roundStartTimestamp - paused) / 1000` — the "round clock" the user perceives.
    - Done-button label + handleDone stamp read `elapsedSec = (now - legStartTimestamp - paused) / 1000` — the segment time committed to the leg's `timeSec`.
    - Both timestamps reset on round-transition + a fresh `totalPausedMs: 0`. `totalPausedMs` also resets on run→station transition so segment-clock arithmetic is clean within the same round.

547. **`activeSession.hyrox` shape** (additive — no persist version bump):
    ```
    activeSession.hyrox = {
      exerciseId,                  // round-template library id
      prescription: {              // committed at Begin round 1 (B42 overlay)
        roundCount, runDistanceMeters, restSec, stationId,
      },
      currentRoundIdx,             // 0-indexed
      currentLeg,                  // 'run' | 'station'
      roundStartTimestamp,         // ms — visual clock anchor
      legStartTimestamp,           // ms — segment stamp anchor
      totalPausedMs,
      isPaused, pauseStartedAt,
      completedLegs: [             // chronologically ordered, LoggedHyroxRound shape
        { roundIndex, type:'run', distanceMeters, distanceMiles, timeSec, completedAt },
        { roundIndex, type:'station', stationId, distanceMeters?, weight?, reps?, timeSec, completedAt },
        ...
      ],
      completedAt,                 // set on station-Done of final round
    }
    ```

548. **Done flow per implementation plan B43.3.** `handleDone` is a `useCallback` with `[hyrox, activeSession, saveActiveSession, elapsedSec, navigate, exerciseId]` deps so the closure always reads the latest segment-elapsed:
    - **Run leg** → stamp `{type:'run', distanceMeters, distanceMiles, timeSec, completedAt}` (distance pulled from prescription, miles via `metersToMiles`); flip `currentLeg` to `'station'`; reset `legStartTimestamp = Date.now()`; **do NOT** reset `roundStartTimestamp`; `totalPausedMs: 0`. Update URL → `/round/N/station`.
    - **Station leg of NON-final round** → stamp `{type:'station', stationId, ...raceStandardDimensions, timeSec, completedAt}` (distance/weight/reps pulled from the catalog station's `raceStandard`); reset BOTH timestamps; advance `currentRoundIdx`; flip back to `'run'`. Update URL → `/round/N+1/run`. (B44 wires the post-round flash + rest countdown between.)
    - **Station leg of FINAL round** → stamp the leg; mark `hyrox.completedAt`; navigate to `/log/hyrox/:id/summary` (B45 surface — for B43 the route falls through to no-match since summary doesn't exist yet, which is fine — B45 wires the summary).

549. **App.jsx route swap.** `import HyroxRoundLoggerStub` → `import HyroxRoundLogger`; route element swapped accordingly. Stub component file deleted from disk. `path="/log/hyrox/:exerciseId/round/:roundIdx/:leg"` unchanged.

550. **`hybrid-b43-sanity.mjs` at worktree root.** 41/41 pass. Covers (1) `getStationHistory` cross-template aggregation with synthetic Tuesday + Friday + lift-only sessions (3 SkiErg legs across 2 templates correctly aggregated newest-first; sled push 1-leg lookup; cold-start empty array); (2) `getRunLegHistory` exact-distance match; (3) `computePaceFromHistory` 35.5 s/100m on synth SkiErg history; (4) `buildIntraLegComparison` exact-match `mode='exact'` + status direction (current 340 < last 360 = 'ahead'; 380 > 360 = 'behind'; 0 = 'neutral'); (5) cross-station rotation invariant (today's SkiErg compares against SkiErg history NOT against the round-position-prior Row leg); (6) pace fallback when distance mismatched; (7) cold-start band-hide; (8) run leg comparison + run pace fallback; (9) Done-flow state-transition rules across run→station, station-non-final→next-run, station-final→summary; (10) Brooke Tuesday + Friday integration spot-check.

551. **Live preview verified** (port 5177, mobile 375×812, synthetic Brooke Tuesday split + injected prior session with two rounds: Run 800m @ 240/248s + SkiErg 1000m @ 350/360s). Round 1 run leg shows "ROUND 1 · RUN" / "800m Run" / "800m · 0.50 mi" subtitle / RUN CLOCK at 0:04 (later ROUND CLOCK after the dual-clock fix) / comparison band `−3:50 vs your last 800m run / 4:00 last time` in green. Tap Done → stamps `{run, 800m, timeSec:16}` and advances to round 1 station. Station leg: "ROUND 1 · STATION" / "SkiErg" / "Race standard" / ROUND CLOCK keeps running (e.g. 0:23 = 19s run + 4s station) / Done button shows segment-only 0:03 / `R1 run 0:19` chip in recent splits / `−5:37 vs your last SkiErg / 6:00 last time` (anchored to most-recent SkiErg leg = R2's 360s = 6:00). Tap Done → stamps station with segment-only time, advances to round 2 run. Round 2: ROUND CLOCK + segment clock both reset to 0:00, "R1 total 0:33" chip in recent splits (16+17=33). Comparison band `−2:52 vs your last 800m run / 4:00 last time`. BottomNav + HamburgerMenu + RestTimer all correctly hidden on `/log/hyrox/*`. Console clean of new errors (only pre-existing key-via-spread `<ExerciseItem>` warnings from BbLogger.jsx, documented known caveat).

552. **Build.** `npx vite build --outDir /tmp/test-build` → 847.49 KB bundle / 228.57 KB gzipped (+15.30 KB / +3.94 KB vs Batch 42, accounted for by `getStationHistory` + `getRunLegHistory` + `computePaceFromHistory` + `buildIntraLegComparison` (4 helpers, ~5 KB); `GymClock` component (~2.5 KB); `HyroxRoundLogger` (~7.5 KB)).

### Batch 44 (April 25, 2026) — Hybrid Training v1: post-round flash + rest countdown between rounds

Eighth batch of the Hybrid Training v1 workstream and the **fourth user-visible HYROX surface**. Wires two transient overlays between non-final rounds: PostRoundFlash for ~2.5s with a station-anchored comparison headline, then RestBetweenRoundsTimer counting DOWN from prescribed rest. Final round skips both — the user is funneled directly to `/summary` (B45). Branch shipped to `claude/hybrid-b44-rest-countdown` — NOT auto-merged to main, awaiting user review.

553. **`computeRoundDelta(roundIndex, completedLegs, sessions, prescription)` in `helpers.js`.** Composes the headline + subheadline rendered on PostRoundFlash. Three branches per design doc §14.2:
    - **`'round-position'`** — same exerciseId (round template) + same stationId at the same roundIndex in a prior session. Headline `"Round N done · M:SS"` with subheadline `"X faster than last time"` (round-total delta — sum of run + station for the round). Same-exact-match path, most specific.
    - **`'station-anchored'`** — fallback when branch 1 doesn't match (rotation pool, different round position, different template). Honors the headline B43 invariant: stations are the comparison primitive. Headline `"Round N done · {Station} M:SS"` with subheadline `"X faster than your last {Station}"` (station-leg delta against most-recent prior leg of same station, dimensions matching when possible; widens to all-station when exact dims unavailable).
    - **`'cold'`** — station has no prior history at all. Headline `"Round N done · M:SS"` with `subheadline: null` and `deltaSec: null`.
    Returns `{ headline, subheadline, mode, deltaSec }`. Pure — defensive against null/empty/malformed inputs at every layer (returns null for invalid roundIndex / completedLegs; returns 'cold' result when sessions array is null/empty so the cold path always composes a round-total).

554. **`PostRoundFlash.jsx` (new, `src/pages/log/`).** Full-screen yellow-on-black overlay at z-70. Renders for ~2.5s after a non-final round's station leg is stamped Done. Layout:
    - Yellow `Round complete` chip (echoes the round logger header for visual continuity).
    - 30px headline ("Round N done · 5:42" or "Round N done · SkiErg 5:34").
    - 16px subheadline ("0:18 faster than last time" / "0:14 faster than your last SkiErg") — hidden on cold-start.
    - Bottom-fixed muted hint: `Tap to continue · rest is next`.
    Yellow radial-glow background matching the StartHyroxOverlay treatment (B42). 3-line CSS keyframe fade-in animation (matches B42's headline pattern). Tap anywhere or wait `durationMs` to advance — both call `onAdvance`. Pure presentational; parent owns the phase transition + absolute timestamp for background-survival.

555. **`RestBetweenRoundsTimer.jsx` (new, `src/pages/log/`).** Full-screen yellow countdown clock at z-70. Auto-fires after PostRoundFlash dismisses (or skipped). Layout:
    - Top: yellow `Rest between rounds` chip + "Round N+1 of M starts soon" subline.
    - Center: the existing `GymClock` component with `mode='rest'` + remaining seconds passed via `elapsedSec`. Eyebrow auto-renders "REST". Walk-it-off hint underneath.
    - Bottom: `Skip rest` (white ghost) + `+ 30 sec` (yellow filled) buttons.
    100ms tick mirrors B43's HyroxRoundLogger pattern. Auto-completes via `useEffect` when `remainingMs <= 0` — fires `onComplete` which advances to the next round. Pure tick math: `remainingSec = max(0, (restEndTimestamp - now) / 1000)`. Background-survives reload because parent uses absolute `restEndTimestamp`.

556. **`GymClock` extended for low-time visual urgency** (`src/components/GymClock.jsx`). When `mode='rest'` AND `total <= 5` seconds remaining, the digit color shifts from `#FEF08A` (yellow-200) to `#FCD34D` (amber-300). Small visual cue for "almost done" that costs nothing — no animation, no API change, pure render-side. Existing `mode='up'` callsites unaffected. Eyebrow + softer wash from B43 unchanged.

557. **Phase state machine on `activeSession.hyrox`** (`HyroxRoundLogger.jsx`). Five new fields layered onto the B43 shape (no persist version bump — additive):
    - `phase: 'logging' | 'flash' | 'rest'` (default `'logging'`).
    - `flashStartTimestamp: ms | null` — set on station-Done of non-final round, cleared on flash-advance.
    - `restStartTimestamp: ms | null` — set on flash-advance, cleared on rest-complete (B45 reads this for `restAfterSec` derivation when composing `rounds[]` for session save).
    - `restEndTimestamp: ms | null` — absolute target for the countdown. Add-30s shifts it forward, skip clears it. Background-survives reload.
    Initial seed includes the full set as null/`'logging'` for a clean B45 read; the existing B43 hydrate path keeps existing-session fields intact when the user is mid-flow.

558. **handleDone for non-final station leg rewritten.** B43 advanced directly to next-round's run leg (resetting both clocks + URL + currentRoundIdx in one transition). B44 splits this into THREE transitions:
    - **Station-Done (non-final) → flash**: stamp the leg into `completedLegs`, set `phase: 'flash'`, set `flashStartTimestamp: Date.now()`. Round/leg clocks NOT reset. URL stays at `/round/N/station` so reload mid-flash returns to the flash overlay.
    - **handleFlashAdvance: flash → rest**: reads `prescription.restSec`, sets `phase: 'rest'`, `restStartTimestamp: now`, `restEndTimestamp: now + restSec * 1000`. Reload mid-rest returns to the rest overlay.
    - **advanceToNextRound (rest-complete OR skip-rest): rest → logging**: resets both clocks, advances `currentRoundIdx + 1`, flips `currentLeg` to `'run'`, clears phase fields, navigates URL to `/round/N+1/run`.
    Each transition is idempotent — the handler bails if the current phase isn't what it expects, so PostRoundFlash's auto-advance + tap-to-continue + RestBetweenRoundsTimer's auto-complete-at-zero + skip-button can't double-fire and corrupt state.

559. **`handleAddRestSeconds(deltaSec)`.** Bumps `restEndTimestamp += deltaSec * 1000`. Validates that `phase === 'rest'` before mutating (prevents stray taps from extending nonexistent rest). Verified live: 86s remaining + tap +30 → 116s remaining (exact 30s shift on the absolute timestamp).

560. **Render branch in HyroxRoundLogger return.** New early-return blocks BEFORE the main logger UI:
    - `if (hyrox.phase === 'flash')` → render `<PostRoundFlash delta={computeRoundDelta(...)} durationMs={Math.max(0, FLASH_DURATION_MS - elapsedFlashMs)} onAdvance={handleFlashAdvance} />`. The `durationMs` math means a reload-after-flash-already-elapsed renders for 0ms then auto-advances synchronously.
    - `if (hyrox.phase === 'rest')` → render `<RestBetweenRoundsTimer restEndTimestamp={...} onSkip={handleSkipRest} onAddSeconds={handleAddRestSeconds} onComplete={handleRestComplete} />`. Reload-after-rest-elapsed similarly auto-completes immediately.
    Both overlays are full-screen z-70, so the underlying logger UI is fully covered. The B43 round/segment-clock useEffects keep ticking under the hood (negligible cost) and resume rendering correctly when phase flips back to `'logging'`.

561. **Final round behavior unchanged.** When `currentRoundIdx >= roundCount - 1` AND the user taps Done on the station leg, `handleDone`'s pre-existing isFinalRound branch runs: stamps the leg, sets `hyrox.completedAt`, navigates straight to `/log/hyrox/:id/summary`. No flash, no rest. The phase fields stay at `'logging'` through the transition. Verified live: 4-round walkthrough end-to-end, R4 station Done → URL `/summary` immediately, phase still `'logging'`, completedLegs has all 8 entries.

562. **`hybrid-b44-sanity.mjs` at worktree root.** 70/70 pass. Covers:
    - **Test 1 — Branch 1 'round-position'**: faster path (delta=-20s, "faster than last time"), slower path (+50s, "slower"), match path (0s, "Matched last time").
    - **Test 2 — Branch 2 'station-anchored' across templates**: Saturday template's R0 SkiErg compares to Friday R1 SkiErg (cross-template aggregation), delta=-30s.
    - **Test 3 — Branch 2 fallback within same template, different position**: Friday R2 with SkiErg compares to Friday R1 SkiErg (R2 of prior had Sled Push, so branch 1 fails → branch 2 fires).
    - **Test 4 — Branch 3 'cold start'**: no prior history at all, mode='cold', subheadline=null, deltaSec=null. Also lift-only-prior case.
    - **Test 5 — Defensive cases**: null/negative/non-number roundIndex; null/non-array/empty completedLegs; mismatched roundIndex; null sessions; null prescription. All handled gracefully.
    - **Test 6 — Rest countdown decrement math**: 0s/30s/89.5s/90s/120s (clamp)/-5s.
    - **Test 7 — Add 30s shifts restEndTimestamp**: +30 / +60 / remaining bumps by 30.
    - **Test 8 — Final round detection**: 5/5 cases across roundCount=1/4/5/6.
    - **Test 9 — Phase state-transition shapes**: station-Done → flash (clocks NOT reset), flash → rest (restStart + restEnd set), rest → logging (clocks reset, currentRoundIdx + 1, phase fields cleared, completedLegs preserved).
    - **Test 10 — Skip rest = rest-complete**: identical state shape regardless of timing.

563. **Live preview verified** (port 5178, mobile 375×812, synthetic Brooke Tuesday split + injected prior session). Walked through:
    - Round 1: run leg → station leg → tap Done → flash overlay renders with `Round 1 done · 0:20` + `9:30 faster than last time` (round-position branch, today's 20s vs prior R0's 590s) + tap-to-continue hint.
    - Tap dialog → phase flips to `'rest'` → REST gym clock at 1:42 (102s remaining of 120s). Skip rest + + 30 sec buttons visible. URL still `/round/1/station`.
    - Tap +30s → `restEndTimestamp` shifts forward by 30000ms (verified: 86s → 116s remaining). Tap Skip rest → phase=`'logging'`, currentRoundIdx=1, currentLeg='run', clocks reset, URL `/round/2/run`.
    - Continued through R2, R3 (each producing flash → rest → advance), then R4 final station Done → URL `/summary` immediately (no flash, no rest, phase stays `'logging'`, completedLegs.length=8).
    - **Reload mid-rest**: phase + restEndTimestamp survived; rest dialog re-rendered with countdown picking up at 95s remaining (was 104s, ~9s elapsed during reload + load). URL stayed at `/round/2/station` (correct — round hasn't advanced).
    - No new console errors — only the pre-existing `<ExerciseItem>` key-via-spread warnings from BbLogger.jsx that predate B41 (documented known caveat).

564. **Files modified.**
    - `src/utils/helpers.js` — `computeRoundDelta` + extended block comment.
    - `src/components/GymClock.jsx` — `digitColor` prop on `DigitBox` + final-5s amber treatment in `mode='rest'`.
    - `src/pages/log/PostRoundFlash.jsx` (new) — flash overlay component.
    - `src/pages/log/RestBetweenRoundsTimer.jsx` (new) — rest countdown component.
    - `src/pages/log/HyroxRoundLogger.jsx` — phase state machine + new handlers + flash/rest render branches.
    - `hybrid-b44-sanity.mjs` (new, worktree root) — 70-assertion sanity script.

565. **Build.** `npx vite build --outDir /tmp/test-build` → 855.59 KB bundle / 230.49 KB gzipped (+8.10 KB / +1.92 KB vs Batch 43, accounted for by `computeRoundDelta` (+1.5 KB), `PostRoundFlash` (+2.5 KB), `RestBetweenRoundsTimer` (+2.5 KB), and the HyroxRoundLogger phase-state-machine wiring (~1.5 KB) + GymClock prop addition (~0.1 KB)).

### Batch 45 (April 25, 2026) — Hybrid Training v1: HYROX session summary + comparison chart + branching CTA

Ninth batch of the Hybrid Training v1 workstream and the **fifth user-visible HYROX surface** — the post-final-round summary screen (mockup 4). Replaces the no-route-match that B43+B44 left in place when station-Done on the final round navigated to `/log/hyrox/:id/summary`. Composes `activeSession.hyrox.completedLegs[]` into the saved-session `rounds[]` shape, persists into `activeSession.exercises` so B41's section preview lights up its `✓ done` state on Back-to-lift, then renders hero total time + fastest round + vs-last delta + per-round comparison chart (today yellow solid + station-anchored synthetic prior white dashed) + round breakdown table + branching CTA. Branch shipped to `claude/hybrid-b45-summary` — NOT auto-merged to main, awaiting user review.

566. **Five new pure helpers in `helpers.js`** — all defensive against null / non-array / malformed inputs:
    - **`composeHyroxRoundsForSave(completedLegs, prescription)`** → groups flat `completedLegs[]` by `roundIndex` into nested `LoggedHyroxRound[]` per the B38 schema. `restAfterSec` for round N derived from gap between this station's `completedAt` and the next round's earliest leg `completedAt`, MINUS the next leg's `timeSec` (so the rest figure represents actual rest + flash time, not double-counted run time). Final round → `restAfterSec: 0`. Strips `roundIndex` from individual legs (the round wraps it).
    - **`getHyroxSessionTotalTime(rounds)`** → sum of every leg's `timeSec` + every round's `restAfterSec`. Mirrors `HyroxSectionPreview`'s done-state walker so the summary's hero total matches the section preview's "✓ done" stat exactly. Returns 0 on null / empty / malformed.
    - **`buildSyntheticPriorSeries(todayRounds, sessions, prescription, currentSessionId)`** → per-round synthetic prior, station-anchored per design doc §14.3. For each `roundIndex`, looks up the most-recent prior **run leg** (by run distance) AND most-recent prior **station leg** (by stationId + dimensions). Falls through to **pace projection** when exact dims don't match (flagged via `priorPaceFallback: true` so the chart renders hollow circles vs filled). Returns `null` for a round when EITHER leg has no resolvable prior. Self-exclusion via `currentSessionId` filters out today's session if it's already in `sessions[]` (post-finish state).
    - **`getHyroxBests(rounds, prescription)`** → cold-start sidebar data per §14.4. Returns `{ fastestRound, fastestRunLeg, fastestStationLeg }` with each value as `{ timeSec, label }` (e.g. `R2 SkiErg`, `R1 800m`).
    - **`computeBranchingCta(workout, activeSessionExercises)`** → per §16.1, walks the workout's non-HYROX sections; any uncompleted lift (`completedAt: 0` or missing) → `{ label: 'Back to lift →', action: 'lift' }`. All complete or HYROX-only → `{ label: 'Finish workout →', action: 'finish' }`. Lift-section detection mirrors B41's predicate (label trimmed-lowercased ≠ `'hyrox'`).

567. **`HyroxSessionSummary.jsx` (new, `src/pages/log/`).** Routes at `/log/hyrox/:exerciseId/summary`. Layout:
    - **HYROX COMPLETE pill** at top — yellow brand `#EAB308`.
    - **Round template name** centered.
    - **Hero total time** rendered via `GymClock` with `mode='up'` + `eyebrowOverride='Total time'` — preserves the brand visual continuity from B43's round logger.
    - **Two stat tiles**: Fastest round (with R-label subtitle); vs-last delta (negative green / positive amber) with `{prior totalTime} · {N} rounds` subtitle. Cold-start variant: VS LAST shows `—` + "No prior session".
    - **Per-round comparison chart** — inline SVG ~280×140, today yellow solid `#EAB308` line + prior white dashed (`rgba(255,255,255,0.5)`). Filled circles on exact-match prior, hollow circles on pace-fallback prior, omitted entirely when prior is null. Y-axis shows `M:SS` ticks at min/max; x-axis labels `R1/R2/R3/R4`. Today/Prior legend in card header.
    - **Cold-start variant** (when ALL today's stations are first-time logged): chart replaced with `Today's bests` sidebar — Fastest round / Fastest run leg / Fastest station leg with R-position labels.
    - **Round breakdown table** — header row + one row per round. Columns: `RD / RUN + STATION / TOTAL / VS LAST`. The VS LAST cell uses `computeRoundDelta` (B44) — reuses the same three-branch logic the post-round flash overlay uses. Cold rounds show `—` + `· first time` annotation.
    - **Branching CTA** at bottom — `Back to lift →` or `Finish workout →` per `computeBranchingCta`. Both navigate to `/log/bb/:type` (the lift-tap returns the user to the workout view; the finish-tap leaves the user on BbLogger where they tap the existing Finish session button — B46 will auto-open the finish modal).

568. **Mount-only useEffect persists rounds[] into `activeSession.exercises`** (top-level — BbLogger's flat shape during a live session, NOT the nested `data.exercises` shape that `addSession` writes at finish-modal save time). Finds the HYROX placeholder exercise seeded by B41's `templateExercises` builder via id-or-name match, updates it in place with `rounds: composed`, `prescribedRoundCount`, `prescribedStationId`, `prescribedRunDistanceMeters`, and `completedAt`. Idempotent — skips when the matching exercise already has a non-empty `rounds[]` (re-mount after reload). Fires AFTER render commit (per B42's lesson) so Zustand subscribers don't trip the "Cannot update a component while rendering" warning. Reload mid-summary survives via the existing `activeSession` round-trip in localStorage.

569. **Important schema caveat — BbLogger's `saveActiveSession` writes a FLAT shape** (`{type, exercises, sessionNotes, sessionStarted, ...}`) WITHOUT `hyrox`. So when the user taps Back-to-lift and BbLogger re-mounts, its first `saveActiveSession` write drops `activeSession.hyrox`. **The rounds[] persistence is preserved** because it lives in `activeSession.exercises` (not `hyrox`), so B41's section preview keeps showing ✓ done correctly. The hyrox-drop is fine in this batch since the round logger only consumes `hyrox` during a live session — once summary lands, hyrox's job is done. B46 will need to handle hybrid finish without depending on `hyrox`.

570. **Route registration in `App.jsx`.** New `<Route path="/log/hyrox/:exerciseId/summary" element={<HyroxSessionSummary />} />` registered between the `/round/:roundIdx/:leg` route and `/cardio`. The fullscreen-flow predicate from B42 (`path.startsWith('/log/hyrox/')`) already hides BottomNav + RestTimer + HamburgerMenu on the summary route — no additional wiring needed.

571. **`hybrid-b45-sanity.mjs` at worktree root.** 69/69 pass. Covers: composeHyroxRoundsForSave correctness (4-round single-station, restAfterSec gap math, final-round = 0, defensive cases incl. malformed legs); getHyroxSessionTotalTime walker; buildSyntheticPriorSeries across exact-match cross-template / mixed-history / pace fallback / cold-start / self-exclusion; getHyroxBests across 4-round dataset with varying times; computeBranchingCta across un-done lift / all-complete / HYROX-only / template-only / defensive (null workout, missing sections, "  hyrox  " whitespace, string-shape exercises); Brooke Tuesday integration spot-check (4 rounds composed + restAfterSec=30s + total=2490s=41:30).

572. **Live preview verified** (port 5179, mobile 375×812). Three scenarios pass:
    - **Brooke Tuesday with un-done Lift exercise**: summary renders HYROX COMPLETE pill, 41:20 hero clock, Fastest round 9:50 R2, vs-last +20:00 (today 4-round 41:20 vs prior 2-round 21:20), per-round chart with 4 today-yellow + 4 prior-dashed-white points, breakdown table with 4 rows showing −0:20/−0:40/−0:17/−0:13 (green) deltas, CTA reads `Back to lift →`. Tap → returns to `/log/bb/brk_tuesday`, B41 section preview now correctly shows `✓ done · 41:20 · +20:00 vs last` + `HYROX Run + SkiErg Round` for the HYROX section. Lift section's Cable Lateral Raise still un-done.
    - **HYROX-only workout** (Lift section removed): CTA correctly reads `Finish workout →`.
    - **Cold start** (sessions cleared): VS LAST tile shows `—` + "No prior session", chart replaced with `Today's bests` sidebar (Fastest round R2 9:50 / Fastest run leg R2 800m 4:05 / Fastest station leg R2 SkiErg 5:45), breakdown table all rows show `—` + `· first time` annotation.
    - Zero new console errors — only pre-existing `<ExerciseItem>` key-via-spread warnings from `BbLogger.jsx:3806` that predate B41 (documented known caveat).

573. **Files modified.**
    - `src/utils/helpers.js` — 5 new helpers (composeHyroxRoundsForSave, getHyroxSessionTotalTime, buildSyntheticPriorSeries, getHyroxBests, computeBranchingCta).
    - `src/pages/log/HyroxSessionSummary.jsx` (new) — full summary screen.
    - `src/App.jsx` — route registration + import.
    - `hybrid-b45-sanity.mjs` (new, worktree root) — 69-assertion sanity script.

574. **Build.** `npx vite build --outDir /tmp/test-build` → 871.91 KB bundle / 234.64 KB gzipped (+16.32 KB / +4.15 KB vs Batch 44, accounted for by the 5 new helpers (~6 KB) + HyroxSessionSummary component including inline SVG chart + cold-start sidebar + round breakdown table (~10 KB) + route registration (~0.3 KB)).

### Batch 45 followup (April 25, 2026) — HYROX Hybrid as a starting-point template + addSplitWithLibrary

User-requested: ship Brooke's HYROX program as a discoverable template in the split chooser so any user (including Brooke) can pick it from `+ → New split → How do you want to start?`. Direct-to-main since the change is purely additive — new template entry + new store action + minor `normalizeExerciseEntry` extension. No persist version bump, no schema break.

575. **`HYROX_HYBRID_WORKOUTS` + new template entry in `src/data/splitTemplates.js`.** All 6 Brooke workouts encoded inline with object-shape exercises carrying `type` + structured `rec` + (for HYROX-round entries) `roundConfig`. Tuesday's "HYROX Run + SkiErg Round" carries `{stationId: 'sta_skierg', runDistance: 800m, rounds: 4, rest: 120s}`. Friday's "HYROX Simulation Round" uses a 7-station rotation pool + 1000m run + 4 rounds + 90s rest. Saturday's "Wall Balls + 200m Run Round" carries `{stationId: 'sta_wall_balls', runDistance: 200m, rounds: 3, rest: 60s}`. Workout ids are namespaced `hyx_*` so they don't collide with BamBam's Blueprint or any other template. The template entry is the **first** in `SPLIT_TEMPLATES` so it appears at the top of the chooser. Rotation: rest (Sun) → Mon-Sat workouts.

576. **`loadTemplateForDraft` deep-clones object exercises (`splitTemplates.js`).** Pre-followup the helper did a shallow array copy (`[...(s.exercises || [])]`) which preserved object references — a user editing a HYROX-round entry's roundConfig in one draft would mutate the template literal. Now wraps each non-string exercise in `JSON.parse(JSON.stringify(e))` so the source template is fully isolated.

577. **`normalizeExerciseEntry` extended to preserve `type` + `roundConfig` (`helpers.js`).** Pre-followup the helper returned either a bare string or `{name, rec}`, dropping any other fields. That stripped the metadata `collectLibraryAdditionsFromSplit` needs to detect HYROX-round / running entries on save. Now: when `type` (non-empty string) or `roundConfig` (object) is present alongside the name, returns `{name, [rec], [type], [roundConfig]}` with the optional fields conditionally included. Bare-string fast path preserved for the common case (no rec, no type, no roundConfig). UI surfaces (SplitCanvas / WorkoutEditSheet / formatRec) ignore the extra fields — they only render name + rec — so they're zero-cost metadata that rides along until the save path needs them. Migration-18a-sanity + hybrid-b40-sanity both still pass (no regression).

578. **`addSplitWithLibrary(splitData)` store action (`useStore.js`).** Symmetric to the existing `importSplitWithLibrary(data)` (Batch 40) but takes a raw split object directly instead of the file-import wrapper. Walks `collectLibraryAdditionsFromSplit` to find HYROX-round / running entries that need library entries, fires `addExerciseToLibrary` for each (per-entry try/catch so a single bad exercise doesn't abort the whole save), then calls the existing `addSplit`. Returns the created split so callers that need the id for activate-on-save still work.

579. **SplitCanvas's create path uses `addSplitWithLibrary`** (`SplitCanvas.jsx`). Edit path still uses `updateSplit` (the user is modifying an existing split, no new library entries should be implicitly created — they create them explicitly via the WorkoutEditSheet picker). Only the create path goes through library spawn so template-derived splits and any HYROX-round / running entries the user typed manually get their library entries auto-spawned on Save.

580. **`hyrox-hybrid-template-sanity.mjs` at worktree root.** 61/61 pass. Covers: template registration (HYROX Hybrid first in chooser, name/emoji/cycle correct, 6 workouts, 7-day rotation starting with Sunday rest); per-workout section + exercise count integrity (Mon: 6+2, Tue: 5+1, Wed: 6+1, Thu: 1+3, Fri: 5+1, Sat: 5+1); HYROX-round entries carry valid roundConfig with correct stationId / rotationPool / runDistance / round count / rest; loadTemplateForDraft deep-clones object exercises so source isn't mutated; collectLibraryAdditionsFromSplit against the deep-cloned draft against a HYROX-only seeded library yields exactly 3 hyrox-round + 4 running entries to create + 0 errors (Farmers Carry / Sled Push / Sled Pull resolve to seeded catalog and are correctly NOT in toCreate); roundConfig fields preserved end-to-end through draft.

581. **Live preview verified** (port 5180, mobile 375×812, fresh install). Five scenarios pass: (a) `/splits/new/start` chooser renders HYROX Hybrid card at the top with 7-day badge + Brooke description + rest-day "R" rotation chip preview; (b) tapping the card seeds `splitDraft` with all 6 workouts including roundConfig metadata, navigates to `/splits/new`; (c) SplitCanvas renders Glutes & Light Run (8 exercises), Shoulders & HYROX Intervals (6), Hamstrings (7), Active Rest (4), Back & HYROX Simulation, Heavy Glutes & Finisher; (d) tap `Save & Activate` → split commits, library grows from 98 → 128 entries (8 stations + 3 newly-spawned hyrox-rounds + 4 newly-spawned running + 25 needsTagging weight-training), HYROX Hybrid is now the active split, splitDraft cleared, navigated to `/splits`; (e) navigate to `/log/bb/hyx_tuesday` → workout view renders 5 Lift exercise cards (Cable Lateral Raise / Shoulder Press Machine / DB Lateral Raise / Incline Bench Front Raise / Reverse Flies) + the immersive yellow HYROX section preview card with `INTERVALS · 4 ROUNDS / HYROX Run + SkiErg Round / RUN LEG 800m / STATION SkiErg / REST 2:00 / Start HYROX →` button. Zero new console errors — only pre-existing `<ExerciseItem>` key-via-spread warnings from BbLogger.jsx.

582. **Files modified.**
    - `src/data/splitTemplates.js` — HYROX_HYBRID_WORKOUTS + first SPLIT_TEMPLATES entry + deep-clone in loadTemplateForDraft.
    - `src/utils/helpers.js` — normalizeExerciseEntry extended for type + roundConfig.
    - `src/store/useStore.js` — addSplitWithLibrary action.
    - `src/pages/SplitCanvas.jsx` — handleSave create path uses addSplitWithLibrary.
    - `hyrox-hybrid-template-sanity.mjs` (new) — 61-assertion sanity.

583. **Build.** `npx vite build --outDir /tmp/test-build` → 877.37 KB bundle / 236.07 KB gzipped (+5.46 KB / +1.43 KB vs Batch 45, all in the inline HYROX_HYBRID_WORKOUTS template data + the addSplitWithLibrary action + normalizeExerciseEntry extension).

### Batch 48 (April 25, 2026) — Per-gym editor in ExerciseEditSheet (silence the mid-session prompts)

User feedback: "It starts to get a little bit overwhelming, the number of prompts that appear mid-session… I'd like to just be able to set the machine or availability for gyms inside the exercise library." Surfaces every piece of information the GymTagPrompt + Machine chip ask for mid-session up into the library so the user can configure it once and silence the noise.

584. **`defaultMachineByGym?: { [gymId]: string }` on `Exercise`** (`useStore.js`). Per-gym default machine instance — set in the new ExerciseEditSheet → Gyms section. Wins over historical session values when the BbLogger seed pass picks the Machine chip's initial value. Empty/missing means "no library default" (falls back to history). Setting an empty/whitespace value removes that gym from the map; emptying the map drops the field entirely (shape stays minimal).

585. **`setDefaultMachineByGym(exerciseId, gymId, instance)` store action** (`useStore.js`). Trims input to 40 chars, no-ops on missing args, idempotent on no-change writes. Lives next to the existing gym-tagging actions.

586. **`getDefaultMachineForGym(exercise, gymId)` helper** (`helpers.js`). Pure read of the map. Returns null on missing gym / missing exercise / malformed map / whitespace-only value. Defensive against null inputs so callers don't need to guard.

587. **Gyms section in `ExerciseEditSheet.jsx`** — new section between Unilateral and Save row, only renders when `settings.gyms.length > 0` and the type is not `hyrox-round` (round templates aren't gym-scoped). For each configured gym, renders a card with:
    - Gym label + Default badge (when this is `settings.defaultGymId`).
    - Three-button segmented status row that **mirrors the GymTagPrompt buttons exactly** (per user request — "the three buttons in the gym selection in the exercise library you're designing need to mirror the three buttons that are shown in the session logger"): `Yes, tag it` (accent-filled), `Not this time` (bg-item neutral), `Hide for this gym` (red ghost). Selected = full opacity, unselected = 35% opacity. Mapping: Yes → adds to `sessionGymTags` (silences the auto-tag prompt); Not this time → neutral default state, no library writes (mid-session prompt may still fire); Hide for this gym → adds to `hiddenAtGyms` + `skipGymTagPrompt` (filters out of logger + silences forever).
    - For machine-equipment exercises (Selectorized + Plate-loaded), a `Default machine` text input. Setting a value auto-promotes the gym to `Yes, tag it` (since "this is the Hoist at VASA" implies "this exercise IS at VASA").
    - When status is Hidden, the machine input drops + an italic "This exercise won't appear in workouts logged here." hint shows.
    - Buttons sized at `text-[10px] / px-2 py-1 / leading-tight / whitespace-nowrap` so all three fit on one line at 375px without wrapping.

588. **Commit-on-Save diffs** (`ExerciseEditSheet.jsx`). Local Sets/Map track the editor state; `commitGymChanges()` runs first inside `handleSave` and diffs against the original exercise's gym fields, calling the per-action store mutations (`addExerciseGymTag` / `removeExerciseGymTag` / `addHiddenAtGym` / `removeHiddenAtGym` / `addSkipGymTagPrompt` / `setDefaultMachineByGym`) for each delta. Hidden gyms always also get added to `skipGymTagPrompt` (mirrors the mid-session "Hide for this gym" button's dual-write behavior). `skipGymTagPrompt` is never auto-removed — a user who hid + later un-hid likely still doesn't want the prompt to fire there.

589. **BbLogger seed wired** (`BbLogger.jsx`). `templateExercises` and the `defaultExercises` extras path both read `libraryMachineFor(name)` from a new local helper that calls `getDefaultMachineForGym(libraryByName.get(name), seedGymId)`. Priority for `equipmentInstance` at session start becomes: (1) library `defaultMachineByGym[seedGymId]` ← new, wins; (2) most-recent same-gym session's `equipmentInstance` (existing); (3) most-recent anywhere session's `equipmentInstance` (existing); (4) blank. Resumed sessions read `savedSession.exercises[i].equipmentInstance` directly — they don't re-seed, so mid-flow library edits don't disturb the in-progress session. Mid-session gym swaps via the SessionGymPill also don't re-seed (existing behavior, carries over).

590. **Verified live in preview** (port 5173, mobile 375×812, three test gyms VASA/Training Room/Lanhammer seeded). Scenarios pass: (a) Tagging Chest Supported Wide Row's VASA = Available, TR = Available + machine "Cybex", Lanhammer = Hidden persists exactly to localStorage as `sessionGymTags: ['gym_vasa','gym_tr']`, `hiddenAtGyms: ['gym_lanhammer']`, `skipGymTagPrompt: ['gym_lanhammer']`, `defaultMachineByGym: { gym_tr: 'Cybex' }`. (b) Re-opening the sheet correctly re-seeds all three states from the persisted exercise. (c) Starting a fresh Pull session at TR seeds `equipmentInstance: 'Cybex'` on Chest Supported Wide Row — the cyan Machine chip reads `Cybex` in the toolbar without any user interaction. (d) Starting a fresh Pull session at VASA (tagged but no machine value) seeds `equipmentInstance: ''` — Machine chip stays in dashed-empty state. (e) Console clean throughout.

591. **Files modified.**
    - `src/store/useStore.js` — `setDefaultMachineByGym` action.
    - `src/utils/helpers.js` — `getDefaultMachineForGym` helper.
    - `src/components/ExerciseEditSheet.jsx` — Gyms section + commit-on-Save diff logic; +`useStore` + `isMachineEquipment` imports.
    - `src/pages/log/BbLogger.jsx` — `getDefaultMachineForGym` import + `libraryMachineFor` helper + seed priority extended in both `templateExercises` and `defaultExercises` paths.

592. **Build.** `npx vite build --outDir /tmp/test-build` → 892.32 KB bundle / 240.03 KB gzipped (+14.95 KB / +3.96 KB vs Batch 45 followup, accounted for by the new Gyms section UI in ExerciseEditSheet (~12 KB) + the seed wiring + helper + store action (~3 KB)).

### Batch 49 (April 25, 2026) — HYROX yellow accent preset + custom hex color picker

User feedback: "I'd also like to add a stark yellow contrast theme… add a custom color picker theme where you can just select exactly what tone you want for the accent and have it applied. The yellow accent should match the HYROX accent." Two new entries in the accent picker — Yellow (matches HYROX brand `#EAB308`) and Custom (rainbow swatch wrapped around the user's picked hex). Custom uses CSS variables under the hood so the user can pick any hex without bundling new Tailwind classes; named themes keep their static class strings unchanged (zero risk).

593. **YELLOW preset in `THEMES`** (`theme.js`). Tailwind classes `bg-yellow-500 / text-yellow-400 / etc.` (`yellow-500` IS HYROX `#EAB308`). Light-accent treatment per WCAG: `contrastText: '#1A1A1A'`, `textOnBg: 'text-black'` etc. so text on yellow buttons reads dark instead of white.

594. **Custom-color helpers in `theme.js`.** `normalizeHex(input)` accepts `#RGB`/`#RRGGBB`/`RRGGBB`, returns `#RRGGBB` uppercase or null. `getContrastTextForHex(hex)` uses the WCAG-style luminance formula (0.299/0.587/0.114) with a 0.55 threshold — light hex → `#1A1A1A`, dark hex → `#FFFFFF`. Internal `lightenHex(hex, amount)` mixes toward white for the `:hover` state.

595. **`getTheme('custom', customHex)` branch + module cache** (`theme.js`). `_customAccentHex` module-level cache primed via the new `setCustomAccentHex(hex)` setter so the 22 existing `getTheme(settings.accentColor)` call sites don't need an extra arg threaded through. App.jsx calls the setter synchronously during render, so children rendering in the same pass get the right hex without a flash. Custom theme returns `accent-bg / accent-text / accent-border / accent-ring / accent-bg-subtle / accent-bg-hover / accent-text-on-bg / accent-text-on-bg-muted / accent-text-on-bg-dim` — fixed CSS class names that read CSS variables.

596. **`applyAccentToRoot(theme)`** (`theme.js`). Sets CSS variables on `<html>` for the accent: `--accent-hex / --accent-bg-hover / --accent-bg-subtle / --accent-border / --accent-ring / --accent-text / --accent-contrast / --accent-text-on-bg-muted / --accent-text-on-bg-dim`. No-op for named themes — they paint via Tailwind class strings, the variables stay unset (or stale from a prior custom session, harmless). Light-accent paths invert muted/dim opacity on a black base, dark accents stay on white.

597. **`.accent-*` classes in `index.css`.** Static utility classes that consume the CSS variables: `.accent-bg / .accent-bg-hover:hover / .accent-bg-subtle / .accent-text / .accent-border / .accent-ring (sets --tw-ring-color) / .accent-text-on-bg / .accent-text-on-bg-muted / .accent-text-on-bg-dim`. These are the class strings the custom theme returns, so any consumer reading `theme.bg`, `theme.text`, etc. gets the right color whether the active accent is named or custom.

598. **`settings.customAccentHex` default `'#EAB308'`** (`useStore.js`). HYROX yellow as the starting point so a user who taps Custom without picking a hex gets a sensible color out of the gate. New users + persist merge fall through cleanly via the existing settings deep-merge — no migration needed.

599. **App.jsx integration.** `setCustomAccentHex(settings.customAccentHex)` runs synchronously during render to prime the cache before any child reads `getTheme('custom')`. `useEffect([settings.accentColor, settings.customAccentHex])` calls `applyAccentToRoot(getTheme(...))` after commit so a freshly-picked color applies instantly without a full reload.

600. **HamburgerMenu picker — 11 named swatches + 1 custom** (`HamburgerMenu.jsx`). Grid bumped from 5 to 6 columns. The 12th cell is a `<label>` wrapping a hidden `<input type="color">` (native OS picker on tap — Chrome/Edge/Safari/iOS all expose a hex input inside the picker for users who want a specific value). Visually it's a 28px circle: outer rainbow conic-gradient ring (`#f43f5e → #f97316 → #facc15 → #22c55e → #06b6d4 → #3b82f6 → #8b5cf6 → #ec4899 → #f43f5e`) wrapping a 20px inner circle painted with the user's currently-picked hex. Selection state: 2px white outline on the outer ring. When the active accent is `'custom'`, a small `Custom: #XXXXXX` line renders below the grid in tabular-nums.

601. **Verified live in preview** (port 5173, mobile 375×812). Six scenarios pass: (a) Yellow swatch tap → entire UI repaints HYROX yellow including hero CTA, today calendar circle, BottomNav home icon, dashboard dots; text on yellow surfaces reads dark via the `text-black` override. (b) Custom swatch tap opens the OS color picker; picking violet `#7C3AED` repaints everything violet incl. BottomNav home icon. (c) Switching to coral `#FF6B6B` (light accent) renders dark text on the hero CTA (`getContrastTextForHex` returned `#1A1A1A`). (d) Switching back to violet (named theme) renders identically to pre-Batch-49 violet — Tailwind class strings paint directly, CSS variables ignored. (e) HYROX UI surfaces (round logger / overlay / summary) keep their hardcoded brand yellow regardless of user accent — independent per design doc §12.4 ("Don't theme yellow with the user's accent — it's a fixed brand color"). (f) Console clean, no React warnings, no flash of wrong color on initial render.

602. **Files modified.**
    - `src/theme.js` — YELLOW preset + custom-color helpers (normalizeHex / getContrastTextForHex / lightenHex / hexToRgba / buildCustomTheme) + `getTheme('custom')` branch + `setCustomAccentHex` cache + `applyAccentToRoot`.
    - `src/index.css` — `.accent-bg / .accent-text / .accent-border / .accent-ring / .accent-bg-subtle / .accent-bg-hover / .accent-text-on-bg / .accent-text-on-bg-muted / .accent-text-on-bg-dim` classes consuming CSS variables.
    - `src/store/useStore.js` — `settings.customAccentHex` default `'#EAB308'`.
    - `src/App.jsx` — synchronous cache prime + `useEffect` for `applyAccentToRoot`.
    - `src/components/HamburgerMenu.jsx` — 6-column grid + custom-hex picker swatch with rainbow ring + native color input + `Custom: #XXXXXX` indicator.

603. **Build.** `npx vite build --outDir /tmp/test-build` → 895.64 KB bundle / 241.02 KB gzipped (+3.32 KB / +0.99 KB vs Batch 48, accounted for by the custom-color helpers + applyAccentToRoot + the CSS variable / class additions + the picker UI). CSS bundle +0.97 KB (CSS-var classes).

### Batch 50 (April 25, 2026) — Post-workout feedback batch (UI polish + crash defense + numpad cleanup + auto-tag-on-machine + GymTagPrompt removed + notes redesign)

User-reported live-feedback batch from a Lanhammer session — see plan at `C:\Users\User\.claude\plans\some-thoughts-feedback-and-rippling-nebula.md`. Seven discrete items; six landed (one deferred for live repro). User-locked decisions: numpad's Done button removed; GymTagPrompt banner removed entirely; auto-tag-on-machine-entry; per-exercise notes pencil pill + Mark as Done share one row; session notes button-styled like + Add Exercise.

604. **`+ Add Exercise` dashed border removed** (`BbLogger.jsx:4193`). Was `border-2 border-dashed border-c-base` — now plain text. Lower visual weight; user said "I want it to be less obvious that it's there."

605. **Section label divider line removed** (`BbLogger.jsx:2160` `GroupLabel`). Dropped `<div className="flex-1 h-px bg-current opacity-20" />`. Section labels (PRIMARY / CHOOSE 1 / IF YOU HAVE TIME) now render as bare uppercase text with no trailing line.

606. **Numpad Done button removed entirely** (`src/components/CustomNumpad.jsx`). Per-card `Mark as Done` becomes the single way to mark an exercise complete. The numpad's `Next →` is promoted from `outlineKeyStyle` to `accentKeyStyle` — filled accent color, contrast text, fontWeight 700. `handleDone` callback deleted. The legacy `onDone` prop in the numpad config typing stays for backwards compatibility (BbLogger still passes it from useNumpadField hooks); just no longer rendered. User decision: removing the numpad's Done eliminates the redundancy that produced inconsistent collapse + scroll behavior.

607. **Add-exercise crash defenses** (`BbLogger.jsx`). Three layers: (a) `(query || '').trim()` null-safety in `handleAddTyped`; (b) try/catch around `findSimilarExercises` so an exotic-input throw falls through to the create modal instead of crashing the logger; (c) user-visible `alert("Couldn't add exercise: {message}")` wrapper around the parent's `handleCreateSave` so any throw from `addExerciseToLibrary` (validation failure, type-aware constraint mismatch, hyrox-round roundConfig requirement) surfaces as a friendly error and keeps the modal open instead of bubbling up unhandled and white-screening the page.

608. **Auto-tag gym + library write on Machine chip commit** (`BbLogger.jsx` ExerciseItem + EquipmentInstancePopover wiring). When the user types a non-empty machine value AND `currentGymId` is set, the popover's `onChange` now dual-writes:
    - `setDefaultMachineByGym(libraryEntry.id, currentGymId, value)` → the value shows up in the Exercise Library's Gyms section as the per-gym default. Previously only ExerciseEditSheet wrote to this field; mid-session entries lived only in session history, so users had to re-type the same value in the library editor to see it persist.
    - `addExerciseGymTag(libraryEntry.id, currentGymId)` → auto-tags the gym (mirrors ExerciseEditSheet Save flow per Batch 48). Typing a machine value implicitly says "this exercise IS at this gym," so the gym is tagged silently. Side benefit: the auto-tag prompt would never fire for that (exercise, gym) pair again — but the prompt is gone now anyway (see #609).
    Closes the data-continuity gap the user called out: "I have tagged or named the machine type on a plate-loaded or selectorized machine mid-session. Those values are definitely not showing up in the exercise library under the gym section. As a user, nothing is worse than having to input something two times."

609. **GymTagPrompt banner removed from BbLogger ExerciseItem.** User feedback: "It starts to get a little bit overwhelming, the number of prompts that appear mid-session." Deleted the entire `<GymTagPrompt>` block + the `showGymTagPrompt` predicate + the unused store subscriptions (`addSkipGymTagPrompt`, `addHiddenAtGym`, `dismissGymPrompt`) inside ExerciseItem. With the prompt gone, the auto-tag path from #608 handles the silent case for machine exercises; non-machine exercises don't get auto-tagged at all (no Machine chip exists for them) and the user manages availability via the Exercise Library's Gyms section. The `GymTagPrompt` component export in `Recommendation.jsx`, the `settings.dismissedGymPrompts` slice in `useStore.js`, and the `dismissGymPrompt` action are all left on disk as harmless dead code; cleanup sweep deferred.

610. **Per-exercise notes redesign — pencil pill + Mark as Done share one row** (`BbLogger.jsx` ExerciseItem). The dedicated full-width "Notes for this exercise…" input row is gone. Replaced with a single flex row inside the expanded card: a 1/3-width pencil pill on the left, the 2/3-width primary action (Mark as Done / Finish superset / Undo completion) on the right. Pencil pill is muted `bg-item border-transparent` when notes are empty; flips to filled accent (`${theme.bgSubtle} border ${theme.border} ${theme.text}`) when notes are set so the user can see at a glance which exercises carry context. Tap toggles `notesOpen` — when expanded, an inline text input appears below the row, auto-focused. Blur with empty value auto-collapses; Enter or Escape commits + collapses. `lastExNotes` italic line stays but only renders when the editor is closed (avoids double-stacking). Layout applies to all three action states (not done / not done in superset / done). User UX note: removes the always-visible textarea row that took ~50px of card real estate even when the user didn't have notes for 90% of exercises.

611. **Session Notes redesign — button-styled like + Add Exercise** (`BbLogger.jsx`). The 3-row textarea card at the bottom of the workout list is gone. Replaced with a button matching the `+ Add Exercise` styling — full-width, no border, `text-c-muted`, font-medium, slightly smaller py (`py-3` vs `py-4`), `mt-1` gap so it sits just beneath. Tap to expand a 3-row textarea below; auto-focus on open, blur with empty value auto-collapses. Default state: closed for fresh sessions, open if the saved `activeSession.sessionNotes` already has content (so resuming a session with notes doesn't require an extra tap to see them). User UX note: matches the visual hierarchy of "+ Add Exercise" — both are tap-to-reveal CTAs, neither dominates the page.

612. **Plate setup bar weight bug — could not reproduce, deferred.** User reported the popover's bar selection didn't apply on first Confirm tap; required reopen + reselect. Walked the code path against a clean install with synthetic state: single tap of "None" correctly committed `barDefault: 45 → 0` + every `set.barWeight: undefined → 0` + recomputed `set.weight: '' → '0'` via `calcTotal(emptyPlates(), 0, 2)`. With one 45 plate loaded (weight=135), tapping 25 lb → `barDefault: 45 → 25`, `set.barWeight: 45 → 25`, `set.weight: '135' → '115'` (correct: 25 + 2×45). Batch 34's `onBarChange` always-stamp pattern appears to be working correctly post-deploy. Suspect the user's repro depends on specific localStorage state from a session that pre-dated the Batch 34 hotfix or a state I didn't construct. Punted: when the user can capture a backup right after reproducing, replay against the captured state should pin the divergence.

613. **Files modified.**
    - `src/components/CustomNumpad.jsx` — Done button + `handleDone` removed; `Next →` uses `accentKeyStyle`.
    - `src/pages/log/BbLogger.jsx` — items 1, 2, 3, 7, 10, 11 (largest diff): + Add Exercise styling, GroupLabel, handleAddTyped + handleCreateSave defenses, EquipmentInstancePopover dual-write, GymTagPrompt block deletion + state cleanup, per-exercise notes redesign (pencil pill + Mark as Done shared row + inline-expand input), Session Notes redesign (button + inline-expand textarea).

614. **Build.** `npx vite build --outDir /tmp/test-build` → 895.53 KB bundle / 241.01 KB gzipped (−0.11 KB / −0.01 KB vs Batch 49 — net wash). Items 1/2/3/7 verified live (port 5184, mobile 375×812): clean labels, no dashed border, gym auto-tag confirmed end-to-end via `localStorage` round-trip showing `defaultMachineByGym: { gym_vasa: "TestHoist" }` + `sessionGymTags: ['gym_vasa']` after typing one machine value. Items 5/10/11 not preview-verified — pushed straight to main per user instruction; user will report from production.

### Batch 51 (April 25–26, 2026) — Dashboard v2 (B51)

615. **Dashboard rewrite.** Hero card owns the fold; narrative replaces the v1 6-stat grid; sparkline + computed weekly sentence; tier-colored streak; particles preserved. Iterated across ~12 polish rounds with the user. See `HANDOFF-B51.md` for the full retrospective and the design principles distilled (one hero per fold, factual labels over interpreted ones, sparklines need a metric label, daylight contrast pattern).

### Post-B51 stability pass (April 27, 2026) — three crash fixes + ESLint + ErrorBoundary

The day after Dashboard v2 merged, two render crashes surfaced (one for the user himself on a HYROX split, one for his brother on the deployed PWA). Both bugs were specific code paths the dev loop never exercised during ~12 polish rounds. This pass fixes both and ships defensive layers so the next render bug becomes a recoverable banner instead of a fully gray screen.

616. **Dashboard `missedYesterdayWorkout` ReferenceError fix** (`8c91576`). Line 732 of `Dashboard.jsx` referenced a variable that was deleted during the v2 rewrite ("picking up from yesterday" branch was removed per HANDOFF-B51 design principle #7) but one orphan reference was left behind. JS `&&` short-circuit evaluation hid the bug for any rotation without `'rest'` entries (BamBam's Blueprint), so v1's polish loop never tripped it. Crashed deterministically for any user whose split has rest days when today's slot lands on rest. Fix: drop the `&& !missedYesterdayWorkout` clause. One-line change.

617. **Lesson learned: previous-session "don't revert" advice was overstated** (post-mortem). The April 26 black-screen incident was misdiagnosed as a cache problem, leading to a fix-forward path (vercel.json cache headers, /recovery.html, boot-time diagnostic) that was useful infrastructure but didn't address the actual bug. The previous session pushed back on the user's revert instinct citing "asset hash churn risk" — that argument was technically true but practically minimal. **Heuristic going forward: if a known-good revert exists, take it first, then debug forward.** Reverting is reversible; users hitting a crash for hours while we investigate isn't.

618. **CreateExerciseModal Rules of Hooks violation fix** (`f05aaa4`). `useMemo` for `canSave` was declared AFTER `if (!open) return null`. When the modal toggled open, hook count differed between renders → React threw "Rendered more hooks than during the previous render" → whole tree unmounted → fully gray screen with no UI. Trigger: Exercise Library (`/exercises`) → tap "+ New" button. Fix: lift the `useMemo` above the early return. A codebase-wide audit found no other instances of this pattern.

619. **Top-level ErrorBoundary** (`006892c`, `src/components/ErrorBoundary.jsx`). Wraps the route tree + chrome (Routes / HamburgerMenu / BottomNav / RestTimer / Toast) so any future render error renders a recovery banner instead of unmounting the app. Banner shows the error message + collapsible stack/component-tree details + a Reload button (cache-busts via query string) + a link to `/recovery.html`. Captured errors push onto `window.__appErrors` so the boot-time diagnostic from the April 26 incident still sees them on screenshot probes. Future render bugs are now recoverable in-place without delete-and-reinstall cycles.

620. **ESLint with `rules-of-hooks` + `no-undef` as errors** (`35e01d0`). Both bugs above would have been caught at edit time by lint:
    - `no-undef` → catches dangling identifier references like `missedYesterdayWorkout`.
    - `react-hooks/rules-of-hooks` → catches hooks called after early returns or inside conditionals.
    Style rules (semi/quotes/indent) deliberately off — no churn pass on 100+ files. `exhaustive-deps` and `no-unused-vars` stay as warnings. `npm run lint` runs `eslint src --quiet`. Current state: 0 errors, 2 trivial unused-var warnings in `helpers.js` (`belowFloor`, `thisRunLeg`).

621. **CLAUDE.md Rule 5 added: run `npm run lint` before every commit.** Plus a "Pre-flight checklist for redesign batches" section above with concrete steps to walk three state combinations (empty / weight-only / HYROX-active) before merging. Both v2 bugs would have been caught by following that checklist.

622. **Files modified.**
    - `src/pages/Dashboard.jsx` — one-line `missedYesterdayWorkout` fix.
    - `src/components/CreateExerciseModal.jsx` — `useMemo` lifted above early return.
    - `src/components/ErrorBoundary.jsx` (new) — class component with getDerivedStateFromError + componentDidCatch.
    - `src/App.jsx` — wrap Routes + chrome in `<ErrorBoundary>`.
    - `package.json` — `lint` script added.
    - `.eslintrc.cjs` (new) — config with rules-of-hooks + no-undef as errors.
    - `CLAUDE.md` — Rule 5 (lint), Pre-flight checklist section, this entry.

623. **Verification.** Reproduced both bugs live in dev preview before fixing. Verified both fixes live in dev preview after fixing. Ran a runtime sweep of all 9 main routes + all 6 HYROX workouts + all 5 BamBam workouts + key surfaces (StartHyroxOverlay, ExerciseEditSheet on HYROX-round, ChooseStartingPoint, hamburger menu, tier modal, preview overlay, custom accent) — zero new errors anywhere. Lint passes with 0 errors. Build passes (897.41 KB bundle / 241.39 KB gzipped post-ErrorBoundary, +1.78 KB from boundary code).

624. **Open followups.**
    - **Storage-failure UX defense** (toast on save failure / quota banner) — defer until Supabase migration so it gets built once, not twice. Pre-Supabase silent localStorage failures are a real concern (might explain BAMBAM's empty `sessions[]`) but the right fix is integrated with the offline-queue infrastructure that Supabase Phase 1 needs anyway.
    - **`useStore.js` `addSession` audit** — clean; the silent-failure path is in Zustand's default localStorage adapter, not in user code. Documented in HANDOFF-BLACK-SCREEN-DEBUG.md.
    - **CI for lint** — `npm run lint` on push via GitHub Actions. Not shipped today; user can add when ready.
    - **B60 SplitManager polish** — queued as the next redesign batch (small, low-risk warm-up after this incident). See `HANDOFF-B60.md`.

### Batch 52 (April 27, 2026) — SplitManager polish (provenance fade + Built-in pill)

First post-stability-pass batch. Small confidence-rebuilder per `HANDOFF-B60.md` — feel the new `ErrorBoundary` + ESLint safeguards work end-to-end on a low-stakes surface before tackling Progress or Logger v2. Two surgical edits to `SplitCard` in `SplitManager.jsx`. No store changes, no schema changes, no route changes. Spec: `Gains-Design-Critique.md` §4 + `Gains-Design-Mockups.html` lines 965–973 + 2032–2153.

625. **Provenance line fades on inactive cards** (`SplitManager.jsx:325`). Added `&& isActive` to the `provenance && (...)` render condition. The `Started March 22, 2026` / `Created April 12, 2026` line is useful context on the active card but row noise on inactive cards. Users can see provenance dates via the edit screen if they care. One-line change.

626. **"Built-in" pill on built-in cards** (`SplitManager.jsx:288-296`). New `<span>` in the existing top-row flex layout between title and `⋯` overflow button, gated on `split.isBuiltIn === true`. 9px / uppercase / `tracking-[0.08em]` / `font-bold` / `text-c-faint` / transparent bg — matches the mockup's `.split-builtin-pill` CSS spec but nested into the flex row instead of absolute-positioned, so it can't overlap the `⋯` button. `shrink-0 self-start mt-1.5` aligns the pill's baseline with the title row text. Theme-neutral (uses `text-c-faint`, not `theme.hex`), so no daylight + light-accent risk. Distinguishes built-in starting templates from user-created and user-imported splits at a glance.

627. **Verification.** Lint exit 0, build clean (895.87 KB bundle / 241.68 KB gzipped — no measurable change). Three state combinations walked in preview at 375x812: BamBam active + built-in (pill + provenance both render), BamBam inactive + built-in via toggling HYROX active (pill renders, provenance hidden), HYROX active + custom (no pill, provenance renders, Active pill renders). `⋯` menus correct on both card types — built-in shows Edit/Duplicate/Export, custom shows Set Active/Edit/Duplicate/Export/Delete. Daylight + white accent passes — pill correctly uses daylight `--text-faint` (`#aaaaaa`). Computed pill styles match mockup spec exactly (9px / 700 / 0.72px = 0.08em / `text-c-faint` / uppercase / transparent bg / 6px top margin). Console clean of new errors and new warnings.

### Batch 53 (April 27, 2026) — ExercisePicker hybrid search + per-row usage counts + Logged/Never filter

User feedback after a Lanhammer split-builder session: "search for an exercise that didn't appear, and then I would re-add it. I'm wondering if I'm creating a duplicate." Investigation found the picker's main search at [ExercisePicker.jsx](src/components/ExercisePicker.jsx) used pure substring (`name.toLowerCase().includes(q)`) — fuzzy `findSimilarExercises` was wired only to the `+ Or add your own` create path, after the user had already seen "no match" and decided to re-add. So `"DB Lateral"` failed to surface `"Lateral DB Raises"` (token-order variant), `"Pec deck"` (typo) failed to surface `"Pec Dec"`, etc. Plus no usage signal anywhere — users with multiple similar entries couldn't tell which had session history without leaving the picker. Spec: `.claude/plans/exercise-picker-search-and-usage.md`. One-file change.

628. **Hybrid three-tier search** (`ExercisePicker.jsx` `searchView` useMemo). When a query is typed, results are layered:
    - **Tier 1 — substring** (`name.toLowerCase().includes(q)`): familiar, strong-intent matches first.
    - **Tier 2 — token-subset** (multi-word queries only) with light s-stemming: query's tokens all present in candidate in any order. `"db lateral"` → `"Lateral DB Raises"`. `"curl hammer"` → `"DB Hammer Curls"` (curl ≈ curls via the stemmer that drops trailing `s`).
    - **Tier 3 — trigram jaccard ≥ 0.5** via existing `findSimilarExercises`: typos and partial-overlap matches. `"pec deck"` → `"Pec Dec"`. `"leg presss"` → `"Leg Press"`.
    The seen-set short-circuits dedup across tiers so the same row never appears twice. Tiers 2 + 3 entries get a `≈` glyph prefix (subtle muted color, `aria-label="Similar match"`) so users understand why a non-substring match is showing. The original 0.85-threshold create-path dedup at `handleAddCustom` is unchanged.

629. **Per-row distinct-session usage count.** New `useCountByKey` useMemo subscribes to `useStore(s => s.sessions)` and walks all bb-mode sessions once, incrementing `map[exerciseId || name]` per occurrence. Drops are nested in their parent set, not double-counted at the exercise level. Lookup per row: `useCountByKey[ex.id] ?? useCountByKey[ex.name] ?? 0`. Display in a secondary line beneath the exercise name: `Logged 8 times` / `Logged once` / `Never logged` (the never case rendered in `text-c-faint` to de-emphasize). The user can now visually distinguish "the variant with 5 sessions" from "the never-logged duplicate" without leaving the picker.

630. **Logged / Never logged filter chip.** New chip row above the muscle tabs (`All / Logged / Never logged`) — orthogonal to the muscle axis, composes with it. Default `All`. Mirrors the Batch 16l pattern from `/exercises`. Empty-state copy adapts: `"No logged exercises here yet."` when filter is Logged + 0 hits; `"Every exercise here has been logged."` when filter is Never + 0 hits.

631. **Sort default in the All tab when browsing.** When no query AND active tab is `All`, sort by usage-desc then alphabetical. Lands the user on their high-history entries first — answers "what have I been training?" at a glance. Recent + muscle tabs preserve their natural source order (alphabetical from EXERCISE_LIBRARY) so the user's mental model of where things live in those slices doesn't shift. With a query, tier order wins (substring → token-subset → trigram).

632. **`picker-search-sanity.mjs` at worktree root.** 14/14 pass. Mirrors the picker's three-tier logic against a 12-entry synthetic library. Validates: token-order via Tier 2 (lateral db ↔ DB Lateral Raises both ways), s-stemming (curl hammer → DB Hammer Curls), trigram typos (pec deck → Pec Dec, leg presss → Leg Press), substring same-order, case-insensitive substring, token-subset for word-spread queries (overhead extension → Overhead DB Extension), and false-positive ceiling (banana / qwerty zxcvb → 0 matches). Plus a noisy short-query check confirming `press`/`row`/`curl` don't pull noise — they hit only their substring matches.

633. **Verification.** `npm run lint` exit 0. `npx vite build` → 897.76 KB bundle / 242.43 KB gzipped (+1.89 KB / +0.75 KB vs Batch 52, accounted for by the three useMemo blocks + tier-search loops + chip row + per-row metadata line). Live preview walk at 375×812 in dev preview (port 5173) with synthetic sessions seeded (Pec Dec ×5, Flat Bench Press ×3, Leg Press ×1):
    - Picker opens from SplitCanvas → Push — Chest → expand Primary section → `+ Add exercise`.
    - **Sort default verified**: All tab, no query → top rows render in usage-desc order (Pec Dec `Logged 5 times` → Flat Bench Press `Logged 3 times` → Leg Press `Logged once` → never-logged entries alphabetically).
    - **Token-subset verified**: query `"lateral db"` → `Lateral DB Raises` (substring, no marker), `≈ DB Lateral Raises` (Tier 2, ≈ marker), `≈ DB Lateral Raise` (Tier 2). Exactly the case the user reported as broken.
    - **Trigram typo verified**: query `"pec deck"` → `≈ Pec Dec Logged 5 times Added` (Tier 3, ≈ marker, with usage count visible).
    - **Logged-only filter verified**: chip `Logged` → only the 3 logged rows show; chip `All` restores the full list.
    - Console clean of new errors. The remaining errors (CreateExerciseModal Rules of Hooks via ExerciseLibraryManager, BbLogger ExerciseItem key-spread) all predate this batch and are documented known caveats.

634. **Future merge UI follow-up — shipped in B54** (was previously deferred). With B53's usage counts visible in the picker the user could spot duplicates, but acting on them required deletion (which loses history). User asked for explicit merge so history stays. See B54.

### Batch 54 (April 27, 2026) — User-initiated merge UI in ExerciseEditSheet

Surfaces the existing `mergeExercises(keepId, mergeIds)` store action (alive since Batch 15 but only ever called by the v3 migration's auto-dedup) through `ExerciseEditSheet`. Closes the loop on B53's usage-counts: now that you can SEE which duplicate has history, you can fold the never-logged sibling into the canonical entry without losing anything.

Investigation found the seed library at [exerciseLibrary.js:51-52](src/data/exerciseLibrary.js) literally seeds two variants `DB Lateral Raises` + `Lateral DB Raises` as separate `isBuiltIn` rows — the case the user reported is real and pre-existing in the data file. Plus user-flow duplicates can land via the v3 migration creating `needsTagging` entries from unresolved session names. Either way: the merge UI handles both.

635. **`mergeCandidates` useMemo in `ExerciseEditSheet.jsx`.** Calls existing `findSimilarExercises(exercise.name, others, { suggestThreshold: 0.7, max: 8 })` against all library entries except the current one AND filtered to entries of the same `type`. Cross-type merges (e.g. weight-training entry into running entry) are blocked because their schemas differ — `dimensions` / `roundConfig` / muscle tags don't survive a cross-type fold.

636. **`useCountByKey` useMemo.** Same shape as B53's ExercisePicker counter — walks bb-mode sessions once, increments per `exerciseId || name`. Drives the candidate list's usage labels + the confirm dialog's pre/post counts.

637. **"Merge into another exercise…" button** rendered above the existing Delete/Save action row. Visibility gated on `mergeCandidates.length > 0 && !confirmingDelete` — hidden when no similar entries exist OR when the user is mid-delete-confirm (don't pile destructive actions). **Built-ins are not gated** — the seed bug at line 51-52 means even built-in entries need merge ability. Button label: `Merge into another exercise… (N similar)` so the user sees the candidate count before committing the next tap.

638. **Two-phase merge view** (replaces the form when active):
    - **Phase 1 — Picker**: `Merge "{currentName}" into…` heading + subtitle showing how many sessions will move ("2 sessions on this entry will be reattributed. This entry will be deleted." OR "This entry has no logged sessions. It will be deleted." for the zero case). Each candidate row shows name + `Logged N times` / `Logged once` / `Never logged` + match percentage (e.g. `95% match` for token-sort variants like the Lateral DB pair). Single-select — tap a candidate → Phase 2.
    - **Phase 2 — Confirm dialog**: `Merge?` header + bullet showing source → target with both session counts, plus a clear consequence line. Two buttons: Back (returns to Phase 1) + Merge (commits via the store action). The merge action call is `mergeExercisesAction(target.id, [exercise.id])` — target wins as `keepId`, current entry's id is absorbed. Toast fires with "Merged X into Y — N sessions reattributed", then `onCancel()` closes the sheet (the current entry no longer exists).

639. **Why current-into-target direction.** The mental model: user opens the duplicate they want gone, picks the canonical entry as the target. This matches "this is a dupe, send it home." If the user opens the canonical first and wants to absorb a dupe, they'd have to switch to the dupe's sheet first. Documented in the picker's heading text: `Merge "Lateral DB Raises" into…`. Single-select per merge action — for the rare case where multiple dupes exist, the user merges them one at a time. Multi-select is a future improvement if it surfaces as a real friction.

640. **Verification.** `npm run lint` exit 0. `npx vite build` clean (~898 KB). Live preview walk at 375×812 with synthetic seed (2 sessions on `Lateral DB Raises`, 4 on `DB Lateral Raises`):
    - Open `/exercises` → tap `Lateral DB Raises`. Sheet opens with normal edit form.
    - Merge button visible: `Merge into another exercise… (1 similar)`.
    - Tap → Picker renders with `Merge "Lateral DB Raises" into…` heading + `2 sessions on this entry will be reattributed.` subtitle. Candidate row: `DB Lateral Raises · Logged 4 times · 95% match`. Back button works.
    - Tap candidate → Confirm dialog: `"Lateral DB Raises" (2 sessions) → "DB Lateral Raises" (4 sessions)` with consequence `2 sessions will be reattributed to "DB Lateral Raises". "Lateral DB Raises" will be deleted.`. Back/Merge buttons.
    - Tap Merge → library count 128 → 127 (`Lateral DB Raises` removed); all 6 sessions now point to `ex_db_lateral_raises` (4 + 2 reattributed); sheet auto-closes; toast shows confirmation message.
    - Console clean of new errors. Pre-existing `BbLogger ExerciseItem` key-spread warnings remain (documented).

641. **What's preserved vs. lost.** Sessions that pointed at the absorbed entry now point at the keep — volume / PR history all moves. The absorbed entry's library metadata (name aliases, sessionGymTags, hiddenAtGyms, defaultMachineByGym, repRangeUserSet, etc.) is **lost**. Keep wins all metadata. If the user merges the wrong direction (canonical absorbed into dupe), there's no undo — they'd need to manually re-tag. The confirm dialog spelling out source → target with names is the safety net.

### Batch 55 (April 27, 2026) — A few adjustments: focus mode + plate alignment + Incline Treadmill + WEEK rotation + cross-type ghost rows

Five live-feedback items from a HYROX session that morning. Three surgical UI tweaks (1, 2, 3); one small data-flow change (5); one meaningful semantic addition to the split rotation system (4) — addressing the user's reported "today is Monday but Dashboard shows Push" bug. Spec at `.claude/plans/a-few-adjustments-soft-flame.md`.

642. **Item 1 — Hide HYROX section preview in focus mode** (`BbLogger.jsx:4128`). When the numpad is open during a hybrid workout (e.g. HYROX Hybrid → Tuesday with Lift section + HYROX section), the immersive yellow HYROX preview card stays visible and visually crowds the weight-set inputs. Now mirrors the existing `GroupLabel` hide predicate at line 4171 — when `numpadIsOpen`, the HYROX render branch returns null. One-line gate alongside the existing `isHyroxSection && !group.isActiveSuperset` check.

643. **Item 2 — Plate row left-padding alignment** (`PlateSetRow` at `BbLogger.jsx:580`). The 100/45/35/25/10/+ plate buttons sat directly underneath the yellow "Work" Set Type chip — visually cramped against the accent. User: "I would like the 100 lb increment and all subsequent increments to begin on the right of that vertical line." Fixed by wrapping the existing `PlatePicker` call in a `<div className="pl-16">` (4rem = 64px = `w-14` SetTypeBtn + `gap-2`). Plate buttons now align with the start of the Total column. Verified live: Set Type "Warm" left=32 right=88, plate "100" starts at left=96 (8px gap matching the top row's gap-2). PlatePicker itself is unchanged — just the wrapping div.

644. **Item 3 — Add Incline Treadmill cardio type** (`CardioLogger.jsx:10` + `:22`). Prepended `'Incline Treadmill'` to `BUILTIN_TYPES` so it renders FIRST in the type grid. Updated `getDistanceUnit` to return `'miles'` for Incline Treadmill (same as regular Treadmill). No Dashboard / History updates needed — both read the type string verbatim. Incline % / elevation gain dimension deliberately not added; user's notes field is sufficient for v1.

645. **Item 4a — `rotationMode` field + persist v9→v10 migration** (`useStore.js`, `helpers.js`, `splitTemplates.js`). New optional field on `Split`: `rotationMode: 'cycle' | 'week'`. Defaults to `'cycle'` everywhere it's missing — preserves the legacy session-anchored behavior for BamBam's Blueprint and every pre-Batch-55 user split. New `migrateSplitsToV10(splits)` helper in `helpers.js` walks each split and stamps `rotationMode: 'cycle'` if absent (idempotent, returns same reference when nothing changed). Persist version bumped `9 → 10` with the additive migration block. HYROX Hybrid template (`splitTemplates.js:264`) gains `rotationMode: 'week'` since its 7-day rotation maps Sun=rest, Mon-Sat=workouts by design. `loadTemplateForDraft` extended to preserve `rotationMode` through draft serialization.

646. **Item 4b — `getRotationItemOnDate` + `getNextBbWorkout` honor `rotationMode`** (`helpers.js:217` + `:414`). New optional `rotationMode` arg (default `'cycle'`). When `rotationMode === 'week'` AND `rotation.length === 7`:
    - `getRotationItemOnDate(dateStr, ...)` parses `dateStr + 'T00:00:00'` as local midnight, returns `rotation[date.getDay()]` directly. Bypasses session-anchor logic entirely so today always gets today's slot.
    - `getNextBbWorkout(sessions, ...)` walks forward from today's day-of-week up to 7 days, returns the first non-rest slot.
    Defensive: rotationMode='week' with non-7-length rotation falls through to cycle behavior so a malformed split doesn't crash. Cycle mode preserves the legacy behavior exactly.

647. **Item 4c — SplitCanvas saves/loads `rotationMode`** (`SplitCanvas.jsx:82` + `:131` + `:283`). `rotationView` useState now initializes from `existingSplit?.rotationMode || 'cycle'` so reopening a week-mode split shows the WEEK toggle. Mount-time draft pre-load restores rotationMode from the draft if present. `handleSave` includes `rotationMode: rotationView` in `splitData`, with a defensive guard that forces `'cycle'` when `rotationView === 'week'` but `rotation.length !== 7`. Auto-save useEffect now writes `rotationMode` into the draft so toggle changes persist through reload.

648. **Item 4d — Dashboard + Log thread `rotationMode`** (`Dashboard.jsx:600-601` + `:765` + `:797`, `Log.jsx:103-105`). All four call sites of `getRotationItemOnDate` + `getNextBbWorkout` now pass `activeSplit?.rotationMode || 'cycle'`. Dashboard reads it once via `const rotationMode = activeSplit?.rotationMode || 'cycle'` and threads to all rotation-aware helpers (today's CTA card, weekly calendar's `getFullRotationItem`, monthly's past-rest detection). Log.jsx threads it to the picker's "next workout" detection. Every consumer is now rotation-mode-aware.

649. **Item 4e — `batch-55-rotation-sanity.mjs`** at worktree root. 30/30 pass. Covers (1) week-mode `getRotationItemOnDate` across 14 calendar days mapping Sun→Sat correctly + the user's bug repro (Monday in week mode returns Quads, not rotation[0]=Push fallback); (2) defensive cases — week mode with non-7-length rotation falls to cycle, empty rotation returns null; (3) cycle mode preservation — same-day-as-anchor returns anchor's slot, 1-day-after returns next slot, default mode (omitted arg) behaves as cycle; (4) week-mode `getNextBbWorkout` — all-rest 7-day returns undefined, 5-day rotation in 'week' mode falls back to cycle returning sequence[0]; (5) `migrateSplitsToV10` adds default to missing entries, preserves existing 'week', returns same reference when idempotent, defensive against null/undefined/empty.

650. **Item 5 — `lastExDataByName` cross-workout-type fallback** (`BbLogger.jsx:2999`). The pre-Batch-55 build only scanned `allPastSessions` (filtered to `s.type === type`), so a fresh split with new workout-type IDs got an empty `lastExDataByName` even though the user had logged Squat under a different type. Asymmetric with the recommender's `recHistory` (cross-workout-type via `getExerciseHistory`) — the user saw Coach's Tip but no Last Time ghost rows. Fix: third pass after the existing same-type/same-gym → same-type/any-gym chain. Walks ALL bb-mode sessions newest-first and fills in any still-missing exercise names. Same-type passes still win — no behavior change for active splits with their own history. Fields preserved exactly (sets, plates, barWeight, unilateral, equipmentInstance) so all downstream consumers (ghost row sets, plate-mode init, Uni toggle init, equipment instance seed) keep working.

651. **Live preview verification** (port 5185, mobile 375×812). All 5 items pass:
    - **Item 1**: HYROX Hybrid → Tuesday → Cable Lateral Raise expanded → focus weight input → numpad opens → Start HYROX button no longer in DOM (rendered=false, rect=null), HYROX preview wrapper gone. Confirms render-time gate.
    - **Item 2**: BamBam → Push → Flat Bench Press → Plates ON → plate row "100" at left=96 = 32 + 64 (Set Type chip ends at 88). pl-16 wrapper applied correctly.
    - **Item 3**: `/cardio` page renders 7 type chips with `Incline Treadmill` first.
    - **Item 4**: `/splits/new/start` → tap HYROX Hybrid template → splitDraft seeds `rotationMode: 'week'` → SplitCanvas WEEK toggle pre-selected → Save → split persists with `rotationMode: 'week'` → Activate → Dashboard renders `🍑 Glutes & Light Run` (rotation[1]) for Monday. BamBam stays in cycle mode unchanged.
    - **Item 5**: Seed prior session under workout-type 'push' with Squat (135×5 warmup, 225×8 working) → navigate to BamBam's Pull (different workout type, no Squat history under 'pull') → Add Exercise → Squat → expand → Last Time ghost rows render `Warm 135×5` and `Work 225×8`. Cross-workout-type fallback works.
    - Console clean of new errors. Pre-existing `<ExerciseItem>` key-via-spread warnings at `BbLogger.jsx:3837` are documented caveats predating B41.

652. **Files modified.**
    - `src/pages/log/BbLogger.jsx` — Items 1, 2, 5 (HYROX hide gate + PlatePicker pl-16 wrapper + lastExDataByName third pass).
    - `src/pages/CardioLogger.jsx` — Item 3 (BUILTIN_TYPES + getDistanceUnit).
    - `src/utils/helpers.js` — Item 4 (`migrateSplitsToV10` + week-mode branches in `getRotationItemOnDate` + `getNextBbWorkout`).
    - `src/store/useStore.js` — Item 4a (persist v10 migration + import).
    - `src/data/splitTemplates.js` — Item 4a (HYROX Hybrid `rotationMode: 'week'` + loadTemplateForDraft preservation).
    - `src/pages/SplitCanvas.jsx` — Item 4c (rotationView init from existingSplit + draft pre-load + handleSave commit).
    - `src/pages/Dashboard.jsx` — Item 4d (3 call sites threading rotationMode).
    - `src/pages/Log.jsx` — Item 4d (1 call site threading rotationMode).
    - `batch-55-rotation-sanity.mjs` (new) — Item 4e (30 assertions).

653. **Build.** `npx vite build --outDir /tmp/test-build` → 902.61 KB bundle / 243.67 KB gzipped (+5.24 KB / +1.59 KB vs Batch 54, accounted for by the rotation-mode helper branches + migrateSplitsToV10 + Dashboard/Log threading + SplitCanvas mode wiring + the lastExDataByName third pass).

### Batch 56 (April 28, 2026) — Monthly Coaching Summary card on /progress

First of three Progress-page redesign batches per `Gains-Design-Critique.md` §2 (the design doc internally calls this "B53"; landing as Batch 56 in the sequential changelog). Reframes Progress from a static report into a coaching surface. Pure additive — yellow HYROX-tinted card pinned above the existing 5 stat tiles (Weekly Load / Duration / Radar / PR Timeline / Heatmap), no edits to those tiles. Hidden when < 3 bb-mode sessions logged in the rolling 30-day window so new users don't see a card screaming "log more data."

User-locked decisions (from planning round): rolling 30-day window (not calendar month — smooth, no end-of-month jitter); hybrid scope (lift PRs + HYROX rounds + running all contribute); cold-start hides the card entirely. Card uses HYROX yellow `#EAB308` independent of user's accent — same brand cue as the round logger / Start overlay / summary screen.

654. **`buildMonthlyCoachingSummary({sessions, cardioSessions, restDaySessions, splits, activeSplitId, now})` in `helpers.js`.** Pure aggregator over the rolling 30-day window. Returns `{eyebrow, headline, bullets[], suggestion | null, meta} | null`. Composes existing engine helpers — `calcSessionVolume` walker pattern, `getExerciseHistory` + `getProgressionRate`, `detectRegression` + `detectPlateau` (swing intentionally skipped — too noisy for a monthly read), `getWorkoutStreak` + `getBestStreak`, plus Batch 45's `getHyroxSessionTotalTime` for HYROX session totals. Defensive against null / non-array / malformed inputs at every layer. Uses `toLocalDateStr` semantics implicitly via `Date.parse` — same timezone behavior as the rest of the app.

655. **Headline picker — single line, highest-impact wins.** Priority: PR-led (≥3 PRs → `Strong push month. 3 PRs on Bench Press.`) > Streak milestone (`currentStreak === bestStreak && ≥7` → `12-day streak. Your best run yet.`) > HYROX-led (≥3 HYROX sessions → `Strong HYROX month. 4 sessions, fastest 42:30.`) > Volume-led (`±15%` delta → `Up 12% volume. Your strongest 4 weeks.` / `Volume down 22%. Your reset month.`) > Default (mirrors Dashboard v2's narrative — `5 sessions this month, 1 ahead of last.`). Push/pull/legs framing inferred from the top-PR exercise's name via `categoryForExerciseName` keyword scan.

656. **Bullet composer — at most 3, ordered by impact.** Skips bullets that duplicate the headline (volume bullet hidden when headline is already volume-led; HYROX bullet hidden when headline is HYROX-led). Composes from: volume delta line, top-progressor `{exercise} trending +{rate}%/wk across {n} sessions.`, HYROX session count + fastest, `{streak}-day streak holding (best: {best}).`, and PR count breakdown. Top-progressor pickup gates at `n ≥ 4 && rSquared ≥ 0.6 && rate ≥ 0.5%/wk` to filter low-confidence fits.

657. **Suggestion line — render only when triggered.** Three branches: regression anomaly fires `⚠ {exercise} has dipped {n} sessions. Consider a recovery week.` (highest priority); workout-type drift fires `💡 {workout} sessions down {pct}%. Want to add a {workout} day this week?` (gates at `≥3 prior sessions && ≥30% drop`, scoped to the active split's workouts when `activeSplitId` is provided); plateau anomaly fires `💡 {exercise} is flat for {n} sessions. Try dropping 10% and chasing reps to break through.` Silence beats verbosity — no "everything looks great" filler when nothing's actionable.

658. **`MonthlyCoachingCard` inline component** (`src/pages/Progress.jsx`). Slotted at the top of the px-4 flex column above `<SectionLabel text="Weekly Training Load" />`. Yellow gradient + border match `Gains-Design-Mockups.html` `.coach-card` spec exactly: `linear-gradient(135deg, rgba(234,179,8,0.08) 0%, rgba(0,0,0,0.4) 65%)`, `1px solid rgba(234,179,8,0.18)`, `border-radius: 22`, `padding: 18`. Eyebrow `✨ COACH · LAST 30 DAYS` in `#EAB308` 11px tracking-wider. Headline 19px / 700 / `var(--text-primary)`. Bullets 13px / `var(--text-secondary)` with `·` prefix in yellow. Suggestion box: `rgba(0,0,0,0.35)` bg, `1px solid rgba(234,179,8,0.25)` border, ⚠ for warnings / 💡 for tips. `role="region"` + `aria-label="This month's coaching summary"`.

659. **`formatVolumeShort(n)` helper** (`helpers.js`). Reserved for future use by the bullet composer — `47000 → 47k`, `5500 → 5.5k`, `950 → 950`. Not actively rendered in B56 but stays available for B57 (strength drill-down) when per-exercise volume tiles need a compact format. Defensive against null / non-numeric input.

660. **`b53-coaching-summary-sanity.mjs` at worktree root.** 77/77 pass. Covers: `formatVolumeShort` across 6 cases; cold-start gate (0/1/2/null sessions → null); headline priority (PR-led / volume-up / volume-down / default with session-count math + ahead/behind framing); volume delta math (drops contribute via Batch 22 decision 2, div-by-zero handled when prevVolume=0); top-progressor pickup (n ≥ 4 + rSquared ≥ 0.6 cut, low-confidence n=2 NOT picked); regression anomaly + suggestion copy; plateau detection; regression beats plateau when both fire; swing intentionally NOT surfaced; workout-type drift detection (75% drop on Pull → suggestion fires); HYROX integration (4 hyrox-round sessions earn the headline; lift-only inputs don't surface HYROX bullets); hybrid scope (streak includes cardio + rest); defensive cases (null sessions / non-array / bad dates / missing data / no sets); return shape (eyebrow / headline / bullets array ≤ 3 / suggestion null-or-typed / meta object); bullet composition (top-progressor surfaces in non-headline-PR case); real-data spot check skips gracefully when backup absent.

661. **Verified live in preview** (worktree port 5185, mobile 375×812). Three states pass with zero new console errors:
    - **Empty data** (`sessions: []`, no splits) → card hidden; existing 5 stat tiles render with their canonical empty-state messages.
    - **Populated weight-only** (BamBam Blueprint active, 22 synthetic sessions: 6 Pec Dec progression + 3 Bench Press PRs + 10 Leg Press regression + prior-window comparison data) → card renders `Strong push month. 3 PRs on Bench Press.` headline, two bullets (`Volume up 77% vs last 30 days.` + `Pec Dec trending +9.3%/wk across 6 sessions.`), and the regression suggestion `⚠ Leg Press has dipped 6 sessions. Consider a recovery week.`.
    - **HYROX-active** (HYROX Hybrid week-mode split, 4 Tuesday sessions with 4-round HYROX SkiErg + 3 Friday lift-only) → card renders `Strong HYROX month. 4 sessions, fastest 42:30.` headline + `💡 Cable Lateral Raise is flat for 6 sessions. Try dropping 10% and chasing reps to break through.` plateau suggestion.
    - **Daylight + white accent** test passes — yellow eyebrow holds contrast, dark `var(--text-primary)` headline + bullets read cleanly on the gradient. Yellow is independent of accent (verified by the explicit `#EAB308` literal in the component, not via theme threading).

662. **Files modified.**
    - `src/utils/helpers.js` — `buildMonthlyCoachingSummary` aggregator + `formatVolumeShort` helper + 6 inline private helpers (sumVolumeInWindow / countPRsInWindow / prCountByExercise / distinctExerciseRefsInWindow / typeCountsInWindow / hyroxStatsInWindow / categoryForExerciseName / workoutNameForType).
    - `src/pages/Progress.jsx` — `MonthlyCoachingCard` inline component + slot above existing tiles + `activeSplitId` added to the `useStore()` destructure.
    - `b53-coaching-summary-sanity.mjs` (new, repo root) — 77-assertion sanity script.
    - `CLAUDE.md` — Last-updated header + this Batch 56 entry.

663. **Deferred for B57 (strength drill-down) and B58 (time-range picker + tile collapse)** per the Critique doc. B56 is purely the coaching headline; the existing 5 stat tiles below stay unchanged. No filter-by-exercise / muscle-group axis this batch — that question surfaced during preview and was deferred to a follow-up after B57 ships, when direct-manipulation drill-down may make a global filter axis unnecessary.

664. **Build.** `npx vite build --outDir /tmp/test-build` → 917.98 KB bundle / 247.84 KB gzipped (+15.37 KB / +4.17 KB vs Batch 55, accounted for by the aggregator + 6 private helpers + the inline MonthlyCoachingCard component).

### Batch 57 (April 28, 2026) — Strength drill-down on /progress

Second of three Progress page redesign batches per `Gains-Design-Critique.md` §2 lines 137–138. Replaces the legacy PR Timeline tile (list-only, hard-to-parse colored dots) with a Strength tile that surfaces actual e1RM trends per exercise. Each row: name + 60×24 sparkline + weekly growth %. Tap → bottom-sheet drill-down with full E1RMSparkline (300×72 with tap-to-select dots), 4 stat tiles (Current e1RM / Weekly growth / Sessions / All-time PR), and the recent-sessions list (newest first, 20 visible + Show all toggle). User-locked decisions: sort by progression rate desc; top 4 with `Show all (N) →` inline expansion; cold-start exercises (<2 sessions) hidden entirely; weight-training only (HYROX rounds + running entries skip the tile).

665. **`buildStrengthTileData({sessions, exerciseLibrary, windowSize = 6})` in `helpers.js`** — pure aggregator. Walks bb-mode sessions once via two passes: pass 1 collects distinct exercise refs (keyed by `exerciseId` with name fallback for pre-v3 sessions), pass 2 builds per-exercise history via existing `getExerciseHistory`, computes weekly progression rate via existing `getProgressionRate`, computes current e1RM via existing `getCurrentE1RM`. Filters to `type === 'weight-training'` (or library entries with no `type` field — pre-v8 entries default to weight-training so legacy data doesn't disappear). Drops exercises with `<2` sessions (sparkline returns null below that threshold). Sorts by `rate` desc. Returns `{exercises: [{id, name, history (last windowSize), fullHistory, rate, rSquared, n, currentE1RM, totalSessions, latestDate, libraryEntry}, ...], totalCount}`. Defensive against null / non-array / malformed inputs.

666. **`E1RMSparkline` exported from `Recommendation.jsx`** with new optional `windowSize` prop (default 6, matches pre-Batch-57 hardcoded `slice(-6)`). The Strength tile's bottom-sheet drill-down passes `windowSize: Math.max(2, totalSessions)` so all sessions render in the chart, not just the last 6. RecommendationSheet's existing call site doesn't pass the prop, so behavior in the recommender is unchanged.

667. **`StrengthTile` + `CompactSparkline` inline components in `Progress.jsx`.** CompactSparkline is minimal — 60×24 SVG path-only, no dots / trend line / labels, stroke at user's accent color (per the `.strength-row` mockup spec at [Gains-Design-Mockups.html:559–578](Gains-Design-Mockups.html#L559)). StrengthTile renders one `<button>` per top-N row (`flex-1 name [13px text-secondary fw-500] / 60×24 sparkline / 11px right-aligned trend pct`), `Show all (N) →` toggle when `totalCount > 4`, and an empty-state copy "Log a few sessions to see your strength trends here" when zero rows. Trend pct color: emerald `#10b981` when ≥ +0.5%/wk, amber `#f59e0b` when negative, muted text-secondary otherwise. Each row's `aria-label="Open {name} history"` opens `ExerciseHistorySheet` for that exercise.

668. **`src/components/ExerciseHistorySheet.jsx` (new)** — bottom-sheet portal at z-260 (peer with `ExerciseEditSheet`, sits above `RecommendationSheet`'s 250). Mirrors `ExerciseEditSheet`'s container: `rounded-t-3xl md:rounded-3xl`, `max-h-[90vh] overflow-y-auto`, backdrop tap dismiss, scroll containment via `overscrollBehavior: 'contain'`. Layout: header (eyebrow `STRENGTH HISTORY` + exercise name + close ×) → full E1RMSparkline at full-history depth with tap-to-select dots → inline label below the chart that shows Peak / selected-session / Weekly growth → 4 stat tiles → recent sessions list (newest first, 20 visible + Show all toggle, each row showing `weight × reps`, `formatRelativeDate(date)`, `e1RM`, 🏆 badge for all-time PR row, equipmentInstance + gymId appended in muted text when set). All hooks (useState/useMemo) called BEFORE the `if (!open || !exercise) return null` early return per the pre-flight checklist's hooks-after-early-return defense. Lessons learned from the post-B51 stability pass: hooks must run unconditionally on every render to satisfy React's hook-count invariant.

669. **`Progress.jsx` wiring** — DELETED the legacy `PRTimeline` component (~73 lines of dead code) + `PR_COLORS` constant + the section header / StatCard wrapper that called it. ADDED `useState` import; new `useStore(s => s.exerciseLibrary)` subscription; `[openExercise, setOpenExercise]` state at the page level; the `<StrengthTile>` render block (replacing PRTimeline's slot at the same position in the section order); the `<ExerciseHistorySheet>` render at the page bottom. Section order is now: Coaching → Weekly Training Load → Time in the Gym → Muscle Group Balance → **Strength** → Consistency.

670. **`b57-strength-tile-sanity.mjs` at repo root.** 38/38 pass. Covers: cold-start gate (0/1/2-session cases all behave correctly); sort order across 3 synthetic exercises with rates +7%/wk / flat / negative produces correct descending order; type filter (only weight-training surfaces — hyrox-station, hyrox-round, running all excluded; legacy untyped library entries default to weight-training so pre-v8 data still shows); history window (`history` is `slice(-6)`, `fullHistory` is the full array); drop-set bundling correctness (one entry per session, not three when nested drops exist); 7 defensive cases (null/non-array sessions, null/non-array library, undefined args, name-fallback for pre-v3 sessions, non-bb-mode session ignored, malformed nameless exercises skipped); custom windowSize override (3 → history.length === 3; 0 → falls back to default 6); real-data spot check from `workout-backup-2026-04-26.json` (30 sessions / 149 library entries → 34 qualifying rows, all sorted descending by rate, all with ≥2 sessions, all weight-training).

671. **Verified live in preview** (port 5173 dev server, mobile 375×812). Three states pass with zero new console errors:
    - **Empty data** (`sessions: []`, no splits beyond built-in) → Strength tile renders empty-state copy "Log a few sessions to see your strength trends here". Bottom sheet not openable. All other tiles (Weekly Load / Duration / Radar / Consistency) preserve their canonical empty-state messages.
    - **Populated weight-only** (BamBam Blueprint active, real backup data — 30 sessions / 149 library entries) → Strength tile renders top 4: Cable Curls (+37.9%/wk green) / Calf Raises (+19.1%/wk green) / Single Arm Lat Pulldown (+17.2%/wk green) / Leg Press (+14.5%/wk green). `Show all 34 →` button visible — tapping reveals all 34 rows, button text flips to `Show less ←`. Tap a row → `ExerciseHistorySheet` opens at z-260: header reads `STRENGTH HISTORY` + exercise name; full E1RMSparkline renders all sessions with peak emphasized; 4 stat tiles correctly show Current e1RM / Weekly growth / Sessions count / All-time PR (W×R format). Tap a sparkline dot → inline label flips to "Session: 245 lbs · 225×9 · 3 days ago" (selected) instead of the default Peak. Tap the same dot → reverts to Peak. Recent sessions list renders newest first with PR badges on all-time max rows. Close × dismisses, backdrop tap dismisses.
    - **HYROX-active** (HYROX Hybrid week-mode split active, 4 Tuesday sessions with HYROX SkiErg rounds + lift sessions) → Strength tile shows ONLY weight-training entries — no SkiErg, Run, or HYROX-round rows surface. The HYROX section preview on `/log/bb/hyx_tuesday` continues to render correctly (B41 surface unaffected).
    - Daylight + light accent test: emerald/amber colors hold contrast on the lighter background; sparkline stroke uses theme.hex correctly.

672. **Build.** `npx vite build --outDir /tmp/test-build` → 925.21 KB bundle / 249.51 KB gzipped (+7.23 KB / +1.67 KB vs Batch 56, accounted for by the new `buildStrengthTileData` aggregator + `ExerciseHistorySheet` component (~250 lines) + the `StrengthTile` + `CompactSparkline` inline components, offset partly by the deleted `PRTimeline`).

### Batch 58 (April 28, 2026) — Progress redesign closure: time-range picker + 3-tile reframing + Achievements

Third and final Progress page redesign batch per `Gains-Design-Critique.md` §2 lines 116–139. Closes out the "Progress v2 — Coaching Surface, Not Dashboard" vision: time-range picker chips at the top of `/progress`, 5 visible cards collapsing into 3 (Volume / Strength / Consistency), and a new Achievements section at the bottom that re-homes stats Dashboard v2 (B51) dropped. User-locked decisions: tap-tile-opens-sheet for Volume drill (mirrors B57); Time-in-the-Gym (DurationChart) dropped entirely; picker `1mo/3mo/6mo/All` defaults to `3mo` and filters Volume + Strength; Consistency stays at fixed 13-week heatmap; Achievements ships in this batch.

673. **`buildVolumeTileData({sessions, windowStartTs, prevWindowStartTs, now})` in `helpers.js`** — pure aggregator. Walks bb-mode sessions once, returns `{totalVolume, prevTotalVolume, deltaPct, weeklySeries, byWorkoutType, sessionCount}`. Drops nested in primary working sets contribute to volume per Batch 22 decision 2. Weekly series buckets by Sunday-anchored week-start (local timezone). By-workout-type breakdown sorts descending by volume. When `windowStartTs` is null (All), `prevTotalVolume = 0` and `deltaPct = null`. Defensive against null / non-array / malformed inputs at every layer.

674. **`buildAchievementsData({sessions, cardioSessions, restDaySessions, splits, activeSplitId})` in `helpers.js`** — pure aggregator. Returns `{prsThisSplit, totalSessions, bestStreak, currentStreak, badges}`. Composes `getWorkoutStreak` + `getBestStreak` + `getAchievements`. PR scoping: when `activeSplitId` matches a split in the splits array, only PRs in sessions whose `type` matches one of that split's workout ids are counted. Null `activeSplitId` falls through to all-time PR count. Defensive against null inputs.

675. **`RadarChart` accepts `cutoffDate` prop** (`Progress.jsx:378`). When passed a Date object, uses that as the cutoff for filtering sessions; falls back to legacy 30-day default when `null`. Unblocks the Volume drill sheet's picker-scoped per-muscle-group radar.

676. **`src/components/VolumeDrillSheet.jsx` (new)** — bottom-sheet portal at z-260 mirroring B57's `ExerciseHistorySheet` structural pattern. Renders three slots Progress.jsx supplies as React nodes (rather than importing WeeklyLoadChart / RadarChart directly): `weeklyLoadNode` (3-week stacked-bar comparison anchored to now, NOT picker-scoped), `radarNode` (per-muscle-group radar over picker's window), and a per-workout-type list at the bottom (rows showing type name + volume + session count + % of total). All hooks called BEFORE `if (!open) return null` early return per the pre-flight checklist.

677. **Picker chip row in `Progress.jsx`** — `RangePicker` inline component with 4 chips (`1mo / 3mo / 6mo / All`), default `3mo`. Selected chip uses `theme.hex` + `theme.contrastText`; unselected uses neutral `bg-item`. State (`range`) lives in the `Progress` component; `windowStartTs` + `prevWindowStartTs` derived via `rangeStartTs` / `prevRangeStartTs` helpers. `filteredSessions = sessions.filter(s => t >= windowStartTs)` threaded down to Strength tile. Volume tile uses `buildVolumeTileData` directly (with both window timestamps for prior-period delta).

678. **`VolumeTile` inline component in `Progress.jsx`** — picker-aware aggregate view. Layout: large 28px `47.5k` total volume, `lb · N sessions` subtitle, delta pill (`+12% vs prior 3 months`, green ≥0 / amber when negative / muted when no prior data), 80×24 weekly-volume sparkline at right, `Tap to see breakdown →` hint. Whole tile is a `<button>` opening `VolumeDrillSheet`. Empty state when sessionCount===0: "Log a few sessions to see volume trends".

679. **`AchievementsCard` inline component in `Progress.jsx`** — bottom of page. 3-column stat tile row (`PRs this split` / `Total sessions` / `Best streak`) using `buildAchievementsData` output. Below: badge grid (auto-fill, 100px min-width) rendering `getAchievements()` badges as icon + label + sub. Empty state when totalSessions===0 + no badges: "Log your first session to start earning achievements." Renders "Keep training to unlock your first badge." sub-state when sessions exist but no badges yet.

680. **DurationChart deleted entirely** (`Progress.jsx:292-374`). Time-in-gym is rarely actionable; per Critique §2 the 5→3 reframing implicitly drops it. Replaced with a one-line comment marker.

681. **Page section order (final)**: sticky `<h1>Progress</h1>` + RangePicker → Coaching → Volume → Strength (filteredSessions) → Consistency (unfiltered, 13-week) → Achievements (unfiltered, all-time). Coaching + Consistency + Achievements ignore the picker on purpose — they own their own windowing semantics (30-day rolling, 13-week heatmap, all-time stats respectively). The picker doesn't disappear but doesn't affect them — visual signal: only Volume + Strength change when the user taps a chip.

682. **`b58-progress-redesign-sanity.mjs` at repo root.** 42/42 pass. Covers: cold-start cases (empty/null/undefined/non-array sessions); aggregation math (1mo + All windows; volume + session count + weekly series + prev-period delta = +198% over basic synthetic data); drops contribute to total (Batch 22 decision 2 verified); byWorkoutType breakdown (sorted desc, correct counts); weekly series sorted asc and sums to total; defensive cases (malformed sessions / bad dates / null sets); achievements basic shape (totalSessions / prsThisSplit / bestStreak / badges array including first-session and first-PR badge IDs); PR scoping by active split (PRs in non-active workout types correctly filtered out; null activeSplitId counts all); achievements defensive cases; real-data spot check from `workout-backup-2026-04-26.json` (30 bb sessions, 3mo all-30 since user trains daily — 641,320 lb total, 6 workout types correctly sorted desc, 140 PRs this split, 30 total sessions, 12-day best streak, 7 badges).

683. **Verified live in preview** (port 5185 worktree dev server, mobile 375×812).
    - **Empty data** (`sessions: []`) → page renders without crash. Picker chips visible. Volume tile shows "Log a few sessions" empty state. Strength tile shows its own empty state. Consistency heatmap renders with 0 active days. Achievements shows "Log your first session to start earning achievements." No console errors.
    - **Populated weight-only** (BamBam Blueprint, 30-session backup) → picker default `3mo` selected. Volume tile shows `641k lb · 30 sessions` + No prior data message + weekly sparkline. Tap Volume tile → drill sheet opens at z-260 with header `Volume breakdown / Last 3 months`, 3-week stacked-bar chart, picker-scoped Radar chart, per-workout-type list (legs1 149k lb · 6 sessions · 23% of total → pull → push → push2 → legs2 → custom). Strength tile (B57) renders top 4 over 3mo window — list size matches B57 behavior. Tapping picker chips: `1mo` → Volume shrinks; `All` → Volume balloons; Strength rate computations re-run correctly per window. Consistency heatmap unchanged when picker changes. Achievements section shows 140 PRs / 30 total / 12 best streak + 7 badges in grid.
    - **HYROX-active** (HYROX Hybrid week-mode split active, mixed lift + HYROX sessions) → Volume tile aggregates lift volume only (HYROX rounds don't carry weight×reps in flat-set form). Strength tile only shows weight-training entries (B57 invariant unchanged). Achievements section filters PRs to HYROX Hybrid workout ids correctly. HYROX section preview on `/log/bb/hyx_tuesday` continues to render correctly (B41 surface unaffected).
    - Daylight + white accent test: picker chip contrast holds (white-bg with dark text on selected); delta pill emerald/amber colors readable on white card; weekly sparkline stroke uses theme.hex (same caveat as B57 — white accent stroke is faint on white card; documented).

684. **Files modified.**
    - `src/utils/helpers.js` — `buildVolumeTileData` aggregator + `buildAchievementsData` aggregator.
    - `src/pages/Progress.jsx` — picker chip row state; thread `filteredSessions` to Strength; delete DurationChart; add `RangePicker` + `VolumeSparkline` + `VolumeTile` + `AchievementsCard` inline components; adapt `RadarChart` to accept `cutoffDate` prop; rewrite main return with new section order.
    - `src/components/VolumeDrillSheet.jsx` (new) — bottom-sheet drill-down (~150 lines).
    - `b58-progress-redesign-sanity.mjs` (new, repo root) — 42-assertion sanity.
    - `CLAUDE.md` — last-updated header + this Batch 58 entry.

685. **Build.** `npx vite build --outDir /tmp/test-build` → 936.09 KB bundle / 251.83 KB gzipped (+10.88 KB / +2.32 KB vs Batch 57, accounted for by `buildVolumeTileData` + `buildAchievementsData` aggregators (~3 KB), `VolumeDrillSheet` component (~3 KB), the inline `RangePicker` + `VolumeSparkline` + `VolumeTile` + `AchievementsCard` components in Progress.jsx (~5 KB).

### Batch 58 v2 (April 28, 2026) — Tabbed layout + Coach card fixes + Consistency polish

Live-feedback iteration on B58 v1. User tested the populated state and reported four issues: (1) "92 PRs on a single-arm lat pulldown is bonkers" — the Coach card headline was rendering the GLOBAL pr-count next to a single exercise's name (e.g. `97 PRs on Pec Dec` when really 97 was across 30+ exercises and Pec Dec had 5); (2) Coach card should scope to the picker AND be collapsible; (3) page should be tabbed (Volume / Strength / Consistency / Achievements) instead of one long vertical scroll; (4) Consistency: every day labeled (currently only M/W/F), clarify "13 weeks" caption, add monthly session counts. Iterated on the same `claude/b58-progress-redesign-closure` branch (B58 v1 was never merged to main per the user's preview-gate workflow).

686. **`buildMonthlyCoachingSummary` PR-headline bug fix** (`helpers.js`). Pre-fix line `${prCount} ${noun} on ${topPrEntry.name}` rendered the cross-library total next to one exercise's name, producing strings like `97 PRs on Pec Dec` when Pec Dec only had 5. Fixed by scoping the count to `topPrCount` (top exercise's count specifically). Trigger threshold also tightened: was `prCount >= 3` (any total), now `topPrCount >= 3` (the named exercise must itself have ≥3 PRs). New cross-library bullet surfaces the broader context: `Plus 92 PRs across 41 other exercises.` — clear that the headline number belongs to the named exercise. New invariant captured in feedback memory `feedback_data_clarity_over_completeness.md`: never combine an aggregate total with a single entity's name in the same sentence; users catch bad stats fast and lose trust in the whole card.

687. **Volume bullet framing reworked** (`helpers.js`). User: "Volume up 283% doesn't make sense — is that related to all of my training or what?" Math was correct but framing was alarming because the prior 30-day window was sparse. Headline-led volume case (`volumeDeltaPct >= 15`) now reads `12 sessions this month. Volume up 283% vs last month.` (lead with sessions, then % delta). Bullet-form preserves the % but uses windowed prior label: `Volume up 12% vs prior 3 months.` (was: `vs last 30 days`).

688. **`buildMonthlyCoachingSummary` accepts `windowDays` parameter** (`helpers.js`). Was hardcoded to 30 days; now configurable so the card recomputes when the picker changes. Picker maps: 1mo → 30, 3mo → 90, 6mo → 180, All → null. New `windowShort` / `priorShort` / `eyebrowSuffix` derivations produce picker-aware copy: eyebrow flips `LAST 30 DAYS` → `LAST 3 MONTHS` → `LAST 6 MONTHS` → `ALL TIME`; headline + bullet copy uses `this month` / `this 3 months` / `this 6 months` / `all-time`; volume comparisons use `vs last month` / `vs prior 3 months` / etc. All-time mode skips volume-led headline branches (no prior period to compare against). Meta payload extended: `windowDays`, `isAllTime`, `topPrCount`.

689. **MonthlyCoachingCard collapsible** (`Progress.jsx`). Per user request "be collapsible". State `coachCollapsed` lives in the Progress component. Collapsed renders just the headline truncated with a subtle chevron `▾`; expanded shows full eyebrow + headline + bullets + suggestion. Default expanded; user toggle persists for the visit only (not localStorage-persistent). The eyebrow + chevron live on a single button row so the entire top of the card is the toggle target.

690. **Tabbed Progress page** (`Progress.jsx`). Per user request "rather than everything being in one long horizontal view, you just see the thing you want to see." New `TabRow` inline component sits between the Coach card and tab content, with 4 tabs: `Volume / Strength / Consistency / Achievements`. Selected tab uses `theme.hex` 2px bottom border + bold primary-text color; unselected muted-text + transparent border. State (`activeTab`, default `'volume'`) lives in Progress. Coach card stays pinned above the tabs (it's the headline, applies to all tabs); time-range picker stays in the sticky header.

691. **ConsistencyHeatmap polish** (`Progress.jsx`).
    - **All 7 day labels** (`['S', 'M', 'T', 'W', 'T', 'F', 'S']`) per user "you should have every day of the week labeled". Was sparse `[null, 'M', null, 'W', null, 'F', null]`.
    - **Monthly session counts row** above the heatmap. Two stat tiles side-by-side: `This month: 19 sessions · April` / `Last month: 10 sessions · March +9` (delta in green/amber). Counts unique active calendar days from bb-mode sessions in the current vs prior calendar month. Answers user's "I would like to just quickly see how many sessions did I log last month? This month?" without making them count cells.
    - **"13 wks" caption clarified** to `active days in last 13 weeks` (was `(13 wks)` parenthetical, user asked "What does 13 weeks mean?").
    - **`Last 13 weeks` section eyebrow** above the heatmap explicitly labels its scope.

692. **`b58-progress-redesign-sanity.mjs` extended.** 61 assertions (was 42). New tests covering: PR-led headline scopes count to top exercise (no `97 PRs on Pec Dec` bug); cross-library bullet surfaces extras when applicable; `windowDays` parameterization (30/90/180/null) produces correct meta + eyebrow strings; volume bullets use windowed prior-period framing; `meta.topPrCount` distinct from `meta.prCount`. Real-data spot check (workout-backup-2026-04-26) verifies headline now reads `Strong push month. 5 PRs on Pec Dec.` (was `97 PRs on Pec Dec`) with bullet `Plus 92 PRs across 41 other exercises.`. Existing 77 b53 sanity assertions still pass under the new algorithm. Total: 138/138 across both sanity scripts.

693. **Verified live in preview** (port 5185 worktree dev server, mobile 375×812).
    - **Coach card** at 3mo (default): `LAST 3 MONTHS · Strong push month. 8 PRs on Pec Dec. · Plus 133 PRs across 49 other exercises. · Single Arm Lat Pulldown trending +17.2%/wk across 6 sessions. ⚠ Leg Curls has dipped 6 sessions. Consider a recovery week.` Picker → 1mo: eyebrow flips `LAST 30 DAYS`, headline `Strong pull month. 4 PRs on Single Arm Lat Pulldown.` + `Plus 88 PRs across 40 other exercises.` + `Volume up 226% vs last month.` (different top-exercise wins because window is smaller, all per-exercise counts are bounded). Collapse/expand toggle works: collapsed renders just the headline, expanded restores bullets + suggestion.
    - **Tabs** swap content correctly: Volume → 641k lb · 30 sessions tile + drill-sheet still works; Strength → top 4 by progression rate per B57; Consistency → 19 sessions · April + 10 sessions · March + +9 delta + all 7 day labels in heatmap (S/M/T/W/T/F/S) + `29 active days in last 13 weeks` caption; Achievements → 140 PRs / 30 total / 12 best streak + badge grid.
    - Tab persistence: resets to Volume on each visit (intentional — Progress is a destination).
    - Console clean of new errors. Pre-existing `<ExerciseItem>` key-spread warnings from BbLogger.jsx persist (predate B41, documented).

694. **Files modified.**
    - `src/utils/helpers.js` — `buildMonthlyCoachingSummary` algorithm fix + `windowDays` parameterization + `windowShort`/`priorShort`/`eyebrowSuffix` derivation + new meta fields.
    - `src/pages/Progress.jsx` — `TabRow` component + tab state; MonthlyCoachingCard collapsible + accepts `range`/`collapsed`/`onToggleCollapse` props; ConsistencyHeatmap dayLabels + monthly stats row + caption clarification; main return rewritten to tabbed layout.
    - `b58-progress-redesign-sanity.mjs` — +19 new assertions covering Coach v2 changes (61 total).
    - `CLAUDE.md` — last-updated header + this Batch 58 v2 entry.

695. **Build.** `npx vite build --outDir /tmp/test-build` → 940.38 KB bundle / 253.13 KB gzipped (+4.29 KB / +1.30 KB vs B58 v1, accounted for by `TabRow` component + collapsible coach state + `windowDays` derivation logic + extended monthly-stats row in ConsistencyHeatmap).

696. **Deferred from B58 v2** (still queued):
    - ~~Dashboard look-and-feel parity polish on Progress cards~~ → shipped as B58 v3 (#697 below).
    - Picker-resize for Consistency heatmap (locked to 13-week regardless of picker — user's earlier decision; could revisit if surfaced as friction).

### Batch 58 v3 (April 28, 2026) — Progress card aesthetic mirrors Dashboard hero

User feedback after v2 ship: "I want the card design to mirror the design and feel of the home page cards." Closes the look-and-feel gap deferred from v2.

697. **`StatCard` mirrors Dashboard's hero card style** (`Progress.jsx`). Pre-fix style: `bg-card / borderRadius:16 / padding:14/14/12` — flat neutral surface. New style copies Dashboard.jsx's `heroAccentCardStyle` (lines 1040–1049) verbatim:
    - `borderRadius: 24` (was 16)
    - `padding: '20px 18px 18px'` (was 14/14/12)
    - `background: linear-gradient(135deg, ${hexToRgba(heroSurface, 0.10)} 0%, ${hexToRgba(heroSurface, 0.02)} 70%), var(--bg-card)` — accent-tinted gradient wash on the card surface.
    - `border: 1px solid ${hexToRgba(heroSurface, 0.18)}` — subtle accent-tinted edge.
    - `boxShadow: 0 4px 30px ${hexToRgba(heroSurface, 0.10)}, inset 0 1px 0 rgba(255,255,255,0.04)` — soft accent glow + inset highlight.
    `position: 'relative'` + `overflow: 'hidden'` for visual containment. Now StatCard accepts `theme` + `settings` props; all 4 callsites updated (Volume / Strength / Consistency / Achievements).

698. **`hexToRgba(hex, alpha)` + `resolveHeroSurface(theme, settings)` helpers** (`Progress.jsx`). Mirror Dashboard's local helpers (`hexToRgba` at line 237, daylight + light-accent fallback at line 576). The fallback derivation: when `settings.backgroundTheme === 'daylight'` AND the user's accent is light (luminance > 160 via the standard 0.299/0.587/0.114 formula), the hero surface color flips from `theme.hex` to `#1A1A1A` so the gradient is visible against a white card. Pure-obsidian users + dark-accent users continue to see the actual accent color in the wash.

699. **Coach card and TabRow stay distinct.** The yellow Coach card keeps its HYROX-fixed brand gradient (per design doc §12.4 "Don't theme yellow with the user's accent — it's a fixed brand color"). The TabRow is a chrome navigation element (border-bottom underline) and doesn't take a card treatment. Only the Volume / Strength / Consistency / Achievements content cards get the new hero-style aesthetic.

700. **Verified live in preview** (port 5185 worktree dev server, mobile 375×812).
    - Obsidian + violet: Volume / Strength / Consistency tile each render with violet gradient wash (10%/2% on var(--bg-card)) + violet 18% border + violet 10% glow shadow + 24px rounded corners + 20/18/18 padding. Visual parity with Dashboard's hero workout card confirmed.
    - Daylight + white accent: gradient flips to dark-tinted (`rgba(26,26,26,0.10/0.02)`) so the card retains visible structure on white background. Border + shadow follow the same fallback. Verified via computed-style inspection: `border: 1px solid rgba(26,26,26,0.18)`, `box-shadow: rgba(26,26,26,0.1) 0px 4px 30px ...`.
    - Coach card unchanged (yellow brand wash); TabRow unchanged (border-bottom underline).
    - Console clean of new errors. Pre-existing BbLogger key-spread warnings persist (documented).

701. **Files modified.**
    - `src/pages/Progress.jsx` — added `hexToRgba` + `resolveHeroSurface` helpers; rewrote `StatCard` to accept `theme` + `settings` and apply the Dashboard hero gradient pattern; threaded both props into all 4 StatCard callsites.
    - `CLAUDE.md` — last-updated header + this v3 entry.

702. **Build.** `npx vite build --outDir /tmp/test-build` → 940.55 KB bundle / 253.21 KB gzipped (+0.17 KB / +0.08 KB vs v2 — pure helper additions; the gradient strings are runtime-computed style objects, not bundle-cost).

### Batch 58 v3.1 (April 28, 2026) — Drop-in fade animation across Progress, BbLogger, History

User feedback: "the dashboard screen has a drop-in effect where the screen components sort of fall into place. I would like to add that effect to the progress screen. Also can you apply the same drop-in pattern on the logging session and history pages?" Mirrors Dashboard.jsx's `dashFadeInV2` keyframe + fadeIn(delay) helper across three new pages.

703. **Progress staggered fade** (`Progress.jsx`). Three-stage cascade:
    - Coach card (delay 0ms)
    - TabRow (delay 60ms)
    - Active tab content (delay 120ms, keyed on `activeTab` so React unmounts + remounts on tab switch — fade re-runs each time the user taps a different tab, feels intentional)
    Keyframe `progFadeInV2` injected via inline `<style>` block at the top of the page return. `fadeIn(delay)` helper produces `{ animation: 'progFadeInV2 0.4s ease forwards', animationDelay: '${delay}ms', opacity: 0 }`. Verified mid-animation: opacity 0 → 1 + translateY 8px → 0 over 400ms ease.

704. **BbLogger single-stage fade** (`BbLogger.jsx`). The exercise groups + Add Exercise + Session Notes container all fade in together as one unit (delay 0ms). Sticky header doesn't animate — it's already visible chrome. Sticky Finish footer (createPortal target) doesn't animate either — persistent UI element. Keyframe `bbLoggerFadeInV2` injected via inline `<style>`. Stagger isn't needed here because there's only one main content section to fade in.

705. **History single-stage fade** (`History.jsx`). The session list container fades in (delay 0ms). Sticky header chrome stays static. Keyframe `historyFadeInV2` injected via inline `<style>`. Same shape as BbLogger — single content area gets the drop-in treatment.

706. **Why distinct keyframe names per page** (`progFadeInV2` / `bbLoggerFadeInV2` / `historyFadeInV2`). All three have identical from/to declarations but live in their own page's inline `<style>` block. Distinct names avoid scope coupling — if Dashboard ever changes its keyframe behavior, the other pages don't break.

707. **Verified live in preview** (port 5185 worktree, mobile 375×812):
    - **Progress**: 3 wrappers detected with staggered delays (0ms / 60ms / 120ms). Tab switch re-runs the animation (verified mid-flight: opacity:0 + delay:120ms after tap). Visual cascade matches Dashboard's "components fall into place" feel.
    - **BbLogger** (active session, BamBam Push): 1 wrapper detected with `animation: 0.4s ease 0s 1 normal forwards running bbLoggerFadeInV2; opacity: 0;` confirmed via inline-style inspection. Keyframe block injected into DOM.
    - **History**: 1 wrapper detected with `animation: 0.4s ease 0s 1 normal forwards running historyFadeInV2; opacity: 0;` confirmed.
    - All three pages verified end-to-end. No console errors introduced.

708. **Files modified.**
    - `src/pages/Progress.jsx` — `progFadeInV2` keyframe + `fadeIn(delay)` helper + 3 staggered wrappers (Coach card / TabRow / `<div key={activeTab}>` for tab content).
    - `src/pages/log/BbLogger.jsx` — `bbLoggerFadeInV2` keyframe + inline animation on the `.px-4 pt-3 space-y-2` content container.
    - `src/pages/History.jsx` — `historyFadeInV2` keyframe + inline animation on the `.px-4` session list container.
    - `CLAUDE.md` — last-updated header + this v3.1 entry.

709. **Build.** `npx vite build --outDir /tmp/test-build` → 940.95 KB bundle / 253.30 KB gzipped (+0.40 KB / +0.09 KB vs v3 — three keyframe blocks + inline animation styles).

### Batch 59 (April 28, 2026) — Active set hero + chip toolbar rationalize (Logger v2, 9 iterations)

First user-locked Logger v2 batch per `Gains-Design-Critique.md` §3 + `HANDOFF-LOGGER-V2.md`. Promotes the in-progress set row to a visually distinct hero (accent-tinted gradient + soft border + glow, mirroring the Dashboard hero card recipe) while completed sets above and below shrink to compact dimmer rows. Same primitives, less chrome, clearer hierarchy. Iterated 9 times across user feedback rounds (v1 → v9) before merge — the multi-pass tuning is the headline event; the final visual is a chip toolbar collapsed behind a +/− pill in the title row, a narrow gradient-wrapped active set, dim-but-readable compact siblings, and a centered liquid-glass Finish pill.

710. **`getActiveSetIndex(exercise, activeFieldKey)` in `BbLogger.jsx`** — pure module-level helper. Resolves which set is the "hero" via three tiers: (1) the set whose numpad fieldKey is currently active wins (`weight-${exName}-${i}`, `reps-${exName}-${i}`, `reps-plate-${exName}-${i}` candidates checked by string equality; drop-stage focus on `weight-drop-…-${i}-${j}` or `reps-drop-…-${i}-${j}` hoists the parent working set as hero); (2) first set whose weight or reps is empty; (3) last set in the list. String equality (not regex) is robust to exercise names with hyphens (Pull-up, Single Arm Lat Pulldown). Co-located helpers `hexToRgba` + `resolveHeroAccent` mirror the Dashboard / Progress card recipe with the daylight + light-accent fallback to `#1A1A1A` so the hero stays visible against a white card.

711. **`pendingActiveIdx` state on `ExerciseItem`** (added in v2). When `addSet(autoFocus=true)` fires after the user presses Next on a filled reps field, the new set must render as hero on the SAME commit so its input is in the DOM for the rAF focus call. Without this override, `getActiveSetIndex` keeps the prior set's focus signal pinned, the new set renders compact (no input mounted), `setWeightRefs.current[N]` is undefined, and the rAF focus no-ops silently. The override clears once focus lands and `activeFieldKey` updates naturally. State, not ref, so React re-renders the row as hero immediately.

712. **`SetRow` + `PlateSetRow` accept `phase: 'hero' | 'compact'` + `heroAccent` + `compactDropCount` props.** Hero phase wraps the existing weight/reps inputs in a gradient div (`linear-gradient(135deg, rgba(accent,0.10) 0% , rgba(accent,0.02) 70%)` + `1px` accent-tinted border + `0 4px 24px` accent glow shadow + `borderRadius: 14`). Compact phase renders the same Type chip + lbs input + reps input layout but at h-7 with full `bg-item` (dark) + `text-c-primary` for proper Obsidian contrast. End-cap is a tiny green ✓ when filled, × delete button when empty/incomplete. Tapping any input still opens the numpad → activeFieldKey updates → row promotes to hero on next render. Plate-loaded compact uses explicit column widths matching the column headers (Type w-14 / Total w-20 / Reps w-16) so values align under their labels; plate strip hidden in compact mode.

713. **`DropStageRow` gets `onAdvance` prop wired to `addDropStage(parentIdx)`** (v3 fix). Pre-fix bug: drop-stage reps numpad config had no `onNext` handler, so tapping Next did nothing. Now Next on drop reps adds a new drop stage to the same parent working set — mirroring the working-set Next behavior of `addSet`.

714. **Title row + chip toolbar collapse layout** (final state after iterations):
    - Title row: `[exercise name] [+/− pill] [chevron]` on a single horizontal line. Pill is to the immediate right of the name (left-anchored), chevron on the right edge.
    - +/− pill is SVG-rendered for pixel-perfect centering (`+` and `−` Unicode glyphs drift on the baseline differently per font). Always accent-illuminated (`theme.bgSubtle` + `theme.border` + `theme.text`) regardless of expanded state.
    - Title row padding conditional: `expanded` → `pt-4 pb-2 px-4` (asymmetric — keeps chip row close to title); `collapsed` → `py-4 px-4` (symmetric — title block centers vertically in the row).
    - Parent flex `items-center` so chevron centers vertically against the multi-line title block (name + summary line) on collapsed cards with logged sets.

715. **Chip toolbar — collapsed by default, expand via the +/− pill.** When expanded, chips render in a `flex-wrap` row directly below the title with a small `mt-1` gap. Order:
    1. **Rec** — moved out of the title row in v7 ("first visible pill because it can carry lots of importance"). Tap edits inline, same edit-on-tap behavior preserved at chip-row size.
    2. **Last** — toggles the ghost row showing prior session sets.
    3. **PR** — shows `{maxWeight}×{maxRepsAtMaxWeight}` in bold amber (the "PR" prefix dropped in v3 to fit; full text in hover-title).
    4. **Tip** — recommender chip, opens RecommendationSheet.
    5. **Machine** — equipment instance picker, full label visible (no truncation; wraps to next row if long).
    6. **Uni** — unilateral toggle (purple).
    7. **Plates** — fixed orange (`bg-orange-500/20 border-orange-500/40 text-orange-300`) when active, replacing the prior user-accent treatment that conflicted with the +/− pill accent. Tap opens the existing PlateConfigPopover.
    8. **SS** — superset chip, indigo. LAST in the row per user spec.

716. **`pencil + Mark as Done` action row sized to match `+ Add Set`** (v2). All three buttons at `py-2.5 rounded-xl` with matching height for a clean visual stack at the bottom of the expanded card. Pencil tap-toggle bug fixed via `data-notes-toggle="true"` attribute + `relatedTarget` check in the notes input's `onBlur` — was a blur+click race where the click handler's stale state reopened the editor that the blur had just closed.

717. **Floating Finish pill (v9 final state)** — centered at the bottom of the viewport with a glassmorphism panel: `rgba(255,255,255,0.08)` translucent surface + `backdrop-blur-md`, accent-tinted border at 35% opacity, accent-colored text (`theme.hex`), and a soft accent glow shadow at 18% alpha. Reads as a frosted glass layer lifted off the page rather than a solid CTA. Pre-log state shows a subtle "Log a set to save" muted chip instead of the full pill. Wrapper uses `pointer-events-none` so the page beneath stays tappable except on the button itself.

718. **HYROX section preview + B55 focus-mode-hides-HYROX preserved.** When the numpad is open during a hybrid workout (e.g. HYROX Hybrid Tuesday with both Lift section and HYROX section), the immersive yellow HYROX preview card returns null per the Batch 55 item 1 gate. B59's hero refactor doesn't touch this code path — the HYROX preview's render branch in `BbLogger.jsx` is independent of the SetRow render loop.

719. **All critical primitives preserved** per HANDOFF-LOGGER-V2.md §"Critical primitives to preserve":
    - Active session persistence (`saveActiveSession` round-trip is unchanged; phase prop is render-time-only, never serialized).
    - Drop-set bundling (B22-24) — `set.drops[]` walked correctly by every aggregate; B59 only changes how drops *render* (visible vs hidden via parent phase).
    - Locked-into-Work cycler (B28) — `SetTypeBtn`'s `lockedToWorking` prop unchanged; chip lives inside SetRow, doesn't move.
    - Convert-to-Drop cycler (B32) — preserved verbatim on hero; on compact rows the cycler is replaced by a non-interactive Work/Warm label, requiring promote-then-convert.
    - Cross-card field refs for superset cycle (B36) — `exerciseFieldRefs` and `registerFieldRef` threading unchanged.
    - Numpad focus + auto-classify reverted (B21) — no silent reclassification on weight entry.
    - Coach's Call Use-it button (B28) — `applyRecommendation` populates the first empty working set + focuses reps; the newly-populated set becomes hero (empty reps qualifies under "first unfilled" branch).
    - `lastExDataByName` cross-workout-type fallback (B55 item 5) — three-pass build upstream of B59's render path.

720. **Iteration history.**
    - **v1**: Active set hero + compact rows shipped (`ca5b345`).
    - **v2**: Header style tweaks, button heights matched, pencil tap-toggle bug fixed, Next-after-pounds regression fixed via pendingActiveIdx (`4123d8a`).
    - **v3**: Chip toggle repositioned to title-row right column, Plates orange, all 7 chips fit in one row, drop next added (`63f248a`).
    - **v4**: SVG +/− glyph, title aligned with chevron, content shifted up, dark contrast restored on compact inputs, green ✓ replaces × on completed compact rows, plate compact column alignment (`9fcd511`).
    - **v5**: +/− pill moved to right of chevron in stacked column (`c4fc0b7`).
    - **v6**: Chip row spacing + flex-wrap (`1b76581`).
    - **v7**: Rec moved into chip row at position 1, +/− pill moved to LEFT of chevron (inline with name), machine no longer truncates (`84a5720`).
    - **v8**: Uni + SS moved to end with Plates (SS last), symmetric collapsed padding (`4b4a806`).
    - **v9**: Finish pill centered + liquid glass + accent text (`942e537`).

721. **Files modified.**
    - `src/pages/log/BbLogger.jsx` — every iteration touched this file. Final diff vs main predecessor (B58 v3.1): +719 / −359 lines (~360 net additions across 9 commits).
    - `src/pages/log/Recommendation.jsx` — Tip chip className shrunk to match the new compact chip dimensions.

722. **Build.** Final `npx vite build --outDir /tmp/test-build` → ~942 KB bundle / ~254 KB gzipped (+~1 KB / +~0.5 KB vs B58 v3.1, accounting for the new helpers + phase-aware rendering branches across 9 iterations of refinement).

### Batch 61 (May 6, 2026) — Machine chip persistence fix (A1 + A2 + A3)

User report: "the app does not remember the machine value that I input into the machine chip from session to session or gym to gym." Diagnosis from a backup walk: 11 of 13 ever-typed `equipmentInstance` values never made it into the library's `defaultMachineByGym` map. Two failure classes: pre-Batch-50 typings (April 21–25, before the auto-write code existed at all) AND a structural gap where the chip silently inherits values from session-history fallback (Layer C) without firing the popover's onChange — so Batch 50's dual-write never triggers and the per-gym map stays empty. User-locked decision: keep the chip + fix the persistence (Option A from `wise-forging-breeze.md`).

723. **`migrateLibraryToV11(library, sessions)` in `helpers.js`** (A2). Pure helper. Walks every bb-mode session newest-first; for each `(exerciseId-or-name, gymId)` pair with a non-empty `equipmentInstance`, writes the FIRST occurrence (= most-recent given newest-first sort) into `library[i].defaultMachineByGym[gymId]` if that slot is empty. Existing user-set values are preserved (the user may have set them explicitly via ExerciseEditSheet → Gyms — never overwrite). Sessions without `gymId` are skipped (no slot to write into). Idempotent: returns same library array reference when nothing changed. Defensive against null / non-array / malformed inputs at every layer.

724. **Persist version bumped 10 → 11 in `useStore.js`** (A2 wiring). New `if (version < 11)` block in the migrate chain runs `migrateLibraryToV11(persistedState.exerciseLibrary, persistedState.sessions || [])`. Must run AFTER v9 (sessions normalized) and AFTER v6/v7/v8 (library entries have stable ids + dimensions + the 8 HYROX stations seeded). `importData` also chains v11 after the v8 library migration so pre-v11 backups land in the current schema.

725. **A1 inheritance promotion useEffect in `BbLogger.jsx`.** New mount-only useEffect right after `[exercises] = useState(...)` that walks each seeded exercise; when `equipmentInstance` is set AND `seedGymId` is set AND the library entry has no `defaultMachineByGym[seedGymId]` yet, calls `setDefaultMachineByGymStore(libraryEntry.id, seedGymId, inst)`. Ref guard (`didPromoteInheritance`) prevents re-runs. Mid-session changes are still handled by the popover's onChange (Batch 50 dual-write) — A1 only catches the silent-inheritance case where onChange never fires. Closes the structural gap going forward.

726. **A3 gym-prefer in cross-workout-type pass 3 (`BbLogger.jsx`).** Pre-Batch-61 the cross-type fallback at line 3409-3416 ignored gym entirely, so switching active split (different workout-type IDs cause pass 1+2 to miss) could pre-fill the chip with a value from the wrong gym. Now split into 3a (same gym, any type) + 3b (any gym, any type — legacy fallback). Within-pass-3 same-gym preference closes the active-split-change destabilization the user suspected.

727. **`machine-persistence-sanity.mjs` at worktree root.** 29/29 pass. Covers: defensive cases (null/undefined/empty inputs); basic backfill; most-recent-wins per (exercise, gym) pair; existing values preserved; partial fill (new gym alongside existing); name fallback for pre-v3 sessions; sessions without gymId skipped; empty/whitespace equipmentInstance ignored; non-bb sessions skipped; idempotency (re-run = same reference); complex multi-exercise multi-gym; real-data spot check against `workout-backup-2026-05-06.json` (12 distinct (exercise, gym) pairs in sessions → 11 library entries gain `defaultMachineByGym`, jumping from 2 pre-migration to 11 post-migration; specific assertions for Pec Dec @ TR + VASA = "Life Fitness", Leg Press @ Lanhammer = "Freemotion" preserved); full v6→v7→v8→v9→v11 chain matches importData order.

728. **Live preview verification** (port 5173, mobile 375×812, real backup data injected at v10 to force the v10→v11 migrate hook). Three states pass:
    - **Empty data** → app boots clean, zero new console errors. No regressions.
    - **Populated weight-only** → pushed to `/log/bb/push` at TR with active split BamBam Blueprint. Pec Dec card expanded → +/− chip toggle revealed → Machine chip reads `Life Fitness` with title `Machine: Life Fitness` — pre-filled from the v11 library map, zero taps required. Switched defaultGymId to VASA, restarted session, expanded Pec Dec → chip reads `Life Fitness` (correct — same machine name at both gyms in the user's data). Single Arm Lat Pulldown's library entry verified to have ONLY `gym_TR: "Freemotion"` in its map (gym-axis specificity correct — empty at VASA).
    - **HYROX-active** → switched to Metamorphosis split, opened `/log/bb/hyx_tuesday` (Quads). Page renders cleanly: 5 LIFTS (Leg Extensions, Hammer Strength Leg Extension, Squats, Leg Press, Standing Calf Raise) above the immersive yellow HYROX section preview (HYROX Simulation Round, INTERVALS · 4 ROUNDS, RUN LEG 1000m, STATION Rotates 8, REST 1:30, Start HYROX button). No regressions.
    - All console errors are pre-existing `<ExerciseItem>` key-spread warnings predating B41 (documented known caveat); no new errors introduced.

729. **Files modified.**
    - `src/utils/helpers.js` — added `migrateLibraryToV11(library, sessions)` after `migrateSplitsToV10`.
    - `src/store/useStore.js` — imported `migrateLibraryToV11`, bumped persist `version: 10 → 11`, added `if (version < 11)` migrate block, chained `migrateLibraryToV11(libV8, migratedSessions)` in `importData`.
    - `src/pages/log/BbLogger.jsx` — added A1 inheritance-promotion useEffect after `useState(exercises)`; rewrote pass 3 of `lastExDataByName` build to gym-prefer first then fall through to any-gym (A3).
    - `machine-persistence-sanity.mjs` (new, worktree root) — 29-assertion sanity script.

730. **Build.** `npx vite build --outDir /tmp/test-build` → 967.66 KB bundle / 259.06 KB gzipped (+~25 KB / +~5 KB vs B59 — most of that delta is Hybrid Training-related batches B60+ that landed in main between B59 and B61, NOT B61 itself; the B61 net delta is ~3-5 KB across the migration helper + A1 useEffect + A3 sub-pass).

---

## Active Workstreams

### Hybrid Training v1 — design locked, ready for implementation

A major new workstream is fully designed and ready to ship in batches B37–B46. It adds first-class support for HYROX-style and running/cardio work alongside the current weight-training model. The user (Braden) is building this for a real user named Brooke whose hybrid program (lifts + HYROX rounds + runs) doesn't fit the current `{weight, reps}` data model.

**Three artifacts in the repo root capture the entire design + plan:**

- `hybrid-training-design-v1.md` — the design doc, currently at v3. 17 sections covering: dimension model (5 dimensions: weight/reps/distance/time/intensity), 4 exercise types (`weight-training`/`running`/`hyrox-station`/`hyrox-round`), the 8-station HYROX catalog with locked dimensions, the round container (run leg + station leg, atomic in v1, generalizable to N legs later), the Start HYROX overlay, the gym-clock digital timer spec (§17), the 30 cycling headlines (§13), the comparison-viz tri-surface model (intra-leg/post-round/post-session, ALL station-anchored — same SkiErg leg compares across all sessions where SkiErg appeared regardless of round position), the rest-between-rounds timer (prescribed in round template, configurable per session, full-screen yellow countdown), the bridge-to-finish flow (HYROX summary → back to lift if needed → hybrid finish modal showing both Lift and HYROX wins), and the units model (lbs+miles input, kg+km derived storage, §11). Decisions are locked.

- `hybrid-training-implementation-plan.md` — the 10-batch plan. Mirrors the structure of `coaching-recommender-plan.md`. Phase 1 (B37–B40) is foundation: library schema, session schema, library UI, JSON v3 import. Phase 2 (B41–B45) is the user-facing HYROX flow. Phase 3 (B46) is the hybrid finish flow + share card variant. Each batch has scope, sanity script spec, preview verification, files modified, dependencies. After B46, hybrid v1 is feature-complete.

- `brooke-hybrid-split.json` — currently v2. B40 reissues this as v3 against the real schema. Day-of-week names are stripped from titles and inferred from rotation position.

**Critical primitives for the implementation to honor:**

1. **Station-anchored comparisons.** The station IS the comparison primitive, not the round position. SkiErg accumulates history independently across all sessions and round templates. `getStationHistory(sessions, stationId, dimensions)` is the canonical helper (defined in B43). Comparisons fall back to pace (s/100m) when dimensions don't match prior occurrences. Round-total comparisons are derived from station-anchored leg comparisons, not the other way around.

2. **HYROX visual takeover.** Yellow + black inside HYROX mode (the HYROX brand identity), user's app accent everywhere else. `--color-hyrox` = `#EAB308`. Don't theme yellow with the user's accent — it's a fixed brand color.

3. **Round = run leg + station leg, atomic.** v1 locks `legs[]` to length 2 with run→station ordering. The schema generalizes to N-leg rounds (§4.5) for when Brooke shifts to HYROX-only training in the future, but v1 doesn't expose that flexibility.

4. **The 8 stations are a closed catalog.** Pre-programmed, dimensions locked per station (see `data/hyroxStations.js` to be created in B37). Users cannot create a 9th station in v1.

5. **Two persist migrations.** v7 → v8 (type field + 8-station seed) lands in B37. v8 → v9 (kg/km derived storage + dimensioned set fields) lands in B38.

6. **Pounds + miles for input, kg + km stored alongside.** §11 of the design doc. HYROX stations specifically invert: metric input (500m SkiErg), miles derived. Conversions applied at save time.

When picking up this workstream in a new Claude Code session, read those three artifacts in order: design doc → implementation plan → JSON. Then start at B37.

---

## Where to Go From Here

### v1 shipped — nothing remaining in scope

All 8 steps of the AI Coaching Recommender v1 plan plus §3.8 auto-classify plus step 9 anomaly UI landed across Batches 14–20d. Step 7 (equipment instance) shipped in Batch 19. Step 8 (gym tagging, §3.5 + §9.7) shipped across Batches 20 + 20a + 20b + 20c + 20d — data layer, session gym pill, recommender gym-scoping + auto-tag prompt, picker filter, and Settings CRUD respectively. The coach now has every §2/§3/§4 input wired end-to-end: e1RM history, readiness, goal mode, progression rate, grade, cardio, rest, gap, equipment instance, and gym. Plateau / regression / swing detectors surface on the exercise card. Users can tag exercises per gym, manage the gym list from Settings, and the picker filter honors the tags — closing the loop the user described as "eliminating all the ones that aren't available there."

### Post-v1 roadmap (explicitly deferred per plan Part 4)

- **Back-off sets**: v1 prescribes the top working set only. Top-set labeling already primes the UI. Trivial engine work, interesting UI work.
- **RPE re-introduction**: engine plumbing is alive (16c→16d revert). User observation: "most sets go to failure" — re-enabling needs smart defaults (auto-infer RPE from reps vs target) or a clear opt-in.
- **Anomaly detector refinements**: the three v1 detectors use conservative thresholds. Possible tunings once we have real usage signal: (a) per-exercise threshold (compound vs isolation expect different variance), (b) grade-aware plateau (a flat trend with consistent A grades is different from a flat trend with consistent C grades), (c) tighter swing-detector integration with Batch 19's equipment instance — scoping already naturally suppresses cross-machine swings once enough data accumulates, but an explicit "just switched machines" reasoning pass could be layered on.
- **§8.x tracks** (visual goals, coaching commentary, coach marketplace, macro/nutrition, learned readiness model): out of v1 scope entirely.

### Recommended sequencing

**Step 8 (gym tagging) closes out v1.** Batch 20's data layer is the foundation. The UI substeps (20a → 20d) can ship in order — each is independently reviewable and none depends on later ones. 20a (session gym pill) is the smallest and surfaces the feature visibly first; 20b (auto-fill + prompt) is the richest; 20c (picker filter) is a small chip; 20d (Settings UI) closes the loop with gym lifecycle management.

---
> Source: [bradenlanham/workout-tracker](https://github.com/bradenlanham/workout-tracker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
