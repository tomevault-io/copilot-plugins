## pixeleditor

> Pixel icon editor: PySide6 GUI that edits pixels, layers, and animations, and

# AGENTS.md

## What this is

Pixel icon editor: PySide6 GUI that edits pixels, layers, and animations, and
exports multi-size `.ico`, `.png`, and `.svg` files. Windows target, Python
3.10+ (3.14 confirmed).

## Run

```bash
pip install PySide6 pillow
python main.pyw       # console visible; use pythonw main.pyw for silent GUI
```

No CI. Tests: `python -m pytest -q`. Lint report: `Errors.cmd` → `Errors.md`.

## Architecture (multi-file — NOT single-file anymore)

- `main.pyw` — entry point only: `from src.app import main`
- `src/app.py` — `MainWindow`: wires panels ↔ canvas, file import/export dialogs,
  shortcuts, recent files, autosave, themes, plugin UI, settings persistence
  (`_restore_settings`/`_save_settings` via `QSettings("PixelEditor", "Settings")`).
  Multiple documents: each tab is a `_Document` (canvas + file + `dirty` flag)
  in a `QTabWidget`; `self.canvas` and `self._current_file` are properties that
  delegate to the active document, so all handlers operate on the current tab
  without call-site changes. `_add_document`/`_close_tab`/`_activate_document`/
  `_sync_ui_to_document` manage tabs; `_apply_window_settings(canvas)` pushes
  the window-level view toggles (grid/mirror/onion/EAL/brush preview/tolerance/
  brush size) onto every canvas; `_wire_canvas` sets the color/stack/modified
  callbacks + event filter per canvas. `canvas._on_modified` fires
  `_mark_dirty()` which sets the active doc's `dirty` flag and refreshes the
  tab/window title with a `*` marker; saving (`_write_project`), loading
  (`_load_file`), and a fresh document clear it. File → Save (Ctrl+S) =
  `_save_current` (Save As for untitled docs), File → Save As = `_save_project`.
  `_close_tab`/`closeEvent`/`_load_file` prompt Save/Discard/Cancel (or
  Yes/No for replace) when a document is dirty.
- `src/canvas.py` — `PixelCanvas(QWidget)`: pixel ops, tools, layers, selection,
  zoom, grid, flood fill, mirror, undo/redo, drag-and-drop. Transforms
  (`rotate_cw`/`rotate_ccw`/`rotate_180`/`flip_h`/`flip_v`) apply to the
  selection content when a selection exists, else the active layer in place
  (canvas dims unchanged; non-square content is centered). All are undoable,
  blocked on a locked layer, and use exact PIL transposes
  (`_transform_qimage` → `qimage_to_pil_rgba` + `pil_to_qimage_exact` in
  `helpers.py`).
- `src/panels.py` — `LeftPanel` (canvas/file/export/edit/view/plugins)
  and `RightPanel` (tools/color/layers/anim); `ColorSwatch`, `ToolButton`
- `src/dialogs.py` — `ResizeDialog` (with 9-point anchor), `ExportPreviewDialog`,
  `ExportDialog` (unified ICO/PNG/per-layer export), `ShortcutOverlay`,
  `AnimationPreviewDialog`, `AnimationExportDialog`,
  `PluginDialog`, `PaletteDialog` (GPL/PAL/hex import-export),
  `QuantizeDialog` (2–256-color reduction, live preview, palette extraction)
- `src/export.py` — `export_ico`, `export_png`, `export_svg`, `batch_export`,
  `export_layers`/`layer_images`/`layer_file_names`,
  `export_animation`/`save_animation` (GIF/APNG), `_warn_upscale`
- `src/project.py` — `save_project`/`load_project` (.icon ZIP: meta.json +
  per-layer PNG), `PROJECT_MAGIC`
- `src/helpers.py` — `qimage_to_pil_rgba`, `pil_to_qimage_rgba`,
  `bresenham_line`, `rect_pixels`, `ellipse_pixels`, `polygon_pixels`,
  `parse_palette_text`/`palette_to_text` (pure, tested)
