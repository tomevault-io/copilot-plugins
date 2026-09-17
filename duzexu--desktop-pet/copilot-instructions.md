## desktop-pet

> This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## Development Commands

```bash
# Development
npm run dev              # Launch Electron app in development mode

# Testing
npm test                 # Run all unit tests (Vitest)
npm run test:watch       # Run tests in watch mode
npm run test:e2e         # Run E2E tests (Playwright)
npm run test:e2e:ci      # Run E2E tests without Electron smoke test

# Building
npm run build            # Build renderer processes with Vite
npm run pack             # Package app without creating installer
npm run dist             # Create distributable installer
```

## Development Policy

- Add focused debug logging at key decision points when implementing or changing behavior, especially around IPC boundaries, asset import/transcoding, rule matching, runtime state transitions, animation playback, and error/fallback paths. Prefer the existing logger helpers and include enough structured context to diagnose issues without reproducing them blindly.
- This project is currently in active development. Do not spend effort preserving backward compatibility for existing local data, manifests, configs, or package formats unless explicitly requested. Prefer simple, correct data shapes and migrations-by-reset over compatibility layers.

## Architecture Overview

### Electron Process Model

This is a **three-process Electron application** with strict security boundaries:

1. **Main Process** (`src/main/`)
   - Entry point: `main.js` - initializes config, windows, IPC, tray
   - Window management: `windows.js` - pet window (320x320, transparent, frameless, always-on-top) and panel window (960x680, standard frame)
   - IPC hub: `ipc.js` - 31 registered channels handling config, assets, petpack, display, system settings
   - Services: `config-store.js`, `asset-store.js`, `petpack.js`

2. **Preload Scripts** (`src/preload/`)
   - Security bridge using `contextBridge` with context isolation
   - `pet-preload.js` exposes `window.desktopPet` API
   - `panel-preload.js` exposes `window.desktopPetPanel` API
   - No direct Node.js access from renderer processes

3. **Renderer Processes** (`src/renderer/`)
   - Pet renderer: sprite animation, event handling, rule engine integration
   - Panel renderer: modular control panel UI (see Panel Architecture below)
   - Built with Vite for production

### Panel Architecture (Modular)

The control panel (`src/renderer/panel/`) is modular. Current long-lived tabs are:

- `overview.js` - Overview dashboard, runtime status, recent events
- `assets.js` - Asset management and petpack import/export
- `animations.js` - Default animation, clips, keyframes, green screen settings
- `rules.js` - Trigger rules, inline actions, exit conditions/actions, conflict warnings
- `display.js` - Display settings
- `system.js` - System settings, language, logs

Legacy States/Actions tabs are no longer registered in the tab navigation. The current model is `assets -> animations -> triggerRules -> inline actions`.

**Main Entry**:
- `panel.js` - Initializes DOM/API references, loads config, routes events, coordinates rendering, polls runtime state

**State Management**:
- `state.js` - Global panel state container
- `panel-state.js` - Config operations, form model helpers, ID generation, keyframe normalization

**UI Layer**:
- `ui/banner.js` - Banner message management
- `ui/header.js` - Common header component
- `ui/tabs.js` - Current tab navigation
- `ui/utils.js` - HTML utilities and form helpers

**Reusable Components**:
- `inline-action-editor.js` - Scoped action editor for rule actions and exit actions
- `keyframe-editor.js` - Keyframe input/output mapping editor
- `rule-condition-editor.js` - Rule condition editor
- `rule-action-selector.js` - Action selector
- `condition-browser.js` - Condition type browser

**Event Handlers**:
- `form-handlers.js` - Form submission and form-to-config parsing
- `asset-handlers.js` - Asset and petpack operations
- `event-handlers.js` - Click, change, input, drag/drop sorting, modal interactions

### Petpack System

**Petpack** is the `.petpack` file format (ZIP) containing:
- `manifest.json` - package metadata, animations, trigger rules
- Asset files (GIF, WebP, WebM, MP4, MOV, PNG, SVG)

**Import flow** (`src/main/services/petpack.js`):
1. Validation: check ZIP structure, paths, and supported file types
2. Security: reject symlinks and unsafe paths; stream extraction with dynamic memory/disk resource guards (no fixed per-file, per-package, or entry-count limits)
3. Manifest validation against schema (`src/shared/manifest-validator.js`)
4. Atomic installation: extract to temp → backup existing → atomic rename → rollback on failure

