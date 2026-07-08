## locus-editor

> A greenfield SVG/vector graphics editor in Rust — an Inkscape replacement built from first principles.

# Locus

A greenfield SVG/vector graphics editor in Rust — an Inkscape replacement built from first principles.

## Architecture

- **wgpu** for GPU rendering (custom pipeline, no framework crates)
- **egui** for UI panels (via egui-wgpu + egui-winit)
- **Custom renderer** — only pure-math deps allowed (lyon for tessellation, etc.)

### Workspace layout

```
bin/locus-editor/     Main GUI binary (winit + wgpu + egui app loop)
crates/
  locus-geom/         Point, Vec2, Affine, Bounds, Segment, Path, SubPath, Color (linear RGBA)
  locus-scene/        Node tree (SlotMap-based), Style, Paint, PaintRef, stable IDs
  locus-svg/          SVG import (usvg) / export (stub)
  locus-text/         Text shaping (rustybuzz + ttf-parser) — stub
  locus-tess/         Path tessellation via lyon (fill + stroke)
  locus-render/       wgpu pipeline, shader, ortho projection, scene rendering
  locus-ops/          Undo/redo command history
  locus-tools/        Editing tools (select, pen, etc.) — stub
```

### Key design decisions

- **All four SVG segment types** (Line, Quad, Cubic, Arc) preserved — no normalization to cubics — for round-trip fidelity.
- **Unified node tree** — paints (gradients, patterns) are nodes too; a "defs" subtree holds reusable definitions. Cross-references use SlotMap IDs.
- **Transforms on groups** — stored per-node, cached resolved world transform. Never baked into path coords.
- **Linear RGBA internally** — gradient interpolation in linear space (not sRGB).
- **Text as text** — stored as font + characters in scene graph, only expanded to paths for rendering.
- **Plain floats internally** — optional "origin tag" for values from percentages/em for faithful re-serialization. Units (mm, px) are document-level metadata.

## Build & Run

```sh
cargo run -p locus-editor          # launch the editor
cargo build                         # build all crates
cargo test --workspace              # run all tests
cargo clippy --workspace            # lint
```

## Rust edition

Edition 2024 (rust-version 1.92). Note: edition 2024 changes `ref` pattern defaults — match ergonomics differ from 2021.

---
> Source: [serialexp/locus-editor](https://github.com/serialexp/locus-editor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
