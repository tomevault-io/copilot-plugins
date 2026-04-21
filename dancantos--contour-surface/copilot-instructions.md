## contour-surface

> This is a single-file browser tool (`index.html`) for reconstructing a 3D surface from hand-drawn contour lines (isolines). It is part of an **optical flat trainer** — a teaching/practice tool for understanding surface topology as revealed by optical flat interference patterns (Newton's rings / fringe contours).

# CLAUDE.md

## Project purpose

This is a single-file browser tool (`index.html`) for reconstructing a 3D surface from hand-drawn contour lines (isolines). It is part of an **optical flat trainer** — a teaching/practice tool for understanding surface topology as revealed by optical flat interference patterns (Newton's rings / fringe contours).

The user draws contour lines at specified elevation values, and the tool interpolates a plausible surface that would generate those contours, then renders it as an interactive 3D mesh.

## Architecture

Everything lives in `index.html` — no build step, no dependencies, no server required. Open it directly in a browser.

### Key subsystems

- **Drawing** — click-to-place polyline contours on a 2D canvas, each assigned an elevation (0–100)
- **RBF interpolation** — Thin-Plate Spline (`φ(r) = r² log r`) with a linear polynomial tail, solved via Gaussian elimination with partial pivoting. Capped at 120 sample points to keep the matrix tractable.
- **3D renderer** — painter's algorithm (back-to-front sorted quads), orthographic-style projection with spherical camera (theta/phi), diffuse lighting, terrain colour palette.
- **Clip shapes** — the rendered surface can be masked to a centered circle or square, matching the shape drawn on the draw canvas as a dashed guide.

### Coordinate system

Contour points are normalised to [0,1]² by dividing x/W and y/H. The 3D projection corrects for canvas aspect ratio using `max(drawW, drawH)` as the normalisation constant, so circles in the draw canvas remain circles in the 3D view.

## Things to keep in mind

- `surfaceData` is set to `null` whenever contours change, forcing a fresh reconstruction before the next render.
- `isReconstructing` flag prevents mutual recursion between `reconstruct()` and `switchTab()`.
- The Z-scale slider live-updates the render without reconstruction. Grid and Smooth sliders require a fresh reconstruct.
- Clip shape changes live-update the render without reconstruction.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dancantos) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
