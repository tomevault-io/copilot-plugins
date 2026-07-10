## drill-press-sharpening-disc

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **mechanical design project** for a drill press-mounted sharpening disc system. The project consists entirely of **OpenSCAD files** for 3D-printable components and comprehensive **markdown documentation**. There is no traditional software build system, compilation, or testing infrastructure.

**Target Equipment:**
- Bosch PBD 40 drill press
- Bambu Lab H2D 3D printer with carbon fiber nylon (PPA/PA6)

**Core Design:** 90° bevel gear drive system (2:1 ratio) converting vertical drill press rotation to horizontal disc rotation for tool sharpening.

## OpenSCAD Development Workflow

### Required Library
All `.scad` files depend on the **BOSL2 library** (Belfry OpenSCAD Library v2). Users must install it:
```bash
cd ~/Documents/OpenSCAD/libraries
git clone https://github.com/BelfrySCAD/BOSL2.git
```

### File Organization
```
openscad/
├── bevel_gears.scad           # 30T pinion + 60T wheel (2:1 ratio)
├── bearing_housings.scad      # 6001-2RS (horizontal) + 6000-2RS (vertical)
├── disc_hub.scad              # 250mm MDF disc mounting system
├── gear_housing.scad          # Protective enclosure with dust seals
├── tool_rest_system.scad      # Adjustable angle tool rest (20°/25°/30°)
└── drill_press_mount.scad     # Base mounting to drill press table
```

### Rendering Parts

Each OpenSCAD file uses a **`render_part` variable** to control which component is generated:

```openscad
render_part = "both";  // Options vary by file
```

**Common render_part options:**
- `"assembly"` - Preview all parts positioned correctly
- Individual part names (e.g., `"pinion"`, `"wheel"`, `"housing"`, `"tool_rest"`)
- `"both"` - Two main parts side-by-side for efficient printing

**To generate STL files:**
1. Open `.scad` file in OpenSCAD
2. Set `render_part` to desired component
3. Press F6 (full render)
4. Press F7 (export STL)

### Key Design Parameters

**Critical dimensions referenced across files:**

```openscad
// Gears (bevel_gears.scad)
module_size = 3.0;           // Gear module
pinion_teeth = 30;           // Vertical gear
wheel_teeth = 60;            // Horizontal gear (2:1 ratio)
pinion_bore = 10;            // 10mm shaft
wheel_bore = 12;             // 12mm shaft

// Bearings (bearing_housings.scad)
// Horizontal: 6001-2RS (12mm ID, 28mm OD, 8mm W)
// Vertical: 6000-2RS (10mm ID, 26mm OD, 8mm W)

// Disc (disc_hub.scad)
disc_diameter = 250;         // 10 inch disc
disc_thickness = 6;          // MDF thickness
```

**Material specifications:**
- Gears: Carbon fiber nylon (PPA/PA6), 100% infill on teeth
- Housings: CF-nylon or PETG, 40% infill
- Print layer height: 0.15mm (gears), 0.2mm (other parts)

## Project Architecture

### Component Relationships

```
Drill Press Spindle (vertical)
    ↓
10mm Vertical Shaft
    ↓
Bevel Pinion (30T) ←→ Bevel Wheel (60T) [meshed at 90°]
                           ↓
                    12mm Horizontal Shaft
                           ↓
                    Disc Hub → 250mm MDF Disc
```

**Power flow:** Drill press (200-850 RPM) → pinion → 2:1 reduction → wheel → disc (100-425 RPM)

### Critical Design Decisions

**Why bevel gears instead of worm gears:**
- Efficiency: >90% (vs. 30-60% for worm)
- Heat: Low generation (vs. high heat in worm)
- Material: Printable in nylon (worm gears fail quickly in plastic)
- Speed range: Perfect for drill press RPM range

**Design validation:** Reviewed by GPT-5 Pro and Grok 4 (see README.md lines 198-230)

### Bearing System

**Horizontal shaft (12mm):**
- 2× 6001-2RS bearings in printed housings
- Thrust washers for axial load
- Spacing: ~150mm apart

**Vertical shaft (10mm):**
- 2× 6000-2RS bearings
- Top bearing in pinion support
- Bottom bearing in base mount

