## drawverlay

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What it is

Drawverlay is an Electron desktop app that renders an always-on-top overlay for drawing on screen during presentations and recordings. Target platforms: macOS (primary), Windows (future).

## Commands

```bash
npm install        # install dependencies (electron + electron-builder)
npm start          # launch the app
npm run kill       # force-kill stuck electron process
npm run dist:mac   # build macOS distributable (DMG + ZIP, arm64 + x64)
npm run dist       # build for current platform
```

There is no test suite yet.

## Architecture

Root files + `components/` folder for all window assets:

| File | Role |
|------|------|
| `main.js` | Electron main process — windows, global shortcuts, IPC handlers, settings persistence |
| `preload.js` | Context bridge — exposes `window.drawverlay` API to renderer via `contextBridge` |
| `components/renderer.html` | Overlay shell — transparent canvas + bg div + cursor disc |
| `components/renderer.css` | Overlay styles |
| `components/renderer.js` | All drawing logic — canvas 2D, brush/eraser/rect/line/grid, easing loop, mouse events |
| `components/menu.html` | Tray panel — four tabs (Tools / Projects / Settings / Shortcuts) |
| `components/menu.css` | Menu styles |
| `components/menu.js` | Menu logic — color pickers, palette, settings controls |
| `components/tooltip.html` | Tooltip overlay window |
| `components/tooltip.css` | Tooltip styles |
| `components/tooltip.js` | Tooltip logic |
| `web/` | Astro marketing website — `npm run dev` / `npm run build` inside this folder |
| `web/src/` | Astro source — components, layouts, pages |
| `web/src/components/DrawingBackground.astro` | Embeds full drawing app (canvas + menu) in the hero background |
| `web/public/components` | Symlink → `../../components` — serves component JS/CSS at `/components/` |
| `web/public/bridge.js` | Web bridge — implements `window.drawverlay` without Electron (CustomEvents + localStorage) |

## Key implementation details

**Overlay window** (`main.js`)
- `transparent: true`, `frame: false`, `alwaysOnTop: true`, `hasShadow: false`, `focusable: false`
- `setAlwaysOnTop(true, 'screen-saver')` + `setVisibleOnAllWorkspaces` so it sits above full-screen apps and the macOS menu bar
- Sized to `workArea` of the target display (not `bounds`) — avoids bottom-overflow on macOS
- `app.dock.hide()` on macOS — utility overlay, not a regular app
- `focusable: false` means the overlay never steals focus, but it also means **keyboard events never fire in `renderer.js`** — handle keys via global shortcuts in `main.js` or by reading `e.shiftKey` from mouse events

**Menu window** (`main.js`)
- Separate `BrowserWindow` (`menu.html`), shown/hidden on tray click, right-click on canvas, or Escape key
- Draggable via manual IPC drag (`menu:drag-start` / `menu:drag`) — `-webkit-app-region: drag` was removed because it prevents CSS cursor changes
- Height is resized dynamically via `menu:resize` IPC whenever tab content changes
- `showMenu(pos?)` clamps position using `screen.getDisplayNearestPoint(pos).bounds` — always use this helper, never `setPosition` directly, to stay within the correct display

**Passthrough mode** (`main.js` ↔ `renderer.js`)
- `win.setIgnoreMouseEvents(!isVisible || isPassthrough, { forward: true })` — combined condition; overlay is always non-interactive when hidden OR in passthrough. `forward: true` keeps mousemove flowing for cursor tracking. Never call `win.show/hide()` — use `win.setOpacity(isVisible ? 1 : 0)` + the combined `setIgnoreMouseEvents` call instead (both `setVisible` and `setPassthrough` must apply this same combined condition)

