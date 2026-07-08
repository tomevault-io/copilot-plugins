## scripts

> scripts repo for rhino 8 / grasshopper automation at raider boats.

# CLAUDE.md -- raider-scripts

scripts repo for rhino 8 / grasshopper automation at raider boats.
aluminum boat design, fabrication, and shop workflow tooling.

## what this repo is

standalone rhino python scripts and grasshopper C# script components
used in parametric aluminum boat hull design and CNC fabrication.
scripts are run via `_-RunPythonScript`, rhino aliases, or embedded
in grasshopper C# script components.

MIT licensed. no third-party grasshopper plugin dependencies.

## repo structure

```
/rhino-python/      standalone rhino python scripts (.py)
/grasshopper-cs/    grasshopper C# script components (.cs)
/CLAUDE.md          you are here
/LICENSE            MIT
```

## environment

- rhino 8, windows
- grasshopper: vanilla only (no food4rhino plugins in scripts)
- python: rhino's IronPython 2 (legacy) or CPython 3 (`#! python3` shebang)
  - prefer python 3 for new scripts unless IronPython is required
  - python 3 scripts MUST start with `#! python3`
- C#: .NET framework via grasshopper's C# script component
- units: inches, always
- tolerance: document tolerance, typically 0.001" for modeling, 1/16"-1/8" for fab

## supply chain security -- MANDATORY

these rules are non-negotiable. violations can introduce malicious code.

### before installing ANY dependency:

1. verify the package EXISTS and is what you think it is
   - check npmjs.com directly
   - confirm: correct name spelling, real maintainer, not a typosquat
2. check publish date: `npm view <pkg> time`
   - if the package OR the specific version is less than 7 days old, STOP
   - flag it to the user and wait for explicit approval
3. check download count
   - under 100 weekly downloads on npm = suspicious until proven otherwise
4. prefer well-known, high-download packages over obscure alternatives
5. NEVER invent or hallucinate a package name. if you're not 100% certain
   a package exists with that exact name, say so and let the user verify.

### npm hardening (enforce in all npm operations):

- always use `--ignore-scripts` on install unless user explicitly approves scripts
- always commit package-lock.json
- run `npm audit` after any dependency change
- pin exact versions in package.json (no ^ or ~ prefixes)
- prefer `dependencies` vs `devDependencies` correctly

### general dep philosophy:

- MINIMAL dependencies. every dep is attack surface.
- if something can be done in <50 lines of vanilla js, do that instead of adding a dep
- before suggesting a new dep, ask: does this NEED to be a package?
- vendor small utilities rather than importing them
- no meta-frameworks, no orms, no "convenience" layers

### if unsure about a package:

say "i'm not 100% sure this package exists / is safe -- please verify before installing"
-- this is ALWAYS preferable to confidently recommending something wrong.


## conventions

### python scripts

- use `rhinoscriptsyntax as rs` for UI/selection, `Rhino.Geometry` for math
- use `scriptcontext as sc` for doc access
- entry point pattern: define a main function, call it at bottom:
  ```python
  def my_command():
      # ...
      pass

  if __name__ == "__main__":
      my_command()
  ```
- error handling: fail loudly with `print()` messages. no silent failures.
- user interaction: `rs.GetObject`, `rs.ListBox`, `rs.MessageBox` for simple UI.
  `Rhino.Input.Custom.GetPoint` / `GetObject` for anything needing dynamic draw.
- for scripts with dynamic preview (like dim-3d), subclass `GetPoint` and
  override `OnDynamicDraw`

### grasshopper C# components

- file extension: `.cs`
- follows GH_ScriptInstance pattern with RunScript method
- type hints and output variable names must match component setup
- inputs/outputs documented in the summary comment block
- use `Rhino.Geometry.Unroller`, `Brep.CreateDevelopableLoft`, etc. directly
- NEVER use meshes as geometry output. surfaces/polysurfaces only.
  zero-thickness surfaces OK for intermediary ops if thickness is
  accounted for downstream.

### naming

- filenames: `kebab-case.py` or `kebab-case.cs`
- functions: `snake_case`
- classes: `PascalCase`
- constants: `UPPER_SNAKE`

### general

- geometry is always surfaces/polysurfaces, never meshes
- end state for parts: closed polysurfaces with material thickness
- data output format: CSV when exporting tabular data
- layer names follow numbered convention (01-14), see layer table below
- favor generalizable, reusable solutions over one-off hacks
- comprehensive fixes over incremental patches