**Export flow**:
1. Collect package files recursively
2. Merge manifest with user config (animations, trigger rules, interactions)
3. Validate merged manifest
4. Create ZIP with merged manifest, atomic write to target

**Storage structure**: `{userDataDir}/packages/{packageId}/manifest.json` and `assets/`

### Rule Runtime

**Core files**:
- `src/shared/rule-engine.js` - pure rule matching helpers
- `src/renderer/pet/pet-runtime.js` - runtime model building, event history, cooldown state
- `src/renderer/pet/user-trigger-manager.js` - action execution only

**Runtime pipeline**:
1. `pet.js` builds an event context from DOM, timer, lifecycle, or global mouse input.
2. `ruleRuntime.evaluateEvent(context)` appends to a circular event history (last 40 events), matches rules, applies priority/cooldown/continuous semantics, and returns actions.
3. `UserTriggerManager.executeAction(action, context)` executes returned actions against `AnimationController`, media rendering, display state, messages, and window helpers.
4. User rules are **not** registered through `EventBus` in the pet runtime. `UserTriggerManager.loadTriggers()` exists for legacy/tests, but runtime execution is ruleRuntime matching + UserTriggerManager action execution.

**Rule matching model**:
- Rules use `conditions`; legacy `trigger + filters` is normalized when encountered.
- Required conditions (default) must all match within `conditionWindowMs` (default 3000ms).
- Optional conditions (`required: false`) are treated as an optional group; if present, at least one optional condition must match.
- Rules are priority sorted; `stopOnMatch !== false` stops after the first executed matching rule.

**Rule semantics**:
- Per-rule cooldown tracking prevents spam.
- `continuous: true` bypasses cooldown for continuous updates such as keyframe scrubbing.
- Priority-sorted matching returns actions from the highest-priority matching rule by default.
- `actionStrategy: "random"` picks one action; otherwise actions execute in sequence.
- Stateful mouse rules use `sustainMs`, `state.exitConditions`, and `state.exitActions`.
- `trigger + filters` legacy rule shape is normalized to `conditions`.

**Condition types**:
- Pointer: click, doubleClick, rightClick, mouseEnter, mouseLeave, mouseMove, mouseStill
- Drag: dragStart, dragging, dragEnd (with position/delta/distance)
- Duration: hoverDuration, idleDuration
- Timer: timer, randomTimer
- Lifecycle/system: appLaunch, packageLoaded, pomodoroComplete

**Filter operators**: `=`, `!=`, `>`, `>=`, `<`, `<=`, `between`, `in`, `notIn`

**Action execution** (`src/renderer/pet/user-trigger-manager.js`):
- `playAnimation` - request an animation clip; does **not** interrupt a protected clip (see Animation queue below). keyframe clips can be scrubbed by event progress
- `setKeyframeProgress` - set a keyframe clip to a mapped progress value (immediate; bypasses the queue)
- `showMessage` / `randomMessage` - display message bubble
- `changeScale` / `changeOpacity` - update display properties
- `movePet` - move the pet window
- `pomodoroTimer` - start/control Pomodoro runtime
- `hidePet` / `showPet` - toggle visibility
- `resetPosition` - move to configured position
- `openPanel` - show control panel