**Global shortcuts**
- Shortcuts are **user-configurable** — stored in `settings.json` under `shortcuts`; merged with `DEFAULT_SHORTCUTS` on load via `{ ...DEFAULT_SHORTCUTS, ...data.shortcuts }`
- Default bindings: `Cmd+Shift+X` (visible), `Cmd+Shift+C` (passthrough), `Cmd+Shift+B` (background), `Cmd+Shift+Backspace` (eraseAll), `Cmd+Z` / `Cmd+Shift+Z` (undo/redo), `B/E/R/L/G` (tools), `S` (sizing), `Escape` (menu)
- `globalShortcutHandlers` map holds handlers for the four non-single-key global shortcuts (`toggleVisible`, `togglePassthrough`, `toggleBackground`, `eraseAll`); registered via `registerGlobalShortcut(action)`
- `registeredGlobalAccelerators` object tracks currently registered accelerator per action — used by `applyShortcutChange` to unregister the old one before registering the new
- `registeredUndoAccelerator` / `registeredRedoAccelerator` track undo/redo separately; re-registered by `updateUndoShortcuts()` which is called from both `setVisible` and `setPassthrough`
- `applyShortcutChange(action, newAccelerator)` — live-swaps one shortcut; routes to `registerGlobalShortcut`, `updateUndoShortcuts`, or `forceUpdateSingleKeyShortcuts` based on action category; saves settings; broadcasts `shortcuts-changed` to menu
- **Single-key shortcuts** (tool keys, S, Escape) tracked in `registeredSingleKeyAccelerators` array; `forceUpdateSingleKeyShortcuts()` tears down and rebuilds them (needed after a shortcut change because `singleKeyShortcutsActive` state must be reset)
- Single-key shortcuts must still be **physically unregistered** (not just ignored) when the cursor is off the overlay display — `globalShortcut` intercepts at OS level. Use `setInterval` polling + `isCursorOnOverlayDisplay()` to register/unregister dynamically
- `isCursorOnOverlayDisplay()` uses a raw bounds check (`cursor.x >= bounds.x && cursor.x < bounds.x + bounds.width && ...`) — do not rely on display ID comparison, it's unreliable

**Sizing mode** (`renderer.js`)
- `S` fires `sizing-mode-start` IPC with `{ tool, currentSize }` → renderer sets `isSizing = true`, anchors at current cursor position (`sizingStartX/Y`)
- While `isSizing`: cursor disc freezes at anchor; `#sizing-ring` and `#size-label` track the computed diameter; `size = Math.max(1, Math.min(200, Math.round(distance * 2)))` where `distance = Math.hypot(curX - sizingStartX, curY - sizingStartY)`
- Left-click commits (calls `setBrushSize` or `setEraserSize`); right-click cancels (restores `sizingOriginalSize`)
- `isSizing` hijacks the `tick()` loop's early-return path — the normal draw block never runs while sizing

**IPC pattern** (`main.js` ↔ `preload.js`)
- **Getters**: `ipcMain.handle('ns:get-field', () => value)` + `ipcRenderer.invoke('ns:get-field')` (returns a Promise)
- **Setters**: `ipcMain.on('ns:set-field', (_event, value) => { ... saveSettings(); win.webContents.send('field-changed', value); menuWin.webContents.send('field-changed', value) })`
- **Changed events**: most setters broadcast to both `win` and `menuWin`. Exceptions — `bg:set-color`, `bg:set-opacity`, `brush:set-opacity`, and `brush:set-easing` only broadcast to `win`; the menu is the source of truth for those values and doesn't need the echo back
- **Toggle handlers** follow the same pattern but take no value argument and flip state internally
- `preload.js` exposes `getX`/`setX`/`onXChanged` triples on `window.drawverlay` for each setting

**Initial state delivery**
- Overlay (`renderer.html`): all settings pushed via `win.webContents.send()` inside `win.webContents.once('did-finish-load', ...)`; active project canvas loaded via `canvas:load-data`
- Menu (`menu.html`): pulls numeric/string settings via `invoke` calls (e.g. `getBrushSize().then(...)`) on `DOMContentLoaded`; boolean state (passthrough, visible, tool, easing, bg-enabled) is pushed by `showMenu()` every time the menu opens