## layer conventions

scripts that create/reference layers must use these names exactly:

```
01 - Default
02 - Reference        profiles, grids, scale figures
03 - Bake             GH output geometry
04 - Booleans         intersection geo, used booleans
05 - Clipping         layouts/drawings
06 - Annotations
07 - Jig              steel-framed build jig
08 - Static parts     3D model by assembly (Hull bottoms, Hardtop, etc.)
09 - Ink lines        bend marks per part
10 - Fab 3D           in-situ vs flat plate states
11 - 2D geo           flat patterns, sublayers: Inside cut / Outside cut / Mark
12 - Cutfiles
13 - Title block
14 - zArchive
```

sublayer separator is `::` (e.g., `11 - 2D geo::Inside cut`).
sublayer colors for 2D geo: magenta (inside cut), dark green (mark), blue (outside cut).
dark green is rhino's canonical `RGB(0, 127, 0)` / `#007F00` -- used for both the
`Mark` sublayers and the `09 - Ink lines` layer. in scripts use
`System.Drawing.Color.FromArgb(0, 127, 0)`, NOT `Color.DarkGreen` (which is `0, 100, 0`,
a different, darker green).

## domain context

this is for raider boats (raiderboats.com). high-end welded aluminum fishing
boats for PNW offshore. models range 21'-30'.

fabrication pipeline: parametric rhino/GH model -> flat patterns -> DXF ->
RhinoCAM -> G-code -> Multicam 5000 CNC router (5'x10' bed).