**Animation queue** (`src/renderer/pet/animation-controller.js`):
- Clip types: `default` (always loops, fallback), `oneshot` (plays once, then advances), `loop` (loops until switched), `keyframe` (paused media scrubbed by an explicit progress value).
- **Non-interrupting, single-slot, latest-wins.** `playAnimation` never cuts off a *protected* active clip. A clip is protected when it is a `oneshot` still within its `durationMs` timer, or any `loop`. A protected clip runs to completion and only then does the controller advance.
- **Pending slot.** While the active clip is protected, a new `playAnimation` request is stored in a single `pending` slot. If another request arrives before the active clip finishes, it **replaces** the pending one (the in-between request is dropped) — `animation:pending:replaced`. A request equal to the current active clip is ignored — `animation:pending:ignored-duplicate`.
- **Advance.** When the active clip ends, `advanceToPending()` plays the pending clip, or returns to default if the slot is empty. oneshot ends via its `durationMs` timer; loop advances at the next cycle boundary (`durationMs` = single-cycle length; no `durationMs` → advance immediately).
- **stopAnimation / return-to-default** also go through the queue: a protected clip finishes first, then returns to default (or plays a pending clip). The controller owns return-to-default; there is no external `scheduleIdle`.
- **keyframe scrub is the only immediate exception.** `setKeyframeProgress` applies immediately and clears the pending slot, because it is continuous mouse-driven scrubbing, not a discrete playback. `playAnimation` of a `keyframe`-type clip is *not* protected, so it also plays immediately.
- **interrupt clips bypass protection.** A clip with `interrupt: true` cuts through the queue: `playAnimation` clears any timers and the pending slot, then plays it immediately regardless of what is protected. An interrupt clip that becomes active still protects itself once playing (`_isProtected()` is unchanged).
- **clip-bound movement.** A oneshot clip may carry `movement: { direction, speed, easing }`. When such a clip becomes active, `pet.js` starts `createAnimationMovementRunner()`, which owns the delayed start and `requestAnimationFrame` loop and cancels both when the active clip changes. Movement runs over `durationMs - startDelayMs - endDelayMs` (skipped if no finite positive movement duration). `movement.easing` supports `preset` (`linear`, `easeIn`, `easeOut`, `easeInOut`), `strength`, `easeInMs`, `easeOutMs`, `startDelayMs`, and `endDelayMs`; the easing curve preserves distance within the active movement window. Movement is separate from the standalone `movePet` action.
- **durationMs semantics.** For `oneshot` it is total play time before advancing. For `loop` it is the single-cycle length used for boundary-aligned switching (a loop does not self-terminate on `durationMs`; it ends via stopAnimation, being overwritten by the queue, or a stateful rule exit).
- **config hot-reload** (`updateConfig`): if the active or pending clip was deleted, the slot is cleared and playback safely returns to default; if the active clip still exists, it is re-pointed at the rebuilt clip so edited fields (e.g. `durationMs`) take effect and any pending loop boundary is rescheduled.

**Keyframe animation**:
- Clip type `keyframe` supports image/GIF/video assets; GIF/video can be scrubbed by progress.
- Actions can use `progressFrom: "angleToPetProgress"` plus `scale` and `offset`.
- Runtime maps incoming progress through per-clip `keyframes` (`input -> output`) before rendering.
- Keyframe progress is driven only by global mouse events (`eventSource: "globalMouse"`) to avoid local DOM mousemove conflicts when the pet window starts intercepting pointer events.
- Global mouse angle mapping is top = `0`, right = `0.25`, bottom = `0.5`, left = `0.75`.

### Configuration and State

**Config store** (`src/main/services/config-store.js`):
- Location: `{userDataDir}/config.json`
- Deep merge with defaults on load
- Sections: `currentPackageId`, `display`, `system`, `animations`, `interactions`, `triggerRules`

**State flow**:
1. Panel modifies config → `config:save` IPC
2. Main process persists to disk → broadcasts `pet:runtime-updated`
3. Pet renderer rebuilds runtime model → re-evaluates rules
4. New behavior takes effect immediately (no restart required)

**Defaults** (`src/shared/defaults.js`):
- Pre-configured actions for common interactions (click, drag, hover, random, timed, sleep)
- Default trigger rules and messages
- User config overrides defaults via deep merge

### Security Mechanisms

**Path safety** (`src/shared/path-safety.js`):
- All filesystem paths validated before operations
- Rejects: absolute paths, backslashes, URL schemes, `..` traversal, `.` segments
- Enforces POSIX normalization

**Asset validation** (`src/main/services/asset-store.js`):
- Extension whitelist enforced
- Symlink rejection prevents directory traversal
- Assets stored only in package-specific directories
- Import requires prior approval from file picker dialog (sender ID tracking)

**Manifest validation** (`src/shared/manifest-validator.js`):
- Schema version enforcement (currently v1)
- Required fields: `schemaVersion`, `packageId`, `name`, `version`, `animations.default`, `animations.clips`
- Asset reference checking against available files
- Action-to-animation cross-reference validation prevents dangling animation references

**Context isolation**:
- Preload scripts use `contextBridge.exposeInMainWorld()`
- No `nodeIntegration` in renderer processes
- Controlled API surface via preload bridges
- No direct filesystem or Node.js API access from renderers

### Internationalization (i18n)

**Core file**: `src/shared/i18n.js`