**Canvas / drawing** (`renderer.js`)
- `resizeCanvas()` preserves pixel content by copying to a temp canvas before resetting dimensions
- `applyBrushState()` re-applies `lineCap`/`lineJoin` after any canvas reset
- Drawing runs in a `requestAnimationFrame` loop that lerps `curX/curY` toward the real mouse position for smoothing — both the cursor disc and the stroke use the eased coordinates
- Smoothing formula: `factor = 1 - Math.pow(brushEasingCoeff, dt * 60)` — framerate-independent; `coeff = setting / 10 * 0.9`; coeff = 0 → instant
- Brush opacity (`brush.opacity`, 0–1) applied via `ctx.globalAlpha` on each draw call
- **Brush accumulation** (`brush.accumulation`): when `true` (default), opacity accumulates as strokes overlap. When `false`, the entire stroke is drawn at once on an offscreen canvas and composited at opacity — prevents mid-stroke build-up. Shift straight-line mode tracks `shiftBasePoints` and `shiftJustReleased` to manage transitions correctly in non-accumulation mode
- **Brush/eraser**: freehand stroke using `lineTo`. Shift = straight-line preview: restore `drawStartSnapshot` each frame and redraw a single constrained line via `snap45()`. Shift transitions mid-stroke: commit freehand on press, continue freehand on release
- **Rect tool**: captures `rect.startX/Y` and `drawStartSnapshot` on mousedown; each tick restores snapshot and redraws with `fillRect` + `strokeRect`. Shift = square (equal side length). Settings: `fillColor`, `fillOpacity`, `strokeColor`, `strokeOpacity`, `strokeThickness`
- **Line tool**: captures `line.startX/Y` and `drawStartSnapshot` on mousedown; each tick restores snapshot and redraws a single line. Shift snaps endpoint to nearest 45° via `snap45(startX, startY, endX, endY)`. Settings: `color`, `opacity`, `thickness`
- **Grid tool**: captures `grid.startX/Y` and `drawStartSnapshot` on mousedown; each tick restores snapshot and redraws `columns × rows` cells via individual `moveTo`/`lineTo` calls for interior lines plus an outer `strokeRect`. Shift = square cells (cell width forced equal to cell height). Settings: `color`, `opacity`, `thickness`, `columns`, `rows`, `closed` (draws outer border when true). Scroll wheel adjusts `thickness`
- **Cursor disc**: brush/eraser = circle sized to tool diameter; rect/line/grid = fixed 4×4 white dot with dark border
- **Scroll wheel** adjusts active tool's primary size: brush size (brush), eraser size (eraser), stroke thickness (rect/line/grid); `{ passive: false }` to allow `preventDefault()`
- Undo stack: `saveSnapshot()` on each mousedown; max 50 entries; stacks invalidated on canvas resize
- Canvas persistence: renderer listens for `canvas:serialize` → sends `canvas:data` with dataURL; listens for `canvas:load-data` → loads dataURL back onto canvas (or clears if null)

**Multi-display** (`main.js`)
- `getTargetDisplay()` — returns saved display or primary if unavailable
- `repositionOverlay()` — moves overlay to target display's `workArea`
- `buildDisplayList()` — snapshot of all displays for the Settings select
- Display change events (`display-added`, `display-removed`) fall back to primary and notify the menu
- When switching display with menu open, menu is repositioned to top-center of the new display

**Settings persistence** (`main.js`)
- JSON file at `app.getPath('userData')/settings.json`
- Fields: `brushSize`, `brushColor`, `brushOpacity`, `brushAccumulation`, `eraserSize`, `activeTool`, `brushEasing`, `bgColor`, `bgOpacity`, `bgEnabled`, `displayId`, `palette`, `rectFillColor`, `rectFillOpacity`, `rectStrokeColor`, `rectStrokeOpacity`, `rectStrokeThickness`, `lineColor`, `lineOpacity`, `lineThickness`, `gridColor`, `gridOpacity`, `gridThickness`, `gridColumns`, `gridRows`, `gridClosed`, `shortcuts`, `projects`, `activeProjectId`
- `activeTool` always resets to `'brush'` on startup (not restored from disk)
- `palette` is an array of user-added hex strings (max 30); preset palette lives only in `menu.html`

**Projects** (`main.js`)
- Projects are named canvas slots stored in `app.getPath('userData')/projects/{id}.png` (dataURL text files)
- `projects` array in settings: each entry is `{ id, name }` where `id` is a timestamp string
- `activeProjectId` tracks the current project; always the last active one on launch
- `migrateProjects()` runs on startup — creates a "Default" project if none exist (one-time migration)
- Operations: `projects:create`, `projects:switch`, `projects:rename`, `projects:delete` — all `ipcMain.handle` calls that serialize the current canvas before switching, then load the new project's canvas
- Canvas autosaves to the active project 2 seconds after any `canvas:dirty` IPC event (`scheduleAutosave` / `cancelAutosave`); also saved synchronously before quit (`savingBeforeQuit` guard prevents double-quit) and before switching projects

