## cads

> OpenSCAD designs published as a browsable gallery + in-browser editor on

# CADS - Computer-Aided Design Storage

OpenSCAD designs published as a browsable gallery + in-browser editor on
GitHub Pages.

## Layout

- `designs/*.scad` - the OpenSCAD design files (one per design)
- `macros/shapes.scad` - shared modules (`use <../macros/shapes.scad>`)
- `docs/` - the static web app deployed to Pages (gallery, editor,
  OpenSCAD WASM render worker, vendored three.js/CodeMirror bundles)
- `scripts/` - manifest/STL generation (CI), vendor bundle build, and
  design audit tooling

## Working on designs

**Read `SCAD_AUTHORING.md` before creating or modifying any `.scad` file.**
It documents the repo conventions (frontmatter, `_display_mode`, customizer
comments) and the hard-won rules for producing manifold, functional,
printable models — plus the verification workflow.

Verify with:

```bash
./scripts/audit_designs.sh [design]   # render every mode + mesh validation
node scripts/check_stl.js file.stl    # watertight / bodies / volume
```

When a design misbehaves (render errors/warnings, non-manifold meshes,
wrong body counts, parts that don't fit, timeouts), invoke the
**`/scad-debug` skill** — it walks the full diagnostic procedure
(symptom classification, mesh forensics, intersection/pull/sweep tests).

Don't claim a design change works without rendering it (`openscad` CLI in
CI/dev environments, or the web editor).

## Working on the web app

- `docs/` is served statically; CI (`.github/workflows/pages.yml`)
  regenerates `docs/designs.json` and `docs/models/*.stl` from `designs/`
  on every push to main, then deploys.
- Third-party JS is vendored in `docs/vendor/` (no CDN);
  rebuild with `scripts/build_vendor.sh`.
- The OpenSCAD WASM instance can only render once per instantiation
  (`callMain` exits the runtime) - `docs/js/render-worker.js` handles the
  instance-per-render lifecycle off the main thread.

---
> Source: [tyevco/cads](https://github.com/tyevco/cads) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
