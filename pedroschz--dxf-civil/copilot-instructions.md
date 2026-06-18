## dxf-civil

> Inspect, analyze, validate, and modify civil-engineering DXF (and DXF-converted DWG) drawings using ezdxf. Use this skill whenever the user mentions a .dxf or .dwg file, AutoCAD drawings, site plans, plats, road/drainage/utility drawings, layer cleanup, quantity takeoffs, title-block updates, CAD standards checks, or any task that involves reading or writing CAD drawings. Always reach for this skill before trying to read a DXF with cat/grep/head or any text tool — DXF files are too big and too structured for that to work.


# DXF Civil Engineering Skill

A workflow harness for working with civil-engineering CAD drawings using `ezdxf` and progressive disclosure. Optimized for big files (10MB+) where dumping raw contents into context is hopeless.

## The Golden Rules

These rules exist because violating them wastes context and produces bad output. Follow them.

1. **Never read DXF files directly.** Do not `cat`, `head`, `grep`, or `view` a `.dxf` file. They are large, structured group-code text — unreadable to humans and a waste of context for you. Always go through the scripts in this skill.

2. **Always start with `summary.py`.** Before any other action on a drawing, run `python scripts/summary.py <file.dxf>` to get the JSON map of the drawing. Everything else is a drill-down from that summary.

3. **Render before reasoning about geometry.** If a task involves spatial relationships ("is the inlet near the curb?", "do these polylines connect?"), run `render.py` first and *look at the image*. Coordinates lie; pictures don't.

4. **Never modify files in `input/`.** All writes go to `output/`. If the user hasn't set up those folders, ask before writing anywhere near the source file. When in doubt, save to a `.modified.dxf` sibling.

5. **Use `ezdxf.recover` for messy files.** Real-world DXFs (especially from non-Autodesk tools or old exports) are often slightly malformed. The scripts here use `recover.readfile()` by default, which handles minor corruption. If you write your own one-off scripts, do the same.

6. **For files >100MB, stream don't load.** Use `iterdxf` patterns (see `scripts/_lib.py`) instead of loading the whole document into memory.

7. **Check `standards/layers.yml` before creating or renaming layers.** This file encodes the office's CAD standard. New layers must conform. If the user's drawing uses a different convention, ask before "normalizing" it.

## The Workflow

For any task, follow this loop:

```
1. summary.py          → get the map
2. render.py           → see the drawing (if spatial)
3. drill down with:    layer.py / query.py / spatial.py
4. (if modifying)      write your script, dry-run, then apply
5. validate.py         → confirm output is sound
```

Do not skip steps 1 and 2 to "save time" — they cost a few seconds and save many minutes of guessing.

## When To Use Each Script

All scripts live in `scripts/` and are runnable as `python scripts/NAME.py --help`.

### `summary.py` — the always-first call

Returns a JSON snapshot: DXF version, units, drawing extents, layouts, xrefs, every layer (color, linetype, entity counts, per-layer bounding box), block definitions and insertion counts, a sample of text labels, and audit results.

Output is JSON to stdout. Pipe to `jq` or read it back as a dict. Keep the JSON if you'll reference it again — re-running is cheap but not free on huge files.

```bash
python scripts/summary.py drawings/site.dxf > /tmp/site-summary.json
```

### `render.py` — see the drawing

Renders the modelspace (or a specific paperspace layout) to a PNG. Supports layer filtering, bbox cropping, and DPI control. Use it liberally — once per spatial reasoning step.

```bash
# Whole modelspace
python scripts/render.py drawings/site.dxf --out preview.png

# Just storm and sanitary layers, high-res
python scripts/render.py drawings/site.dxf --layers "C-STORM-*,C-SAN-*" --dpi 300 --out storm.png

# A specific paperspace layout
python scripts/render.py drawings/site.dxf --layout "Sheet C-101" --out c101.png

# Crop to a bbox
python scripts/render.py drawings/site.dxf --bbox 1000,2000,1500,2500 --out detail.png
```

After rendering, view the PNG (you have vision; use it). Then plan.

### `layer.py` — drill into one layer