**Export** (`main.js`)
- `export:png` IPC handler serializes the canvas, shows a native save dialog, and writes the PNG file
- Default filename: `drawverlay-export-YYYY-MM-DD.png`
- Triggered from the active project's row in the Projects tab (export button only shown on the active project)

**Login item** (`main.js`)
- `app:get-login-item` / `app:set-login-item` — reads and writes `app.getLoginItemSettings()` / `app.setLoginItemSettings()`

**Color wheel** (`menu.html`)
- HSV model; pure functions `hsvToRgb`, `hsvToHex`, `hexToHsv` defined once and reused for both brush and background color pickers
- Background color picker uses separate `bgH/bgS/bgV` state and separate canvas elements
- `requestAnimationFrame` needed to sync brightness bar canvas size when its container transitions from `display:none` to `display:block`
- When brush color changes via IPC (`brush-color-changed`), call `setColorFromHex` to keep wheel in sync — `brush:set-color` handler in `main.js` must send back to `menuWin` as well as `win`

**Background overlay** (`renderer.html` / `renderer.js`)
- `#bg` div sits behind `<canvas>` — both must be `position: relative` (or both positioned) or DOM order alone determines stacking
- `pointer-events: none` on `#bg` so it never intercepts mouse events

**Action toast** (`renderer.js` / `main.js`)
- `#action-toast` div in `renderer.html` — absolutely positioned, tracks the cursor via `transform` in `tick()` while `toastActive` is true
- `showActionToast(message)` — if message ends in `on` or `off`, splits and color-codes the state word (yellow for on, red for off); restarts animation via `void offsetWidth` reflow trick; clears `toastActive` after 1 s
- Triggered by `action-toast` IPC from `main.js` on: tool switch, visibility toggle, passthrough toggle, background toggle, erase all

**Shortcuts tab** (`menu.js`)
- `SHORTCUT_DEFINITIONS` — ordered array of `{ group }` headers and `{ action, label }` entries defining what appears in the Shortcuts tab
- `renderShortcutsPanel(data)` — builds rows from `SHORTCUT_DEFINITIONS`; shows a reset button per row when `data[action] !== defaultShortcuts[action]`
- `startCapture(action, badgeEl)` — enters key-capture mode; calls `setMenuInputFocused(true)` to suppress single-key global shortcuts; Escape cancels, any other key-combo commits via `window.drawverlay.setShortcut`
- `buildAccelerator(event)` — converts a `KeyboardEvent` to an Electron accelerator string (`CommandOrControl`, `Shift`, `Alt` prefixes + bare key)
- `formatShortcutHTML(accelerator)` — renders accelerator as `<span class="key-cap">` elements; uses SVG icons for Shift/Ctrl/Option on macOS, text fallbacks on Windows
- `updateSettingsTabBadges(data)` — updates tooltip text on Visible/Passthrough labels and Undo/Redo header buttons to reflect current shortcuts
- Tool button tooltips (`renderToolGroup`) pull live accelerator from `currentShortcuts` via `shortcutAction` field on each tool definition

**Palette** (`menu.html` + `main.js`)
- Preset colors are defined only in `menu.html` (const array) — they are never stored in settings
- User colors stored in `palette` setting; broadcast via `palette-changed` IPC to `menuWin` only (overlay doesn't need palette state)
- Active swatch is highlighted by comparing `color.toLowerCase() === currentBrushColor.toLowerCase()`

**`menu.html` helpers**
- `makeDraggable(input, applyFn)` — attaches mousedown/move/up to a stepper `<input>`: horizontal drag of 3px = 1 unit change. Use for any new numeric stepper.
- `resizeToActiveTab()` — measures active tab content and fires `menu:resize` IPC to resize the window. Must be called whenever content height may change (palette re-render, tab switch, toggle sections).

## Coding conventions

@CONVENTIONS.md

---
> Source: [brunosimon/drawverlay](https://github.com/brunosimon/drawverlay) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