## Modifying Designs

### Changing Gear Ratio

To modify the speed ratio, edit `bevel_gears.scad`:
```openscad
pinion_teeth = 30;    // Change this
wheel_teeth = 60;     // And/or this
```

**Important:** Ratio affects:
- Output speed (current: 2:1 reduction)
- Gear mesh geometry (recalculated automatically)
- Cone distance (automatically computed)
- Housing clearances (may need manual adjustment)

### Changing Disc Size

Current: 250mm (10"). To change, edit `disc_hub.scad`:
```openscad
disc_diameter = 250;
```

**Dependencies:**
- Larger discs require rebalancing calculations
- Base dimensions (WOOD_CUTTING_LIST.md) may need adjustment
- Sandpaper availability (10" is standard)

### Adding Custom Features

When modifying `.scad` files:
1. All files use BOSL2 library functions (e.g., `cuboid()`, `bevel_gear()`)
2. Check BOSL2 documentation: https://github.com/BelfrySCAD/BOSL2/wiki
3. Test mesh with `render_part = "assembly"` before printing
4. Verify clearances (use 0.2-0.3mm for sliding fits)

## Documentation Structure

**Technical specifications:** PROJECT_SPECIFICATIONS.md
- Drill press specs (Bosch PBD 40)
- Gear calculations and ratios
- Bearing specifications
- Safety features

**Bill of materials:** COMPLETE_BOM.md
- All bearings, fasteners, rods
- Purchase links and costs
- Alternative suppliers

**Wood components:** WOOD_CUTTING_LIST.md
- Base plate: 350×280×15mm plywood
- MDF discs: 250mm diameter × 6mm
- Cutting diagrams with hole patterns

**Assembly:** ASSEMBLY_INSTRUCTIONS.md
- 10 phases, ~6-8 hours total
- Phase 3 (Gear Assembly) is CRITICAL - gear mesh must be perfect
- Includes troubleshooting section

**Quick reference:** QUICK_VISUAL_GUIDE.md + ASSEMBLY_DIAGRAMS.md

## Common Tasks

### Validating Gear Mesh

Before printing gears, verify mesh geometry:
```openscad
render_part = "assembly";  // in bevel_gears.scad
```

Check in OpenSCAD preview:
- Tooth contact along full face width
- No interference at root or tip
- Proper backlash (0.3mm)
- Cone angles sum to 90°

### Testing Parameter Changes

When modifying critical dimensions:
1. Change parameter in relevant `.scad` file
2. Render preview (F5) - fast check
3. Verify clearances and fits
4. Full render (F6) - check for geometry errors
5. Export test piece, print small section first

### Generating Full STL Set

To generate all printable parts:
1. Open each of 6 `.scad` files
2. Set `render_part` to each component option
3. Export STLs with descriptive names:
   - `bevel_pinion_30T.stl`
   - `bevel_wheel_60T.stl`
   - `horizontal_bearing_housing_left.stl`
   - etc.

**Print time estimate:** 68-82 hours total
**Filament needed:** ~2.0-2.2 kg carbon fiber nylon

## Design Philosophy

This project prioritizes:
1. **Mechanical reliability** - Bevel gears over worm gears for efficiency
2. **Standard components** - Off-the-shelf bearings, shafts, fasteners
3. **Printability** - All parts fit Bambu H2D build volume (325×320×325mm)
4. **Maintainability** - Modular design, replaceable components
5. **Safety** - Enclosed gears, balanced disc, proper guards

## Speed Calculations

Current 2:1 ratio provides:
- 200 RPM input → 100 RPM disc (ultra-fine honing)
- 400 RPM input → 200 RPM disc (ideal sharpening) ⭐
- 600 RPM input → 300 RPM disc (moderate removal)
- 850 RPM input → 425 RPM disc (fast removal)

Speed range is ideal for tool sharpening (100-425 RPM). Commercial disc sanders typically run 800-2000 RPM (too fast for precision sharpening).

---
> Source: [jnyross/drill-press-sharpening-disc](https://github.com/jnyross/drill-press-sharpening-disc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-10 -->