- Dictionary-based translation system
- Supported locales: `en` (English), `zh` (Chinese)
- Fallback chain: requested locale → `en` → key itself
- Simple parameter interpolation: `{paramName}`
- Auto-detection from browser language or localStorage
- Usage: `t("namespace.section.key", {param: value})`

**Translation key organization**:
- `common.*` - shared UI elements (save, delete, cancel, etc.)
- `panel.*` - control panel sections and tabs
- `assets.*` - asset management UI
- `rules.*` - rule editor (condition types, field names, form labels)
- `system.*` - system settings (language, launch at login)

## Key Data Flows

### Package Load
```
User selects .petpack
→ petpack:import IPC
→ Main: Extract, validate, install to packages/{id}/
→ Main: Update config.currentPackageId, emit pet:runtime-updated
→ Pet renderer: Load manifest, merge with config, rebuild runtime
→ Pet renderer: Apply new sprites, rules, schedule timers
```

### Trigger Rule Execution
```
User/global/timer event
→ pet.js builds event context (position, timestamp, eventSource, metadata)
→ ruleRuntime.evaluateEvent(context) matches rules against runtime event history
→ ruleRuntime applies cooldown/continuous/priority and returns actions
→ pet.js sequences returned actions with durationMs delays
→ UserTriggerManager.executeAction(action, context) performs side effects
→ AnimationController (queues playAnimation, latest-wins) / MediaRenderer / message bubble / display settings update
```

### Config Update from Panel
```
Panel form submission
→ config:save IPC
→ Main: Merge with defaults, persist to config.json
→ Main: Reload active package, emit pet:runtime-updated
→ Pet renderer: Rebuild runtime model, clear old timers
→ Pet renderer: Schedule new timers based on updated rules
```

## Important Patterns

**IPC Communication**:
- Request-response: `ipcRenderer.invoke(channel, payload)` / `ipcMain.handle(channel, handler)`
- Push updates: `window.webContents.send(channel, data)` / `ipcRenderer.on(channel, callback)`
- All handlers return `{ok: boolean, error?: string}` tuples

**Atomic Operations**:
- Config writes: serialize → write to temp → atomic rename
- Package installs: extract to temp → backup old → rename → delete backup
- Prevents corruption on failure

**Performance Optimizations**:
- Mouse move events throttled to 100ms (prevents excessive rule evaluations)
- Bounds scheduling coalesces rapid window position updates (one IPC per frame)

**ID Generation** (`panel-state.js`):
- Format: `{prefix}-{timestamp36}-{random36}`
- Used for user-created rules and panel-created records; animation clip IDs are UUIDs

## Extension Points

**Adding new action types**:
1. Add type to `SUPPORTED_ACTION_TYPES` in schema
2. Implement action execution in `UserTriggerManager.executeAction()`
3. Add form fields to `components/inline-action-editor.js` and parsing in `handlers/form-handlers.js`
4. Add manifest validation and i18n keys for labels

**Adding new condition types**:
1. Add to `TRIGGER_PARAMETER_FIELDS` in schema
2. Define available fields and input types
3. Implement event emission in pet renderer (pet.js)
4. Add filter UI support in panel rule editor/components if the field needs custom input
5. Add i18n keys for condition type and field names

**Adding new asset types**:
1. Add extension to `SUPPORTED_ASSET_EXTENSIONS` in schema
2. Update MIME type handling if needed
3. Asset validation automatically includes new types

**Adding new display properties**:
1. Add to `DEFAULT_DISPLAY` in defaults.js
2. Handle in `applyDisplay()` in pet renderer
3. Add IPC handlers if native window changes needed (ipc.js)
4. Add panel controls (panel.js)

## Testing Notes

- **138 unit tests** covering main, renderer, and shared modules (Vitest)
- **E2E tests** validate Electron app launch and basic rendering (Playwright)
- Run `npm run test:e2e:ci` in headless environments (skips Electron smoke test)
- Mock electron-store with in-memory implementation for tests
- Use `beforeEach` to reset global state in shared module tests (especially i18n)

## Project Context

- **License**: GPL-3.0
- **Target platforms**: macOS, Windows
- **Tech stack**: Electron 31, Vite 5, Vitest, Playwright
- **Primary language**: JavaScript (CommonJS for main process, ESM for renderer)
- **User base**: Chinese and English speakers (bilingual UI)

---
> Source: [duzexu/desktop-pet](https://github.com/duzexu/desktop-pet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-17 -->
