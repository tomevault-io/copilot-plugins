## asciishader

> Real-time 3D ray-marched scenes rendered as ASCII art in the terminal. Includes **Chisel**, a high-level shading language that compiles to GLSL, a GPU-accelerated renderer (OpenGL), and a full TUI built on BubbleTea.

# AsciiShader

Real-time 3D ray-marched scenes rendered as ASCII art in the terminal. Includes **Chisel**, a high-level shading language that compiles to GLSL, a GPU-accelerated renderer (OpenGL), and a full TUI built on BubbleTea.

## Quick Reference

```
make build          # Build all binaries (asciishader, chisel, chisel-lsp)
make test           # Run all tests
make lint           # Run golangci-lint
make fmt            # Format all Go code (gofmt -w -s)
make fmt-check      # Check formatting without writing
make generate       # Regenerate editor tooling from lang.go
make all            # Build + test + lint (use before committing)
```

Always use the Makefile — do not run `go test`, `go vet`, `gofmt`, or `golangci-lint` directly.
Run `make all` before committing to catch formatting, lint, and test failures.

## Project Structure

```
cmd/asciishader/           Main TUI application

pkg/
  chisel/                  Chisel language — public API (Compile)
    compiler/
      token/               Token types, Position, Span
      lexer/               Hand-written tokenizer
      ast/                 AST node types, Walk, Print
      parser/              Recursive descent + Pratt
      analyzer/            Type checker, scope resolution
      codegen/             AST → GLSL
      diagnostic/          Error types, terminal renderer
    lang/                  Language registry (single source of truth)
    lang/gen/              Code generator for editor tooling
    format/                Source code formatter
    lsp/                   Language Server Protocol server
    editors/               Generated: tree-sitter, TextMate, VS Code config
    cmd/chisel/            CLI (compile, check, fmt)
    cmd/chisel-lsp/        LSP entry point
  gpu/                     OpenGL renderer — framebuffer, SDF→ASCII, ANSI colors
  shader/                  GLSL shader compilation and template assembly
  core/                    Shared types: RenderConfig, Camera, Vec3, render modes
  scene/                   Scene loader — discovers .chisel/.glsl, watches for changes
  recorder/                Records camera + parameters to .asciirec
  clip/                    Playback engine and binary .asciirec format

tui/
  views/                   Gallery, Help, Player views
  editor/                  GLSL/Chisel editor with syntax highlighting
  controls/                Interactive sliders and parameter controls
  layout/                  Sidebar, panels, resizers, animated transitions
  components/              Reusable UI components
  styles/                  Theme and styling constants

shaders/                   25 example scenes (.chisel)
docs/                      Architecture documentation
```

## Documentation

See `docs/ARCHITECTURE.md` for the system architecture, rendering pipeline, and compiler design. **Keep it up to date** when making structural changes.

Chisel language docs:
- `pkg/chisel/LANGUAGE.md` — Language reference
- `pkg/chisel/IMPLEMENTATION.md` — Compiler implementation guide
- `pkg/chisel/GRAMMAR-DESIGN.md` — Grammar single-source-of-truth design

## Key Concepts

- **Chisel** compiles to GLSL fragment shaders. Pipeline: Lexer → Parser → Analyzer → CodeGen.
- The **GPU renderer** ray-marches at sub-pixel resolution, then maps cells to ASCII characters by shape matching.
- The **TUI** is BubbleTea-based with four views: Shader (F1), Player (F2), Gallery (F3), Help (F4).
- **Language registry** (`lang/lang.go`) is the single source of truth — `go generate` produces all editor grammars.
- **Render modes**: Shapes, Blocks, Braille, Slice, Cost heatmap.

## Development Workflow

1. Edit code
2. `make all` (builds, tests, lints)
3. To test only Chisel: `make test-chisel`
4. To check all shaders compile: `make chisel-check`
5. After changing `lang.go`: `make generate`

## Shader Pipeline Comparison (WGSL Reference)

The `docs/wgsl/` directory contains WGSL shaders captured from the reference SDF editor. Use these to verify our GLSL rendering pipeline matches the editor's output.

### Capturing shaders from the editor

1. Open the browser console **before** loading the editor page
2. Paste the contents of `docs/wgsl/capture_shaders.js`
3. Load/reload the editor with the target scene
4. In the console, run: `downloadShadersAll()`
5. Save the downloaded `all_shaders.wgsl` to `docs/wgsl/`

### Comparing pipelines with shaderdiff

```
go build -o shaderdiff ./tools/shaderdiff/
./shaderdiff -wgsl docs/wgsl/all_shaders.wgsl -chisel shaders/scene.chisel
diff /tmp/ref_pipeline.glsl /tmp/our_pipeline.glsl
```

This extracts the lighting/shading functions from both the WGSL capture and our compiled GLSL, converts the WGSL to GLSL, and outputs two comparable files. Use `diff` or `vimdiff` to find discrepancies.

The tool extracts these functions from the WGSL raytrace shader (the largest one):
- `color_occ`, `multiply_opacity` — AO color preservation
- `lighting_ambient`, `lighting_directional`, `lighting_dome`, `lighting_fresnel`, `lighting_point` — all light types
- `calcSoftshadow`, `calcAO` — shadows and ambient occlusion
- `calcNormal_*` — normal computation
- `applyLighting` — main shading orchestrator
- `postProcessColor` — tone mapping, gamma, contrast
- `blend2_color`, `material_mix` — material blending at boolean operations

### Key architecture differences (WGSL vs our GLSL)

The WGSL editor is a **compute shader** with BVH acceleration, tiled rendering, and a data-driven primitive loop. Our GLSL is a **fragment shader** with compile-time SDF code generation. These structural differences are intentional — focus comparisons on the **lighting math**, not the acceleration structure.

Specifically, do NOT try to match:
- `wmap_dis_volume` / `wmap_dis_pruned_prims` — BVH acceleration (we use brute-force `sceneSDF`)
- `_all_prims_array` / tile bits — data-driven primitive dispatch (we use codegen)
- `intersectFloor` — floor rendering (we don't have floors)
- `_lightingArray` / light type dispatch loop — we use `#define` constants per light

DO match exactly:
- GGX distribution, Fresnel, geometry term formulas
- `color_occ` / `multiply_opacity`
- Dome light FmsEms multi-scatter compensation
- AO step distribution and output formula
- `postProcessColor` pipeline (gamma always, tone mapping/contrast conditional on flags)
- Material blending formulas (`blend2_color` BT_MAT / BT_ADD paths)

### Reference comparison doc

See `docs/WGSL_COMPARISON.md` for the full feature-by-feature comparison with status tracking.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pierre-borckmans) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