Lists entities on a specific layer with their attributes and key geometry. Use after the summary tells you which layer is interesting.

```bash
python scripts/layer.py drawings/site.dxf --layer C-STORM-PIPE
python scripts/layer.py drawings/site.dxf --layer C-STORM-PIPE --type LWPOLYLINE
```

### `query.py` — find text

Searches MTEXT, TEXT, attributes, and dimension overrides for a regex pattern. Use for finding callouts, station numbers, structure IDs, title block fields.

```bash
python scripts/query.py drawings/site.dxf --pattern "MH-\d+"
python scripts/query.py drawings/site.dxf --pattern "STA \d+\+\d+" --layer "*-TEXT"
```

### `spatial.py` — find entities by location

Returns entities near a point (radius) or inside a bounding box.

```bash
python scripts/spatial.py drawings/site.dxf --near 1234.5,6789.0 --radius 50
python scripts/spatial.py drawings/site.dxf --bbox 1000,2000,1500,2500
```

### `validate.py` — audit + standards check

Runs `ezdxf` structural audit and checks layers against `standards/layers.yml`. Reports unrecoverable errors, malformed entities, and layers that don't conform to the office standard.

```bash
python scripts/validate.py drawings/site.dxf
python scripts/validate.py drawings/site.dxf --fix --out drawings/site.fixed.dxf
```

### `takeoff.py` — quantity takeoff

Sums polyline lengths, hatch areas, and block insertion counts grouped by layer. Use for preliminary quantity estimates — **not** for billable takeoffs without engineer review.

```bash
python scripts/takeoff.py drawings/site.dxf
python scripts/takeoff.py drawings/site.dxf --layers "C-ROAD-*,C-CURB-*" --units feet
```

## Writing Your Own Modifications

When the user asks to modify a drawing, write a focused script rather than chaining one-shot commands. Put it in `scripts/oneoff/` (create the folder), give it a clear name, and use this template:

```python
import ezdxf
from ezdxf import recover

INPUT = "input/site.dxf"
OUTPUT = "output/site.modified.dxf"

doc, auditor = recover.readfile(INPUT)
if auditor.has_errors:
    auditor.print_error_report()
    # decide whether to bail or continue

msp = doc.modelspace()

# ... your changes here ...

doc.saveas(OUTPUT)
print(f"Wrote {OUTPUT}")
```

Always:
- Read from `input/`, write to `output/`. Never overwrite the source.
- Use `recover.readfile()` not `readfile()`.
- Print what you did (entity counts, layers touched) at the end.
- Run `validate.py` on the output before reporting back to the user.

## Standards Files

`standards/layers.yml` and `standards/blocks.yml` are templates with placeholder conventions. **The user must customize these to match their firm's CAD standard.** When you first run on a real project, ask the user whether the standards files reflect their office practice, and offer to update them.

## File Format Notes

- This skill operates on `.dxf`. For `.dwg`, the user must convert via AutoCAD (`DXFOUT` command) or the free ODA File Converter before any of this works. Mention this if they hand you a `.dwg`.
- DXF versions R12 through R2018 are supported. The drawing's version is reported in `summary.py` output; if you save modifications, preserve the original version unless the user asks otherwise.
- Units are read from the `$INSUNITS` header variable. Imperial drawings (US civil work) often have `$INSUNITS = 2` (feet); metric drawings use `4` (mm) or `6` (m). If `$INSUNITS = 0`, units are unspecified — ask the user.

## What This Skill Doesn't Do

Be honest with the user about these limits:

- **No DWG support** without external conversion. ezdxf does not read DWG.
- **No Civil 3D parametric objects.** Corridors, surfaces, alignments, pipe networks appear as proxy entities and cannot be edited. You can read their bounding boxes and proxy graphics, nothing more.
- **No rendered output guarantees match AutoCAD's plot.** `render.py` uses matplotlib; it's for the agent's spatial reasoning and quick previews, not deliverable plotting.
- **No legal substitute for a licensed engineer's review.** Anything this skill produces is a draft.

---
> Source: [pedroschz/dxf-civil](https://github.com/pedroschz/dxf-civil) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
