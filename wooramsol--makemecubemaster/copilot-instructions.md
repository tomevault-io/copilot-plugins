## selfie-camera

> Front/selfie camera assumptions for vision and UI


# Selfie camera only

This app uses the **front camera** with a **mirrored video preview** (`scaleX(-1)` on `.camera-feed`).

Vision pipeline rules:

1. Draw video with `drawCameraFrame()` — never flip the canvas.
2. Store **raw** sensor-order colors from detection.
3. Live scan overlay (`DetectionOverlay`): `FaceColorGrid` with `orientation="mirror"`.
4. Scanned result bar (`FaceGridMini`): `orientation="real"` (physical world, no mirror).
5. Only capture when `identifyFaceFromCenter()` identifies the center sticker.
6. Solver orientation search handles **rotation only** on stored raw colors (no mirror — mirrors are not physical).

---
> Source: [wooramsol/makemecubemaster](https://github.com/wooramsol/makemecubemaster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
