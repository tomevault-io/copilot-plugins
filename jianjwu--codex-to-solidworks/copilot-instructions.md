## codex-to-solidworks

> When using the SolidWorks MCP server in this workspace:

# SolidWorks MCP Workflow

When using the SolidWorks MCP server in this workspace:

- Treat `solidworks_mcp` as a local Windows/SolidWorks COM bridge. Do not use WSL for real CAD operations.
- Use millimeters by default unless the user explicitly asks for another unit.
- For SW2022 part creation, pass the explicit part template `C:\ProgramData\SOLIDWORKS\SOLIDWORKS 2022\templates\gb_part.prtdot` when a `create_part` tool call accepts a template path.
- Convert natural-language part requests into a short ordered build plan before calling tools.
- For drawing-based modeling, first produce a dimension/feature extraction table before creating geometry: base datums, main body, axial lengths, diameters, hole counts, pitch-circle diameters, angular positions, boss directions, cut depths, tolerances, and unclear items.
- Before selecting features, classify the model as revolved, non-revolved, or hybrid. If the main body has a stable axis and can be generated from a radial section, use a revolve feature for the base instead of making the shape with many extrudes.
- If any topology-driving value is unclear, such as hole position/PCD/angle, boss center, axial step order, extrusion direction, or whether a feature is a cut or a protrusion, stop and ask for confirmation instead of guessing. Approximate only noncritical cosmetic details after marking them as assumptions.
- For shaft, flange, sleeve, and other axisymmetric parts, create the main body from a revolved section around a named center axis whenever possible. Add holes, keyways, slots, bolt circles, and local bosses only after the revolved base has been inspected.
- Define a modeling sequence from stable references: origin/axis and primary datums first, main revolved/extruded body second, secondary bosses third, cuts and hole features fourth, patterns fifth, cosmetic chamfers/fillets last.
- Use feature patterns and symmetry deliberately. For repeated holes, slots, ribs, teeth, grooves, and bosses, create one fully defined seed feature and then use circular pattern, linear pattern, mirror, or sketch pattern from stable axes/planes instead of manually duplicating geometry. Prefer feature-level patterns over sketch-level duplication when it keeps the model tree easier to edit.
- Before patterning, confirm the pattern definition: count, spacing or pitch-circle diameter, angular increment, pattern axis/plane, seed orientation, and whether geometry should merge. Inspect the patterned result immediately after creation.
- For keyways and key-seat features, identify the key type before modeling: parallel key, Woodruff key, taper key, spline/keyed bore, shaft keyseat, or hub/internal keyway. Confirm width, depth, length, end shape, fillet/chamfer, angular orientation, distance from shoulders/end faces, and whether dimensions apply to the key or to the cut feature. Use standard key/keyway proportions when the drawing references a standard, and ask before approximating unclear key geometry.
- For every boss/extrude/cut, explicitly state and verify the start plane, direction, depth/end condition, and whether it adds or removes material. After creating an axial boss or sleeve, inspect an isometric or section-like preview to catch reverse-direction features.
- Execute CAD work in small steps: create/open model, create sketch, add constrained geometry, create feature, inspect state, then continue.
- After every mutating CAD tool call, read back model state with tools such as `get_model_info`, `list_features`, `get_mass_properties`, or `export_image`.
- Save generated CAD artifacts under `outputs/solidworks/` unless the user gives another path, using `scripts/`, `models/`, `drawings/`, `images/`, and `exports/` subfolders to keep outputs searchable.
- Before rerunning a modeling or drawing-generation command, delete stale generated artifacts for the same model prefix under `outputs/solidworks/` and its subfolders, including old `.SLDPRT`, `.SLDDRW`, PDF/DWG/image exports, preview files, and SolidWorks `~$` lock files when they are no longer active.
- Prefer parametric feature creation over one-shot macro execution. Use macros only when a repeated operation is too awkward through direct tools.
- Stop and report the exact tool error if SolidWorks is not running, COM is unavailable, a sketch is underdefined in a way that blocks modeling, or a rebuild fails.

---
> Source: [jianjwu/codex_to_solidworks](https://github.com/jianjwu/codex_to_solidworks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-17 -->