- `src/plugins.py` — `PluginInfo`, `load_all_plugins()` scans the app
  `plugins/` dir and `%APPDATA%\PixelEditor\plugins\` (via
  `QStandardPaths.AppDataLocation`), accepting `.py`, `.zip`, and
  sub-folders per source; zips stage to `tempfile.mkdtemp`, cleanup runs
  in `MainWindow.closeEvent`. `load_plugins(plugins_dir)` is kept for
  single-directory use.
- `src/constants.py` — `PALETTE_COLORS`, `TOOL_EMOJIS`, `TOOL_CURSORS`,
  `ALL_TOOLS`, `MAX_RECENT_FILES`
- `src/version.py` — single source of truth: `__version__`, `APP_NAME`
  (mirrored in `pyproject.toml`; `tests/test_version.py` enforces the
  match). The window title (`_update_title`) and `AboutDialog` (`Help → About`)
  stamp the version.
- `src/strings.py` — all user-facing text in one flat `dotted.key` table
  (`STRINGS`) plus a `t(key, **fmt)` helper. Call sites in
  `dialogs.py`/`panels.py`/`app.py`/`export.py` look up via `t("...")`.
  Unknown keys fall back to the key itself; placeholders use
  `str.format(**fmt)`. Adding a translation = parallel module with the
  same keys, picked at runtime.

## Key details

- Image format throughout: `QImage.Format_RGBA8888`. QRgb is `0xAARRGGBB`;
  use `_qrgb_from_qcolor` in `canvas.py` for conversion.
- `canvas.image` is a property bound to the **active layer's** image
  (`_layers[active]`). For rendering/export, use `canvas.composite_image()`
  (bottom layer first, honoring visibility + opacity) — `export.py`,
  autosave, and the export preview all use the composite now.
- Dirty tracking: `canvas._on_modified` (a `Callable[[], None]`, set by the app
  in `_wire_canvas` to `_mark_dirty`) fires from `_save_undo_state()`,
  `undo()`, `redo()`, and `set_image_size()` — so every stroke, resize, and
  undo/redo past a save marks the document dirty. Direct `set_pixel_rgba`
  calls (tests, programmatic) do NOT mark dirty.
- Layer-flag toggles are undoable: `toggle_layer_visibility`,
  `set_layer_protect_alpha`, `set_layer_locked` each push undo before
  mutating, and the right-panel opacity slider saves one undo state on
  `sliderPressed` (so a full drag is a single undo step). This keeps the
  flag values stored in undo snapshots consistent with the live layers.
- Layers are `list[dict]` with keys `image`, `name`, `visible`, `opacity`,
  `protect_alpha`, `locked`. Index 0 is the **bottom** layer (`merge_down`
  paints `layers[index]` onto `layers[index-1]`). Undo/redo stores full layer
  snapshots (`_snapshot_layers`), max 80. `protect_alpha` (per-layer checkbox
  in the layer list) makes `set_pixel_rgba` skip fully-transparent pixels, so
  brush/fill/replace can't paint outside existing pixels; `locked` (also
  per-layer) gates `set_pixel_rgba`, `_replace_color`, `flood_fill`, and the
  shape/polygon commit + selection-mutating paths so the active layer becomes
  read-only (eyedropper still reads). `_flood_fill_image` returns early (no
  infinite loop) when `protect_alpha` is on and the fill start pixel is fully
  transparent. `clear_canvas` skips locked layers; `merge_down` refuses a
  locked bottom. Both flags survive undo/redo, `duplicate_layer`,
  `resize_canvas`, and `.icon` save/load.
- Edit-menu state: `MainWindow._update_edit_actions` enables/disables the
  `_menu_undo/_menu_redo/_menu_cut/_menu_copy/_menu_paste/_menu_commit/
  _menu_del` QActions from the same source as the left-panel undo/redo
  buttons, refreshed on stack change, selection change (canvas
  `_on_selection_changed` callback), and tab switch. Menu actions and the
  Ctrl+Z/Ctrl+Y shortcuts bind to the **active** canvas via lambdas (not the
  first document's methods).
- Selection changes notify via `canvas._on_selection_changed` (called from
  `_clear_selection`, marquee release, flood select, paste, delete, commit,
  copy/cut, and selection transforms); `canvas.has_clipboard_image()` reports
  whether paste has content (internal buffer or OS clipboard).
- Canvas is square by default via size combo, but `resize_canvas(w, h)` supports
  non-square (1–1024); ResizeDialog allows W ≠ H.
- ICO export uses Pillow `save(format="ICO")` with NEAREST resampling.
- Animation export (`export_animation`): one frame per **visible** layer,
  honoring per-layer opacity; GIF/APNG via Pillow `save_all`. Animation
  preview (`AnimationPreviewDialog`) iterates layers in list order (bottom
  first) and composites each frame at its layer opacity (same as export).
- `.icon` project files are ZIPs (`project.py`); autosave writes
  `<base>.autosave.icon` next to each **dirty** document's file (every tab, not
  just the active one) and records the list in QSettings
  (`_auto_save`/`_last_autosave_paths`/`_clear_autosave`), with a one-shot
  recovery prompt per file at startup (`_check_autosave_recovery` restores each
  into its own tab).
- Onion skin (`canvas.onion_skin`, `canvas.onion_opacity` 0–1): when on and
  `_active_layer > 0`, `paintEvent` ghosts **all** earlier layers at
  `onion_opacity` (skipping hidden layers), then draws the active-and-up stack
  with per-layer opacity; when off it uses the normal composite pixmap cache.
  The left panel's Onion % slider (1–100) drives it; `onion_skin` /
  `onion_opacity` persist via `QSettings`.
- Grab-based canvas tests: probe interior pixels (`zoom//2`, e.g. `(2,2)` at
  zoom 4) — `(0,0)` lands on a Qt `QImage.scaled` edge blend and is unreliable.
  `resize(w*zoom, h*zoom)` may be clamped by the `_fit_zoom` minimum size;
  content is still drawn at (0,0), so interior probes stay valid.
