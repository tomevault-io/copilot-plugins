## digitalsreeni-image-annotator

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

DigitalSreeni Image Annotator - PyQt6 desktop app for image annotation with SAM 2 integration and multi-dimensional image support.

**Fork of**: https://github.com/bnsreenu/digitalsreeni-image-annotator

## Quick Reference

```bash
# Install
pip install -e .

# Run
python -m src.digitalsreeni_image_annotator.main
# or: digitalsreeni-image-annotator
# or: sreeni
```

## Tech Stack

Python 3.10+ | PyQt6 6.7+ | Ultralytics 8.3.27 (SAM 2) | NumPy | OpenCV | Shapely

**Test suite**: `tests/` (pytest + pytest-qt). 65 tests pass on PyQt6.

## Documentation

For detailed architecture and design information, see **[docs/](docs/)**:

- **[Building Block View](docs/05_building_block_view.md)** - Components, data model, class responsibilities
- **[Runtime View](docs/06_runtime_view.md)** - Workflows and key scenarios
- **[Cross-cutting Concepts](docs/08_crosscutting_concepts.md)** - Coordinate systems, conversions, patterns
- **[Architecture Decisions](docs/09_architecture_decisions.md)** - Why we made key choices
- **[Glossary](docs/12_glossary.md)** - Terms, acronyms, data structures

See [docs/README.md](docs/README.md) for full documentation index.

## Project Structure

```
src/digitalsreeni_image_annotator/
├── main.py                    # Entry point
├── annotator_window.py        # ImageAnnotator - main window, project state
├── image_label.py             # ImageLabel - display, mouse events, rendering
├── sam_utils.py               # SAMUtils - SAM model management
├── utils.py                   # Utility functions
├── export_formats.py          # COCO, YOLO, Pascal VOC exporters
├── import_formats.py          # COCO, YOLO importers
└── [tool dialogs]             # Standalone utility windows
```

## Key Classes

| Class | File | Responsibility |
|-------|------|----------------|
| `ImageAnnotator` | annotator_window.py | Main window, state (`all_annotations`, `class_mapping`, etc.) |
| `ImageLabel` | image_label.py | Image display, zoom/pan, annotation interaction |
| `SAMUtils` | sam_utils.py | Load SAM models, run inference |

See [Building Block View](docs/05_building_block_view.md) for detailed class documentation.

## Common Development Tasks

### Adding a New Annotation Tool

1. Add button in `ImageAnnotator.create_tool_section()`
2. Set `image_label.current_tool` on click
3. Handle mouse events in `ImageLabel` (mousePressEvent, mouseMoveEvent)
4. Render in `ImageLabel.paintEvent()`
5. Call `main_window.add_annotation()` to commit

### Working with Annotations

```python
# Annotation storage: dict[image_filename, list[annotation_dict]]
self.all_annotations[self.image_file_name].append({
    "segmentation": [x1, y1, x2, y2, ...],  # Polygon
    # OR "bbox": [x, y, width, height],     # Rectangle
    "category": class_name
})
```

### SAM Integration

SAM runs in-process; the Ultralytics model object lives on `SAMUtils`
and persists across calls. Inference runs on a background QThread but
the public API is synchronous — see ADR-013 in
`docs/09_architecture_decisions.md`.

```python
# Load model on first selection (downloads weights if missing, ~40-400MB)
self.sam_utils.change_sam_model("SAM 2 tiny")  # blocks UI thread via QEventLoop spin

# Run inference (also runs on worker thread, returns when done)
prediction = self.sam_utils.apply_sam_points(
    qimage,
    positive_points=[(x1, y1)],
    negative_points=[(x2, y2)]
)
# Returns: {"segmentation": [...], "score": float}
```

## Important Notes

### Platform Support
- ✅ Windows, macOS, Linux supported (PyQt6 native integration improved over PyQt5)
- Linux runtime needs libxcb-cursor0 (Qt6 requires this; was optional under Qt5)

### Critical: Project Loading
**Always check `is_loading_project` flag before saving!** Autosave during load corrupts files (v0.8.12 fix).