materials: aluminum sheet 0.125"-0.250" (occasional 0.375").
alloys: 5052-H32 general, 5086-H116 for hull bottoms/CVK.
press brake limit: 0.190" x 13'.
bend allowance: k-factor method, 0.40-0.45 typical for air-bent aluminum.

key geometric concepts:
- developable surfaces (aluminum bends, doesn't stretch)
- DevLoft / CreateDevelopableLoft for hull panels that capture real forming behavior
- ruling lines matter for accurate t-frame cross-sections
- unrolling surfaces to flat patterns is a core operation
- chine, sheer, CVK (center vertical keel) are primary hull curves

## writing new scripts

1. check if similar functionality exists in the repo first
2. include a docstring at the top explaining purpose and usage
3. include alias suggestion if it's a standalone script:
   `alias: my-script -> _-RunPythonScript "path/to/my-script.py"`
4. test with real geometry from the boat models, not toy geometry
5. handle edge cases: empty selections, single-face breps, coincident points, etc.

## things to watch out for

- IronPython 2 vs CPython 3: f-strings only work in python 3 scripts.
  use `.format()` for python 2 compatibility or add the `#! python3` shebang.
- `rs.ObjectType()` returns different values than `ObjectType` enum -- don't mix
- `sc.doc` vs `ghdoc`: in grasshopper scripts, `sc.doc` is the rhino doc.
  set `sc.doc = ghdoc` only if you need to operate on GH geometry.
- brep face normals: in CPython 3, `BrepFace.NormalAt` may ALREADY account
  for `OrientationIsReversed`. manually flipping on that flag can double-reverse
  the normal. safest approach: get raw `NormalAt`, then use a geometric check
  (e.g. partner face centroid direction) to determine the correct sign.
- `sc.sticky` persists across script runs in the same rhino session.
  use it for toggle state, cached values, etc. keys should be namespaced.
- when working with layers, always check `rs.IsLayer()` before creating
- System.Drawing.Color, not python tuples, for layer colors
- `BrepFace.DuplicateFace(False)`: creates a standalone Brep from a face.
  use this when `AreaMassProperties.Compute` or `ClosestPoint` on a BrepFace
  gives wrong results — the Brep overloads are more reliable in CPython 3.
- `rs.LayerIndex()` does not exist. use `sc.doc.Layers.FindByFullPath(name, -1)`
  for integer layer table index.
- `Brep.CreateOffsetBrep` on a polysurface adds fillet faces at bends.
  for sharp corners, build offset geometry from plane math instead.
- `Intersection.PlanePlane` returns `(bool, Line)` — an infinite Line.
  `Line.ClosestParameter` returns a double, not a tuple.
- joined polysurface face normals are NOT consistently oriented.
  use `abs(dot)` when computing angles between faces in a joined brep.
- unfold-to-2d coordinate spaces: the BFS unroll produces geometry in the
  seed face's plane (flat space). ALL output (outside/inside cuts, bend lines,
  ink curves, labels) goes directly to sublayers in flat space. do NOT add a
  flat-to-3D alignment transform -- a previous `align_xform` (PlaneToPlane to
  3D NAS space) introduced sub-tolerance errors that compounded into visible
  bend line misplacement and was removed. if a future feature needs 3D-
  positioned output, recompute the transform fresh from the geometry.
- `TextEntity.CreateCurves` text-facing direction is NOT consistent relative
  to the text plane Normal. for some plane orientations text reads from
  +Normal side, for others from -Normal side. workaround: create text on
  `Plane.WorldXY` (known: readable from +Z), then `PlaneToPlane` transform
  to the target plane. this guarantees text faces the target plane's +Normal.
- `Curve.CreateBooleanUnion` requires overlapping closed curves. if bend
  deduction shifts push faces apart (wrong shift direction), the union fails
  silently or drops faces. use BFS-iterative pairwise union instead of
  all-at-once union for robustness.
- `AreaMassProperties.Compute` centroid on a face with holes (inner loops)
  shifts laterally from the centroid of the equivalent face without holes.
  don't use centroid position as a proxy for face normal direction — the
  lateral shift can overpower the normal-direction offset (t/2) and flip
  the sign of geometric checks.
- unfold-to-2d bend direction detection: the only reliable method is
  `fold_z` — the perpendicular distance of the neighbor centroid from the
  flat plane BEFORE the flatten rotation (transformed by current_xform
  only, not best_combined). cross-product tests, centroid averaging, and
  rotation angle sign interpretation all fail due to edge tangent direction
  ambiguity and face ordering dependencies. `Rotation(-A, -dir)` =
  `Rotation(A, dir)`, so the BFS unfolding works without normalizing
  axis_dir, but extracting direction FROM the angle is not possible.
- unfold-to-2d bend angle: use `180 - abs(chosen_angle normalized to
  [-180, 180])` from the BFS. the `abs(dot)` formula in unroll_by_rotation
  can't distinguish acute (e.g. 70 deg) from obtuse (110 deg) bends
  because abs() destroys the sign. the BFS chosen_angle encodes the exact
  rotation and gives correct angles for all cases.
- `brep.IsPointInside` for normal orientation on NAS faces: unreliable
  for thin parts. NAS centroids are t/2 from both surfaces. offsetting by
  t in the normal direction overshoots through the part (t/2 past the far
  surface), landing outside on the wrong side. IsPointInside returns False
  for both directions, making orientation impossible.
- git: large files (.3dm models) in ANY commit in history will block push
  to github, even if deleted later. use `git filter-repo` or BFG to purge
  from history. adding to .gitignore only prevents future commits.
- doc-wide object scans miss off-layer geometry by default. both
  `for o in sc.doc.Objects` and `rs.AllObjects()` use `ObjectEnumeratorSettings`
  with `HiddenObjects=False`, which silently SKIPS objects on layers that are
  turned off (also individually-hidden objects, and references). any script
  that must see EVERY object -- luid/id uniqueness checks, attribute harvests,
  counts -- has to scan with an explicit permissive enumerator or it WILL miss
  tagged geometry on off layers. this caused the stamp-luid duplicate bug: a
  part on an off layer kept its luid out of the used-set, so the stamper
  reissued it. pattern:
  ```python
  s = Rhino.DocObjects.ObjectEnumeratorSettings()
  s.NormalObjects = True
  s.LockedObjects = True       # incl. objects on locked layers
  s.HiddenObjects = True       # incl. hidden objects AND objects on OFF layers
  s.ReferenceObjects = True    # incl. linked / worksession geometry
  for o in sc.doc.Objects.GetObjectList(s):
      ...
  ```
  `HiddenObjects=True` is the key one -- it covers both hidden objects and
  off-layer objects. ref: stamp-luid.py `_all_doc_objects`, select-by-user-text.py
  `build_key_val_map`. note this means a SELECT tool then surfaces objects the
  user can't see/grab; gate it behind a toggle and report what couldn't be
  selected (cf su's include-hidden checkbox).

---
> Source: [travw/scripts](https://github.com/travw/scripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
