## mvg

> MVG's source is authored for people and coding agents. This guide is the

# AGENTS.md

MVG's source is authored for people and coding agents. This guide is the
operator manual: where to find the source, how to make a safe change, and how
to turn that source into inspected render and atlas output. Read the README for
the public overview; do not treat generated files or this guide as product
source.

## Project Shape

MVG is an Odin-first asset format and renderer for small UI vector assets.
The source of truth is `.mvg`, not SVG and not generated JSON.

Main folders:

- `model/`: core MVG model, lexer, parser, validator, formatter, JSON dump,
  and bounds logic.
- `model/renderer/`: scalar renderer backend, path/stroke/raster/blend code,
  STB PNG writing, and STB truetype text-to-path support.
- `cli/`: thin command wrapper over `model` and `model/renderer`.
- `examples/one-shots/`: isolated component studies with their own theme.
- `examples/material-dark/`: a cohesive Material Dark UI sample, including
  reusable controls.
- `examples/material-light/`: the Material component set rendered with a light
  semantic theme.
- `examples/night-poster/`: the same component contract restyled as a
  hard-edged neo-brutalist event UI.
- `assets/fonts/`: the four IBM Plex fonts required by MVG's bundled examples.
- `showcase/`: curated public renders; ordinary generated output remains in
  ignored `target/` directories.

## Agent Workflow

Use an example directory as a self-contained project. Its `theme.mvg` supplies
colors and fonts to the component files beneath it. Author `.mvg` only:
`*.mvg.json`, PNG renders, atlas images, and layouts are generated diagnostic
or delivery output, never hand-edited source.

For a focused visual iteration:

```sh
make build
target/bin/mvg fmt examples/night-poster --check
target/bin/mvg validate examples/night-poster
target/bin/mvg inspect examples/night-poster
target/bin/mvg render examples/night-poster --asset card --out target/review/card.png
```

Open and assess the rendered PNG after every meaningful visual change. Render
the default asset and each affected variant; a valid file can still have poor
contrast, clipped effects, or unintended geometry. Use `render-all` to review
the full set, then `pack` to confirm atlas output once the component set is
ready:

```sh
target/bin/mvg render-all examples/night-poster --out target/review/night-poster
target/bin/mvg pack examples/night-poster --out target/pack/night-poster --debug-overlay
```

Keep temporary review output under ignored `target/`. Only add a PNG to
`showcase/` when it is deliberately curated for the repository's public
presentation. `--debug-overlay` writes a separate atlas companion showing
canvas, stretch-body, content, and named-slice bounds; it never changes the
runtime atlas.

## Verification Commands

Use these from the repo root:

```sh
make build
make check
make test
make smoke
```

Windows users can run the equivalent root-level `build.bat`, `check.bat`,
`test.bat`, and `smoke.bat` wrappers. They require `odin` on `PATH` and do not
require Make, Bash, or PowerShell.

`make smoke` is the current end-to-end check. It builds the CLI, validates
examples, emits JSON snapshots, inspects the project, renders every example
PNG, packs the examples into atlas/layout output, and checks formatting.

Run `make check` for any code or source edit. Run `make test` and `make smoke`
before handing off parser, renderer, CLI, or example changes. `inspect` can
report visual bounds beyond a fixed source canvas; investigate new warnings and
use a larger canvas plus `body`/`content` metadata when an asset needs
permanent effect space.

## Current CLI

Implemented:

- `mvg parse <file-or-directory> [--json-out <file>]`
- `mvg validate <file-or-directory>`
- `mvg fmt <file-or-directory> [--check] [--out <file>]`
- `mvg inspect <file-or-directory>`
- `mvg render <file-or-directory> [--asset <id>] [--variant <name>] [--out <file.png>]`
- `mvg render-all <file-or-directory> --out <directory>`
- `mvg pack <file-or-directory> --out <directory> [--debug-overlay]`

Not implemented:

- SVG output; SVG import is intentionally out of scope.
- Preview/editor UI.
- Groups, transforms, and richer shared-fragment reuse.

## MVG Source Conventions

Use a shared `theme.mvg` file for colors and fonts. Each example directory is
an independent project, so its theme applies to the assets below it:

```mvg
colors light

palette {
  surface = $slate2
  button_top = $blue9
  button_text = $gray1
}

fonts {
  sans = "assets/fonts/IBMPlexSans-Regular.ttf"
  sans_bold = "assets/fonts/IBMPlexSans-Bold.ttf"
}
```

Product themes should expose semantic palette names:

```mvg
text label "Create" 46 30 size 18 font @sans_bold {
  fill @button_text
}
```

Built-in standard colors are referenced with `$name`, for example `$slate2`,
`$blue9`, or `$redA5`. The `A` scales are alpha colors. Use `colors light` or
`colors dark` to select the built-in table. Project-defined palette colors and
font roles are referenced with `@name`. Bare palette and font references are
invalid source syntax.