```python
def save_project(self):
    if self.is_loading_project:
        return  # Skip during load
    # ... save logic
```

### Coordinate Systems
- **Mouse events**: Screen coordinates → must convert to image coordinates
- **Annotations stored**: Image coordinates (unzoomed, absolute pixels)
- Account for `zoom_factor`, `offset_x`, `offset_y`

See [Cross-cutting Concepts](docs/08_crosscutting_concepts.md#coordinate-systems) for details.

### Multi-dimensional Images
- User assigns dimensions (T, Z, C, H, W) via dialog
- Slices extracted with names like `stack.tif_T0_Z5_C0`
- Each slice annotated independently
- Stored in `image_slices` dict
- TIFF axis hint: `load_tiff` reads `tifffile.series[0].axes` and pre-fills the dimension dialog; ndim≥5 had a `[-ndim:]` slice bug that produced 2560 wrong slices on a 5D `TZCYX` file — see arc42 if you touch this

See [Runtime View](docs/06_runtime_view.md#multi-dimensional-image-loading) for workflow.

### Patterns introduced in v0.9.0 (read before touching these areas)

| Area | Pattern | Why |
|------|---------|-----|
| Pan / zoom-to-cursor in scroll area | Use `event.globalPosition()` for pan; derive post-zoom offset from `viewport().width()`, not `self.width()` | Widget-local coords absorb half the pan delta as the widget shifts; `self.width()` is stale during zoom-out before layout settles. See [Pan + Zoom Reference Frames](docs/08_crosscutting_concepts.md#pan--zoom-reference-frames). |
| Dark mode contrast | No hardcoded `background:` / `color:` in widget `setStyleSheet(...)` | Hardcoded greys override `soft_dark_stylesheet.py` and punch bright boxes into the sidebar. Add a global rule first, then write the widget. See [No Hardcoded Colors Rule](docs/08_crosscutting_concepts.md#dark-mode--no-hardcoded-colors-rule). |
| DINO review state | `image_label.temp_annotations` is a single field, **not** per-image — must be re-synced from `dino_batch_results` on every image/slice switch via `_refresh_dino_temp_for_current` | Otherwise the first image's masks bleed onto every subsequent slice during navigation. See [DINO Temp Annotations](docs/08_crosscutting_concepts.md#dino-temp-annotations--single-field-many-images). |
| DINO batch over stacks | Use `_collect_dino_batch_work_items()` to flatten regular images + every loaded slice; don't iterate `self.all_images` directly | Multi-dim images appear in `all_images` as a single entry — slices live in `self.image_slices[base_name]` and were silently skipped. |
| DINO Enter/Escape during review | Application-wide `_DINOReviewEventFilter`, gated on pending temp_annotations + no modal + no text input | `QListWidget` consumes Enter for `itemActivated` before `ImageLabel.keyPressEvent` sees it. See [ADR-015](docs/09_architecture_decisions.md#adr-015-application-wide-event-filter-for-dino-review-shortcuts). |
| Auto-accept dropdown | Honored by **both** `run_dino_detection_single` and `run_dino_detection_batch` | Easy to forget in the single path because the combo is labeled "batch". |
| GPU model unload | `model.cpu()` → `gc.collect()` → `torch.cuda.empty_cache()` + `ipc_collect()` + `synchronize()` — full reclaim requires app restart due to per-process CUDA context | Setting refs to None alone leaves circular refs pinned and shows zero Task Manager drop. See [Releasing Model GPU Memory](docs/08_crosscutting_concepts.md#releasing-model-gpu-memory). |
| Export image-path lookup | Exact-key match first, substring fallback only | `"bee.jpg" in "honeybee.jpg"` is True — substring-only matching writes the wrong file. See [Export Format Filename Matching](docs/08_crosscutting_concepts.md#export-format-filename-matching). |
| F2 / global shortcuts | Use `QShortcut` with `Qt.ShortcutContext.ApplicationShortcut`, not `keyPressEvent` | `QTableWidget` consumes F2 for in-cell edit before it bubbles up. |

## Development Workflow

**CRITICAL: Always use feature branches — NEVER commit directly to master.**

| Step | Action | Notes |
|------|--------|-------|
| 1 | Create branch: `git checkout -b feature/short-description` | Before any changes |
| 2 | Implement feature | Follow patterns below |
| 3 | Run manual tests | See Testing Checklist |
| 4 | Update arc42 docs if behavior changed | `docs/` — see Documentation section |
| 5 | **Run senior reviewer agent** | `.claude/agents/senior-reviewer.md` — mandatory quality gate before every PR |
| 6 | Commit: `feat: Description` or `fix: Description` | Clear, descriptive messages |
| 7 | Push & create PR | `git push origin feature/branch` |

### Testing Checklist (Manual — No Automated Tests)

Before opening a PR, verify at minimum:

1. **Launch the app** — no import errors, main window renders
2. **Golden path** — perform the new feature's primary workflow end-to-end
3. **Edge cases** — empty state, cancel/escape, large images, missing model files
4. **Dark mode** — toggle and check rendering of new UI elements
5. **Save/load roundtrip** — if the feature touches `.iap` project files, save, close, reopen, verify state restored
6. **Adjacent features** — verify no regression in SAM, annotation tools, export formats
7. **Inference features** — if touching `sam_utils.py` or `dino_utils.py`, verify the model loads end-to-end (no silent load failure), returns masks/boxes, and the UI stays responsive during inference (timers, redraws, progress dialog cancels keep firing — see ADR-013)

### arc42 Documentation Update Rules

When a change affects behavior documented in `docs/`, update the docs in the same PR:

| Change Type | Update Target |
|-------------|---------------|
| New component/module | `05_building_block_view.md` — add to component table |
| Changed runtime behavior | `06_runtime_view.md` — update workflow description |
| New UI pattern or concept | `08_crosscutting_concepts.md` — add section |
| Architecture decision | `09_architecture_decisions.md` — record ADR |
| New domain term | `12_glossary.md` — add definition |

### Quality Gate — Senior Reviewer

Before every PR, run the senior reviewer agent (`.claude/agents/senior-reviewer.md`).

This is **mandatory** — the agent performs an independent end-of-implementation review:
- Reads the actual diff, not commit messages
- Ranks issues P0 (blocks merge) / P1 (should fix) / P2 (nit)
- Checks CLAUDE.md compliance (feature branches, coordinate systems, `is_loading_project` guards, DINO config persistence, in-process inference re-entrancy guards)

**Run it in the foreground** — never `run_in_background: true`. The review is a blocking quality gate: the next steps (address P0s, push, open PR) depend on its findings. Launch the agent and wait for the result before doing anything else, then iterate until clean.

Address all P0s before merging. Address P1s unless there's explicit justification.

## Known Constraints

- No type hints (gradual addition encouraged)
- Print statements instead of logging (acceptable)
- Absolute paths in projects (not portable)
- SAM 2 large crashes on limited RAM
- YOLO training not supported for multi-dimensional images

See [Risks and Technical Debt](docs/11_risks_and_technical_debt.md) for full list.

## Keyboard Shortcuts

| Global | Action |
|--------|--------|
| Ctrl+N/O/S | New/Open/Save Project |
| F1 | Help |

| Canvas | Action |
|--------|--------|
| Ctrl+Wheel | Zoom |
| Ctrl+Drag | Pan |
| Enter | Finish/Accept |
| Esc | Cancel |
| Up/Down | Navigate slices |
| -/= | Brush size |

## Quick Tips

- SAM models cache in working directory (Ultralytics)
- Recommend SAM 2 tiny/small (avoid large)
- Polygon area uses shoelace formula (utils.py)
- Export formats copy images to output directory
- Dark mode changes annotation colors for visibility
- Snake game is hidden Easter egg 🐍

---
> Source: [bnsreenu/digitalsreeni-image-annotator](https://github.com/bnsreenu/digitalsreeni-image-annotator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-01 -->