- Shape tools (Line/Rectangle/Ellipse/Filled Rectangle) share the drag
  `_shape_start/_shape_end/_shape_snapshot` path; `_shape_pixels(start, end)`
  resolves pixels per tool and `_commit_shape` finalizes (snapshot restore +
  brush). Polygon is click-to-accumulate `_poly_vertices`, double-click commits
  (`_commit_polygon`), Escape cancels (`_cancel_polygon`). Spray scatters
  `brush_size`-radius dots along the drag line via `_paint_spray_line`.
  Color Replace uses `canvas.replace_tolerance` (0–100, per-channel max
  difference via `colors_within_tolerance` in `helpers.py`); a "Replace Tol:"
  slider in the right panel drives it and persists via `QSettings`.
- SVG import uses PySide6 `QSvgRenderer` (intrinsic size via
  `renderer.defaultSize()`, scaled to the target, aspect-preserving), no cairo.
- PNG/SVG import (`_apply_pil_to_canvas`) preserves aspect ratio: the image is
  scaled to fit the largest dimension of the target and the canvas is resized
  to the fitted w×h (non-square imports are no longer squashed).
- Imported images (PNG/SVG/ICO) do NOT bind `_current_file`: only `.icon`
  project loads set the document's save path, so Ctrl+S on an imported image
  goes to Save As instead of overwriting the source image with a `.icon` ZIP.
- ICO import: `src/export.py::read_ico_sizes(path) -> list[(w,h)]` (sorted
  largest-first) and `load_ico_image(path, size)` (Pillow `IcoImageFile.size
  = size` then `.load().convert("RGBA")`); `src/dialogs.py::IconImportDialog`
  shows a thumbnail grid (largest preselected) when the file has >1 frame.
  Single-size files import directly via the existing `_load_file` ICO branch.
  New "Import ICO" button on the left panel; strings in
  `src/strings.py` (`window.title.import_ico`, `dialog.import_ico.body`,
  `panel.left.button.import_ico`, `file_filter.ico`).
- Right-click = erase / fill-with-transparent (independent of tool).
- Batch editing: `canvas.edit_all_layers` (False by default; "Edit All
  Layers" checkbox in the left-panel View section, persisted via QSettings)
  makes every pixel op write to **all visible, unlocked** layers at once —
  pencil/eraser/dither/spray and right-click erase via `set_pixel_rgba`,
  lighten/darken via `_adjust_brightness`, flood fill via `_flood_fill_image`
  per layer, color replace via `_replace_color`, and shape/polygon commits
  (whose drag-preview snapshots are `dict[int, QImage]` from
  `_snapshot_batch()`). Hidden and locked layers are skipped; a locked
  active layer still blocks the stroke. Selection cut/copy/paste remain
  active-layer-only.
- Plugin API: `TOOL_NAME`, `TOOL_EMOJI`, `DESCRIPTION`, and `apply(canvas, color)`
  in a `plugins/*.py` file. v2 plugins (`PLUGIN_API = 2`) add optional hooks
  (`on_tool_select`, `on_layer_change`, `on_canvas_change`, `customize_menu`)
  and per-plugin settings (`PLUGIN_SETTINGS` schema + `app.get_plugin_settings`
  / `app.set_plugin_setting`, persisted under `QSettings("PixelEditor",
  "PluginSettings")`). Plugins declaring `customize_menu` add entries to a
  **Plugins** menubar menu instead of the left-panel button. v1 plugins keep
  loading silently. See `docs/PLUGINS.md`.
- Selection: `_selection: QRect` (bounding box), optional `_selection_mask:
  QImage` (per-pixel alpha over the box; `None` = full-rect selection) plus
  `_selection_edges` (outline coords) and `_selection_overlay` (tint image)
  used by `paintEvent` for marching-ants + blue fill. The Select tool drags a
  fresh rectangular marquee; the right-panel **Magic Wand** checkbox
  (`canvas.magic_wand`, shares the Replace Tol slider as tolerance) makes a
  click flood-select the connected same-color region of the active layer
  (`_create_flood_selection`). Hit-testing uses `_in_selection(x, y)`;
  cut/copy/paste/delete/commit and drag-to-move respect the mask via
  `_extract_selection` / `_clear_selection_region`. Ctrl+C/X/V/Shift+V,
  Delete; drag to move. Selection ops are active-layer-only (never batch).
  When a selection transform (`_apply_transform`) has to crop the transformed
  content back to canvas bounds, the same crop rect is applied to the mask and
  overlay so `_selection_mask` always matches `_selection`/`_selection_image`
  dimensions.
- Layer list select buttons (`LeftPanel._layer_select_buttons`) are exclusive:
  clicking one checks only the active layer's button (driven by
  `_activate_layer`, not a bare `set_active_layer` lambda), so the checked
  state never goes stale after tab switches or programmatic activation.