Prefer direct `$` standard colors for isolated examples. Define a palette only
when a cohesive project benefits from semantic product tokens, as in
`examples/material-dark/`. Avoid per-component raw colors or direct font paths
unless there is a concrete reason. If `font` is omitted, text expects a shared
font named `sans`.

## Format Features

Implemented source features include shared `palette` and `fonts` blocks, asset
metadata (`type`, `body`, `content`, `outsets`, `padding`, `slice`), nodes
(`rect`, `circle`, `ellipse`, `line`, `path`, `text`), solid/linear/radial
fills, strokes, rectangular clips, and text alignment.

Assets can include variants that patch the default asset and lower to ordinary
export assets named `<asset>__<variant>`:

```mvg
variant hover {
  add rect focus_ring 1 1 158 46 radius 13 {
    fill none
    stroke $gray1 0.3 width 1
  }

  node base {
    fill @button_hover
    shadow 0 5 10 @button_shadow 0.32
  }
}
```

Variant patches target existing node IDs and can replace fill, stroke, clip,
and the effect list. Use `effects none` inside a node patch to clear inherited
effects. Variants can also `hide <node_id>` or `add <node-kind> ...` nodes.
Metadata such as `body`, `content`, `slice`, `outsets`, and `padding` is
inherited unless overridden inside the variant.

Text supports:

```mvg
text label "Create" 46 30 size 18 font @sans_bold align center valign center
text label "Create" in 12 8 96 28 size 18 font @sans_bold
```

Strokes support optional caps and joins:

```mvg
stroke @accent 1 width 2 cap round join round
```

## Effects

MVG has two dedicated skeuomorphic relief effects for soft UI surfaces:

```mvg
relief_shadow <distance> <blur> <light_color> <light_opacity> <dark_color> <dark_opacity>
inset_relief <distance> <blur> <light_color> <light_opacity> <dark_color> <dark_opacity>
```

Use `relief_shadow` for raised controls. It renders a soft light shadow toward
the upper-left and a darker shadow toward the lower-right. Rounded rectangles
use an analytic renderer with diagonal light/dark ownership and a smooth merge
band so the dark side does not creep over the light top-left edge.

Use `inset_relief` for pressed or recessed controls. It is the inward
counterpart: dark inner relief on the upper-left, light inner relief on the
lower-right, clipped inside the source geometry. It is intended for active
buttons, pressed wells, and input fields.

Both effects accept `@palette_name`, `$standard_color`, or hex color references
for their light/dark colors and use normal `0..1` opacity validation. For
isolated examples, prefer a direct standard-color pairing such as `$gray1` on
the light side and `$bronze9` on the dark side.

Other rendered effects:

```mvg
shadow dx dy blur color opacity
inner_shadow dx dy blur color opacity
glow blur color opacity
sheen inset height color opacity
```

`shadow`, `glow`, and `inner_shadow` render with a simple scalar box blur. The
semantics are renderer-native, not Gaussian. `sheen` is experimental and should
stay subtle unless the syntax is deliberately promoted.

## Boundaries and Implementation Notes

The renderer is intentionally simple and scalar. Keep changes small and
testable; profile before considering SIMD or architectural complexity.

Important decisions:

- Do not add SIMD unless profiling proves it matters.
- Do not add a custom PNG writer. PNG output uses Odin `vendor:stb/image`.
- Text uses Odin `vendor:stb/truetype` and lowers glyph outlines to paths.
- `.mvg.json` is a debug/model snapshot, not an editable source or normal
  in-process bridge.

Known limitations:

- Text bounds in `model/bounds.odin` are approximate.
- SVG path data supports common path commands and arcs, but this is not a full
  SVG engine.
- Atlas packing uses deterministic skyline packing with one logical placement
  table shared across 1x/2x/3x/4x output.
- The packer optimizes a single atlas. Add explicit multi-atlas/page support if
  the project grows beyond a practical maximum size.
- Normal `render` and `render-all` export the asset canvas. `pack` can add
  transparent `--export-padding`, but assets that permanently need visible
  shadow/glow space should use a larger source canvas and logical
  `body`/`content` metadata.
- Generic closed-path strokes emit separate inner and outer contours so rounded
  rect strokes close cleanly. If future stroke changes reintroduce seams at
  closed-path joins, check `model/renderer/stroke.odin`.
- Groups, transforms, SVG output, preview/inspector UI, and richer
  shared-fragment reuse are not implemented.

## Change Boundaries

Keep edits scoped. Preserve the Odin-first model/CLI split:

- Core semantics belong in `model/`.
- Rendering details belong in `model/renderer/`.
- CLI should remain a thin wrapper.
- Do not duplicate parser or validation behavior in future UI code.

When adding assets, start with an existing component whose canvas, metadata,
and variants resemble the intended runtime role. Keep runtime inputs (such as
text fields) content-free unless the asset's purpose is explicitly a static
showcase. Use semantic palette roles for cohesive themes and direct Radix `$`
colors only for small isolated studies.

---
> Source: [Skytrias/mvg](https://github.com/Skytrias/mvg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
