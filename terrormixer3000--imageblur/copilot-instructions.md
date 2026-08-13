## imageblur

> This file is a contributor-facing engineering guide for the ImageBlur repository.

# AGENTS

This file is a contributor-facing engineering guide for the ImageBlur repository.

## Project Overview
- `ImageBlur` is a native macOS app built with `SwiftUI`.
- The goal is non-destructive pixelation of image regions without changing the original image dimensions.
- Supported shapes in the current version:
  - Rectangle
  - Ellipse
  - Lasso
- Regions can be created, selected, moved, resized, rotated, and deleted.
- When opening another image while unsaved changes exist, the app must show a save/discard/cancel confirmation instead of silently replacing the current editing state.

## Tech Stack
- `Swift 6`
- `Swift Package Manager`
- `SwiftUI` for the app and editor UI
- `Core Image` for pixelation
- `ImageIO` for loading and exporting in the original format

## Project Structure
```text
Package.swift
Sources/ImageBlur/
├── ImageBlurApp.swift
├── Models/
│   ├── BlurRegion.swift
│   └── ImageDocument.swift
├── Services/
│   ├── BlurRenderer.swift
│   └── ImageIOService.swift
├── ViewModel/
│   └── EditorViewModel.swift
└── Views/
    ├── ContentView.swift
    └── EditorCanvasView.swift
```

## Architecture Rules
- Store regions in image coordinates, not view coordinates.
- Keep preview and export on the same rendering path.
- Do not change image format, pixel dimensions, or aspect ratio during editing.
- Export always writes a new file in the input format.
- New editing features should integrate with undo/redo.
- Replacing the current image must never silently discard existing edits.

## Working Rules
- Run `swift build` before finishing substantial changes.
- For rendering changes, check both preview behavior and export behavior.
- Keep canvas interaction logic in small, understandable states.
- Avoid unnecessary third-party dependencies.
- Do not silently simplify or remove existing user-facing behavior.
- Do not commit local editor files such as `.vscode/`.
- Keep public documentation aligned with actual app behavior and release scripts.

## Open Source Repo Expectations
- The repository is public and user-facing, so README and release workflows are part of the product.
- CI should at minimum validate `swift build` and release packaging.
- Tagged releases should produce an unsigned `.app.zip` artifact containing `ImageBlur.app`.
- Community-facing files in `.github/` should stay concise and practical.
- Keep the root documentation minimal. `README.md` and `AGENTS.md` are the primary maintained docs.

## Known Limitations
- Lasso shapes can be created, transformed, and rotated, but their points cannot be edited individually yet.
- Angle snapping with `Shift` is not implemented yet.
- The app is currently a single-image editor without batch processing.

## Verification
- Build:
  - `swift build`
- Run:
  - `swift run`
- Assemble app bundle:
  - `./scripts/assemble_app.sh release`
- Build release archive:
  - `./scripts/build_release.sh v0.1.0`

## Typical Next Tasks
- Point editing for lasso shapes
- Rotation angle snapping with `Shift`
- Better performance with very large images
- Additional file types and more robust metadata preservation
- Screenshot automation or polished release media

---
> Source: [Terrormixer3000/ImageBlur](https://github.com/Terrormixer3000/ImageBlur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