- Status bar coords/color via `eventFilter` on canvas MouseMove.
- `QAction` is in `PySide6.QtGui`, NOT `PySide6.QtWidgets`.

## Layout

left panel (canvas/file/edit controls + plugins, 180px) | canvas (stretch) |
right panel (tools/color/palette + layers/animation, 220px). The left panel
is split into 6 collapsible `CollapsibleSection` blocks (Canvas/File/Export/
Edit/View/Plugins) whose header buttons toggle their contents; widgets live
in `section.content_layout` but are still reachable as `self.left_panel.*`
attributes. The right panel holds the tool grid, a Draw Mode combo
(`self.right_panel.draw_mode_combo`: Single/Double/Quad → mirror flags,
drives the left-panel `mirror_x_check`/`mirror_y_check` via
`MainWindow._on_draw_mode_changed`/`_set_mirror_flags`/`_sync_draw_mode_combo`;
mapping: Single=(False,False), Double=(True,False), Quad=(True,True)),
brush-size slider (`self.right_panel.brush_slider`, `[`/`]` shortcut),
tolerance slider (`self.right_panel.replace_tolerance_slider`), color, recent
colors, palette, and collapsible Layers/Animation sections
(`self.right_panel.refresh_layers_ui`), and has no scrollbars: the palette
grid is placed directly in the layout (no `QScrollArea` wrapper).

## Keyboard shortcuts

P/E/F/I/L/R/O/H/V/T/S/J/D/U/K = tools; G = grid; Ctrl+Z/Y = undo/redo;
Ctrl+C/X/V/Shift+V + Delete = selection; Ctrl+0/+/− = zoom; `[`/`]` = brush;
Ctrl+M / Ctrl+Shift+M = mirror X/Y; Ctrl+Shift+R = resize; Ctrl+B = export
(one unified ExportDialog: ICO / PNG / SVG / per-layer PNG / batch);
Ctrl+E = export preview; Ctrl+T = new tab; Ctrl+W = close tab; Ctrl+S = save
current document; Ctrl+R = rotate 90° CW; `?` = shortcut
overlay.
Standard menu bar: File (New/Close Tab/Open/Save/Save As/Recent/Export/Exit), Edit
(Undo/Redo/Cut/Copy/Paste/Resize/Clear/Rotate/Flip/Color), View (Zoom/Grid/Dark/Onion/
Mirror/EAL), Plugins, Help (Shortcuts/About).

## Testing / validation

```powershell
python -m pytest -q                     # unit tests (tests/)
python -m py_compile main.pyw           # syntax
Errors.cmd                              # mypy/ruff/pyflakes/smoke report → Errors.md
```

Known status (2026-08-02): **all green** — 250 tests pass, and `Errors.cmd`
reports PASS on mypy, ruff check, ruff format, smoke, pyflakes, and syntax.
`Errors.cmd` lints the whole `src/` + `tests/` tree (not just `main.pyw`).
GitHub Actions CI: `.github/workflows/test.yml` (pytest + Errors.cmd on Linux
and Windows) and `.github/workflows/build.yml` (Windows PyInstaller
artifact).

## Build

`build.cmd` (venv + deps + `PyInstaller PixelEditor.spec`) → `dist\PixelEditor.exe`
(onefile). `clean.cmd` wipes build artifacts. See BUILD.md.

## Common pitfalls

- `pythonw` may not be on PATH — use full path or `python main.pyw`.
- When editing layers, always call `canvas._save_undo_state()` before mutating
  state (the MainWindow layer handlers do this).
- `_fit_zoom()` clamps zoom to 2–48; `target // max(w, h)`.
- `LeftPanel.refresh_layers_ui` rebuilds layer buttons; it deletes and recreates
  child widgets, so don't cache references to layer buttons.
- Tool keys are guarded by `_select_tool_if_no_focus` so typing in a `QLineEdit`
  (e.g. editable ICO-sizes combo) doesn't switch tools.

---
> Source: [shaheeniptv1/PixelEditor](https://github.com/shaheeniptv1/PixelEditor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-14 -->
