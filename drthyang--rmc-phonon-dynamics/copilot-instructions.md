## rmc-phonon-dynamics

> All UI, feature, and design work happens in **`web/`** — a React + Vite +

# Repository guide (read first)

## The current app is `web/`

All UI, feature, and design work happens in **`web/`** — a React + Vite +
WebGPU app, static / GitHub-Pages hostable. This is the migrated, validated
version of the project. See `web/FEATURE_PARITY_REPORT.md` for what it does.

- Dev:    `cd web && npm install && npm run dev`
- Tests:  `cd web && npm run validate`   (deterministic science/UI-mapping checks)
- Build:  `cd web && npm run build`       (→ `web/dist`, static)

## Legacy — reference only, do NOT edit

These are the original Python runner + phonon viewer, kept for reference and
numerical comparison. **Do not modify them when working on the app:**

- `src/`, `src_gpu/`  — legacy CPU / JAX-GPU phonon code
- `rmcph_gui/`        — legacy Python/JS runner GUI
- `viz/`              — legacy phonon viewer (phononwebsite-derived)
- `archive/`          — old experiments

## Notes for design / UI work

- Edit only `web/src/**`. The compute/IO/math layers (`web/src/compute`,
  `web/src/io`, `web/src/math`) are UI-agnostic and tested — treat them as a
  stable API and avoid changing their behavior; reshape `pages/` + `components/`.
- `data/` holds large RMC datasets (gitignored).

---
> Source: [drthyang/rmc-phonon-dynamics](https://github.com/drthyang/rmc-phonon-dynamics) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-16 -->
