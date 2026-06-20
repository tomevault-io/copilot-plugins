## ansi-skill

> >


# ANSI Art Creation Guide

This skill teaches you how to create compelling ANSI art using the `window.ansi` programmatic API. It covers the artistic fundamentals (color, shadow, composition) and the technical workflow (API patterns, layering, reference conversion).

Read `community-techniques.md` for the full catalog of advanced drawing techniques from community tutorials. Read `material-optics-reference.md` for material optical properties used in Phase 6 mask generation.

## The CGA Palette: Your Paint Box

You have exactly 16 colors. Learning their relationships is the single most important thing for making good ANSI art.

### Color Groups by Luminance

Think of the palette in 4 brightness tiers:

| Tier | Colors | Indexes | Use |
|------|--------|---------|-----|
| **Black** | Black | 0 | Deepest shadow, outlines, negative space |
| **Dark** | Blue, Green, Cyan, Red, Magenta, Brown | 1-6 | Shadows, base tones, backgrounds |
| **Medium** | Dark Gray, Light Blue, Light Green, Light Cyan, Light Red, Light Magenta, Yellow | 7-14 | Midtones, highlights on dark areas |
| **Bright** | White | 15 | Brightest highlights, specular, text |

Dark Gray (8) is your most versatile shadow color — it works with almost everything.

### Natural Color Pairs (shadow → highlight)

These are the pairs that create convincing shading when placed adjacent:

- **Blues:** 1 (dark blue) → 9 (light blue) → 15 (white)
- **Greens:** 2 (green) → 10 (light green) → 15
- **Cyans:** 3 (cyan) → 11 (light cyan) → 15
- **Reds:** 4 (red) → 12 (light red/pink) → 15
- **Magentas:** 5 (magenta) → 13 (light magenta) → 15
- **Browns/Yellows:** 6 (brown) → 14 (yellow) → 15
- **Grays:** 0 (black) → 8 (dark gray) → 7 (light gray) → 15 (white)

**The gray ramp** (0 → 8 → 7 → 15) is essential. It provides 4 neutral tones for any subject.

### Color Relationships for Skin/Flesh Tones

ANSI art has no peach or beige. Artists approximate skin using:
- **Light skin:** Brown (6) base, Yellow (14) highlight, Dark gray (8) shadow
- **Dark skin:** Red (4) or Brown (6) base, Light red (12) midtone, Dark gray (8) shadow
- **Blue/fantasy skin:** Cyan (3) base, Light cyan (11) highlight, Blue (1) shadow
- **Metallic/armor:** Dark gray (8) base, Light gray (7) midtone, White (15) specular

### The Brown Problem

Brown (6) and Yellow (14) are the same hue at different brightnesses. Brown renders as dark orange/brown on screen. There's no background yellow — only background brown. This means yellow text on brown background becomes invisible. Plan around this.

## Shadow & Light Theory

### Establishing a Light Source

Before drawing anything, decide where light comes from. Upper-left is the most natural default. Then apply consistently:

- **Lit faces:** Use the bright variant of your base color
- **Shadow faces:** Use the dark variant — but match the hue (see below)
- **Ambient occlusion:** Where surfaces meet (under chin, inside ears, behind hair), go to black (0)
- **Specular highlights:** Tiny white (15) spots where light hits directly

### Color-Appropriate Shadows

This is critical and easy to get wrong: shadow colors must match the hue of the surface they're on. Dark gray (8) is versatile but it's a neutral — using it on a strongly colored surface kills the color identity.

- **Blue skin/surfaces:** Use blue (1) for shadows, not dark gray (8). Gray shadows make blue skin look lifeless and gray.
- **Green surfaces:** Use green (2) for shadows. Dark gray strips the natural feel.
- **Red surfaces:** Use red (4) for shadows. Gray shadows look like the red is fading.
- **Warm skin (brown/yellow):** Dark gray (8) works here because brown is already desaturated.
- **Metallic/neutral surfaces:** Dark gray (8) is correct — metal is inherently neutral.

The rule: if the subject has a strong hue identity, shadow with the dark variant of that hue, not with neutral gray. Only use gray shadows on surfaces that are already neutral.

### The Three-Tone Rule

Every surface needs at minimum 3 tones to read as 3D:
1. **Shadow tone** — the darkest value on that surface
2. **Base tone** — the dominant mid-value
3. **Highlight tone** — where light strikes

Example for a blue sphere: Blue (1) shadow → Cyan (3) base → Light Cyan (11) highlight, with a single White (15) specular dot.

### Shadow Characters

Shade blocks create gradual transitions between tones:

| Character | Code | Name | Density |
|-----------|------|------|---------|
| `░` | 176 | Light shade | ~25% filled |
| `▒` | 177 | Medium shade | ~50% filled |
| `▓` | 178 | Dark shade | ~75% filled |
| `█` | 219 | Full block | 100% filled |

Use them in sequence for smooth gradients. A transition from light to dark might go:
`░` (light fg on dark bg) → `▒` → `▓` → `█` (solid dark)

The foreground/background color combo on shade characters creates a blended appearance. For example, light cyan foreground + blue background on `▒` creates a mid-tone blue-cyan that neither solid color provides.

#### Shade Blocks for Glow Effects

Shade blocks excel at creating atmospheric glow around bright/energy sources. The technique:
1. Identify the bright energy source (a light, fire, magic, or other luminous element)
2. Apply BFS distance mapping outward from bright cells to nearby dark cells
3. Select shade block density by distance: close (1-2 cells) = `▓`, medium (2-3 cells) = `▒`, far (3-5 cells) = `░`
4. Use the dark variant of the energy color (bright blue glow → dark blue shade blocks, warm fire glow → dark red shade blocks)
5. Apply only to dark/black cells — never overwrite existing art

Example: A magic gem on a dark background gets a glow halo:
- Gem itself: bright yellow (14) in full block `█`
- Adjacent dark cells (distance 1): dark blue `▓` with blue (1) foreground creates electric atmosphere
- Further dark cells (distance 2-3): dark blue `▒` creates a softer halo
- Distance 4+: Either no change or very light `░` if the glow should be subtle

This technique is automated in Phase 4 of the pipeline.

#### Shade Expansion for Solid-Color Cells

In the automated pipeline (Phase 3 tuned), shade blocks aren't limited to cells where top/bottom colors already differ. **Solid-color cells** (full block █ or space) are also evaluated against standard CGA luminance pairs: black↔dark gray (lum 0↔85), dark gray↔light gray (85↔170), light gray↔white (170↔255), and black↔light gray (0↔170). If a shade block matches the source luminance at least 5 units better than the solid color, the cell gets upgraded.

This is the single most impactful improvement for **dark/moody images**: a scene dominated by dark gray (59% of cells) where the source has smooth gradients within the 0–85 luminance range gains 6–7× more shade blocks, transforming flat dark masses into smooth tonal transitions. Example: Frieren portrait went from 1,254 to 8,065 shade blocks after shade expansion — from 10.8% to 69.5% shade coverage.

#### Depth-of-Field Processing

For images with blurry backgrounds and sharp foreground subjects (bokeh portraits, selective focus), use the DOF pipeline:

1. **Phase 0.5:** Run `phase05-sharpness-mask.py <source_image> [cols]` to generate a per-cell sharpness mask. The script uses three detection signals — local variance, edge density, and color coherence — with adaptive weighting (bokeh images lean on variance; subtle depth separation uses all three equally). The transition gradient width is adaptive: narrow for sharp bokeh cutoffs, wide for progressive distance blur.

2. **Phase 3 (DOF):** Run `stage3-charselect-dof.py <hex> <img> <cols> --mask <mask.bin>`. Soft/background cells (mask < 0.3) get **shade blocks only** — no diagonals, no fine lines — creating smooth gradients that read as "blurry." Sharp/foreground cells (mask ≥ 0.7) get the full character set. Transition cells get shades plus horizontals/verticals.

3. **Phase 5 (DOF):** Run `phase5-detail-dof.py <hex> <img> <cols> --mask <mask.bin>`. Detail recovery is skipped in soft regions to preserve the blur effect.

**When to use DOF:** Any image where the background is noticeably softer than the foreground — portrait photos with shallow depth of field, anime scenes with bokeh, character art against blurred environments. **When NOT to use:** Images where everything is in focus, or where both foreground and background should be equally detailed.

#### Small Source Image Strategy (<500 px)

When the source is very small (under ~500 px wide), each ANSI cell covers very few source pixels. The pipeline automatically shifts strategy:

- **Phase 0:** Use `scale=3` with `sharpen_radius=3, sharpen_percent=200` (stronger than default to recover detail after upscaling)
- **Column count:** Keep at 80–120, not higher. At 120 cols on a 360 px source, each cell already covers only 3 source pixels. Going to 200 cols means cells have <2 px of real data.
- **Phase 3:** Use `stage3-charselect-tuned.py` with the `--no-expansion` flag. This keeps the tuned thresholds (lower gradient/edge detection, catches finer diagonals) while **skipping shade expansion** (Pass 1b). At small-source resolution, shade expansion can over-darken bright/colorful images — the Fern child (360×203) pipeline showed 86% of cells converted to dark shades, muddying the scene. With `--no-expansion`, shade blocks still apply to diff-color cells (Pass 1a) but solid-color cells stay crisp.
- **DOF mask:** Usually not needed — small source images are typically close-up portraits or character shots where the subject fills the frame. Exception: if the source clearly has a soft background (e.g., outdoor scene with bokeh), DOF can smooth background speckle.

#### Shade Blocks for Transition Blending

Where two distinct color regions meet (e.g., blue skin against red background), shade blocks smoothly blend the boundary:
- Place the shade block on the boundary cell
- Use foreground = one color, background = the other
- Example: blue skin `▓` with cyan foreground (11) + blue background (1) = transitional cell that reads as both blue and cyan

#### Shade Blocks for Texture Simulation

Flat surfaces gain perceived texture and detail through shade block dithering:
- Use `░▒▓` patterns at sub-cell level to suggest surface irregularity
- Wood/rough texture: mix base color with a darker variant using shade blocks
- Metal: use gray ramp with shade blocks for subtle surface irregularity
- Fabric: strategic shade blocks suggest wrinkles and folds

### Anti-aliasing with Characters

Soften hard edges by placing transitional characters between contrasting areas. Instead of a sharp white-to-black edge, insert a gray cell between them. The shade blocks are perfect for this — they visually blend adjacent colors.

The lesson from classic ANSI artists: "Leave a tiny line of black space between different colors" for clean separation, then use shade blocks to anti-alias curved edges.

## Advanced Character Selection

Not all characters are created equal. The optimal choice for each image region depends on what detail you're trying to capture. The 6-phase pipeline automates this selection, but understanding the tradeoffs helps with manual refinement.

### Character Types and Their Strengths

**Half-blocks** (`▀` code 223, `▄` code 220): Standard choice for most pixel art
- Use for: portraits, smooth gradients, standard reference-to-ANSI conversion
- Strength: Every pixel explicitly placed, predictable results
- Limitation: Can look blocky on diagonal edges

**Shade blocks** (`░▒▓` codes 176/177/178): Best for smooth luminance transitions and atmosphere
- Use for: glow effects, gradual shadows, atmospheric depth, texture
- Strength: Natural blending, atmospheric effects, requires fewer cells to show gradients
- When to prefer over half-blocks: Smooth light-to-dark transitions (3+ tone gradient in 1 cell), glow halos, underwater/fog atmosphere
- Example: A shadow gradient that takes 5 cells with half-blocks might need only 2-3 cells with shade blocks: `▓` (dark) → `▒` (mid) → `░` (light)

**Diagonal characters** (`/` code 47, `\` code 92): Best for edge anti-aliasing and diagonal features
- Use for: diagonal lines at ~45° angles, edge anti-aliasing on curved boundaries, diagonal stripes
- Strength: Anti-aliases diagonal edges smoothly, captures diagonal details
- When to prefer over half-blocks: Corners of circles/ovals, diagonal edges in characters/shapes, diagonal hatching for texture
- Example: A circular object's edge goes from inside (bright) to outside (dark):
  - With half-blocks: blocky staircase effect
  - With diagonals + anti-aliasing: smooth curved edge

### Decision Matrix: When to Use Which

| Situation | Best Choice | Why |
|-----------|------------|-----|
| Portrait or detailed face | Half-blocks | Every pixel matters; smooth gradients |
| Glow around light source | Shade blocks | Atmospheric + requires fewer cells |
| Smooth shadow gradient | Shade blocks or dither | 3+ tones in small space |
| Diagonal edge (45°) | Diagonal chars | Smooth anti-aliasing |
| Hard vertical/horizontal edge | Half-blocks | Clean, predictable |
| Metallic/smooth surface | Half-blocks base + shade blocks for texture | Combine approaches |
| Underwater/fog atmosphere | Shade blocks dominant | Atmospheric effect |
| Thin diagonal line | Diagonal chars | Only way to render at angle |
| Texture on flat surface | Shade blocks dither | Simulate material detail |

### Character Hex Codes Reference

Beyond half-blocks and shade blocks, Phase 3 can output various CP437 characters. Key ones:

| Character | Code | Common Use |
|-----------|------|-----------|
| `▀` | 223 | Upper half-block (standard) |
| `▄` | 220 | Lower half-block (standard) |
| `█` | 219 | Full block (solid) |
| ` ` | 32 | Space (empty cell) |
| `░` | 176 | Light shade (25% fill) |
| `▒` | 177 | Medium shade (50% fill) |
| `▓` | 178 | Dark shade (75% fill) |
| `/` | 47 | Diagonal line (upper-left to lower-right) — 4×4 pixel mask: top-right to bottom-left |
| `\` | 92 | Diagonal line (upper-right to lower-left) — 4×4 pixel mask: top-left to bottom-right |

The diagonal characters `/` and `\` provide single-cell diagonal edges. On a 4×4 pixel grid (one cell's visual size), they split the cell diagonally.

## Half-Block Drawing

Half-blocks (`▀` code 223, `▄` code 220) double your vertical resolution. Each cell becomes two independent "pixels" — the foreground color fills one half, the background color fills the other.

### How It Works

- `▀` (upper half block): foreground = top pixel, background = bottom pixel
- `▄` (lower half block): foreground = bottom pixel, background = top pixel
- `█` (full block): both pixels same color (use foreground)
- ` ` (space): both pixels same color (use background)

On an 80x25 canvas, half-blocks give you effectively 80x50 pixel resolution. At the recommended 120 columns, a typical portrait becomes 120x90 cells = 120x180 half-block pixels — significantly more detail. The `ansi.pixel(px, py, colorIndex)` API method handles this automatically.

### When to Use Half-Blocks vs. Characters

- **Half-blocks** for: portraits, detailed images, anything needing smooth curves, reference image conversion
- **Characters** for: text art, logos, stylized pieces, things with intentional blockiness, adding texture and personality

Many great pieces mix both — half-blocks for smooth areas (skin, sky) and CP437 characters for textured areas (hair, clothing, backgrounds).

## Reference-to-ANSI Workflow

When given a reference image to recreate, **always start by auto-tracing** to get an accurate baseline, then refine artistically. Never try to draw from scratch — the trace gives you correct proportions, silhouette, and spatial layout for free.

### Preferred Workflow: 6-Phase Automated Pipeline

The most powerful approach uses a six-phase automated pipeline followed by manual refinement. This pipeline is fully automated via Python scripts and produces publication-quality results consistently:

**Phase 0: Source Preparation**
Prepare the reference image for optimal color and detail preservation:

```bash
python3 phase0-prep.py <image_path> [scale=3] [sharpen_radius=2] [sharpen_percent=150]
# Operations:
# - Lanczos upscale by scale factor (use scale=1 for large sources 2000+ px)
# - Unsharp mask (enhance fine detail)
# - CLAHE (Contrast Limited Adaptive Histogram Equalization)
# - Contrast boost to increase local variation
```

**Scale factor by source size:** Use `scale=1` for large sources (2000+ px wide, e.g., 4K wallpapers) — they already have enough detail. Use `scale=2` for medium (1000–2000 px) and `scale=3` for small (<1000 px). **For very small sources (<500 px wide):** use `scale=3` with stronger sharpening (`sharpen_radius=3, sharpen_percent=200`) to compensate for Lanczos softening, and keep cols at 80–120 (going wider means cells have <2 px of real information, producing noise instead of detail). The enhancement steps (sharpen, CLAHE, contrast) are always beneficial regardless of source size.

**Phase 1: Auto-Trace with Perceptual Color Mapping**
Map every pixel to the nearest CGA color using perceptual color distance:

```bash
python3 trace-to-ansi.py <image_path> [cols=120] [bg_color=0]
# Outputs: phase1-traced.hex (baseline color grid)
# Uses ITU-R BT.601 weighted color distance
```

This gives you:
- Correct aspect ratio (auto-calculated from image dimensions)
- Accurate silhouette and proportions
- Baseline CGA color mapping using perceptual distance

**Phase 2: Context-Aware Refinement**
Refine colors based on local context and HSL classification:

```bash
python3 refine-<subject>.py <image_path> [cols=120]
# Passes:
# - HSL pixel classification (hue-based color family detection)
# - Context-specific CGA ramp selection (blue skin → blue ramp, etc.)
# - 5 refinement iterations (grayscale → color boundaries → saturation → shadows → highlights)
```

The algorithm detects strong hue identities and remaps neutral grays to hue-matched shadows (blue skin uses dark blue, not gray; red surfaces use red shadows, etc.).

**Monochromatic-lit scenes:** When a single light source (fire, moonlight, neon) illuminates everything, ALL pixels share the same hue. A firelit scene has hue 9-23° across every region. **Hue-based classification collapses.** Switch to luminance-first: use luminance + saturation as primary axes, hue as secondary. See `refine-calcifer.py` for reference.

**Critical Phase 2 guardrails:**
- After running your refine script, run the color temperature verification from AI-README Step 5d. Do NOT proceed if source and output temperatures differ by >25%.
- Keep context categories to 5-8 for most images. Never exceed 10.
- Black outlines (luminance < 40) must map to CGA 0 (Black). Do not use chromatic colors for outlines.
- Match source color temperature: cool source → cool CGA family, warm source → warm CGA family.

**Multi-point sampling:** For images with distinctive features (cartoon eyes, mouths, outlines), use 9+ samples per cell instead of 1. When internal luminance range is bimodal (bright + dark within same cell), an outline crosses through it — mark it as "protected" so cleanup passes don't destroy it.

**Phase 2.5: Feature Exaggeration**
Detect art-style features and exaggerate them for readability at ANSI resolution:

```bash
python3 phase25-features.py <refined_hex> <source_image> [cols=120]
# Algorithm:
# - Sobel edge map from source at cell resolution
# - Intra-cell contrast map (25 samples/cell, detect sub-cell features)
# - Feature clustering via flood-fill
# - Small feature dilation (expand clusters < 8 cells by 1 cell)
# - Contrast exaggeration (dark features → black, bright features → white)
# - Contrast ring (boost 1-cell border around features)
```

This compensates for the fundamental problem that faithful pixel mapping makes small features (1-2 cell cartoon eyes) unreadable. A human artist would draw those eyes 4 cells wide with maximum contrast — this pass does the same automatically. **Use for:** cartoon/anime sources, any image with distinctive facial features, text, logos. **Skip for:** purely atmospheric scenes, landscapes without specific features.

**Phase 3: Character Selection**
Choose the optimal CP437 character for each pixel region based on luminance and edges:

```bash
python3 stage3-charselect.py <refined_hex> <source_image> [cols=120]
# Character selection rules:
# - Shade blocks (▓▒░ B0/B1/B2): Smooth luminance gradients, glow effects
# - Half-blocks (▀▄ DF/DC): Standard pixel art, default choice
# - Diagonal chars (/ \): Edge anti-aliasing, diagonal lines at 45° angles
# - Context sensitivity: Choose character that best represents local pixel cluster
```

This phase outputs a grid where each cell contains a character code and color pair.

**Phase 4: Glow and Atmosphere**
Detect bright/energy source cells and apply atmospheric glow in adjacent regions:

```bash
python3 phase4-glow.py <charselect_hex> [cols=120]   # Note: NO source image arg
# Algorithm:
# - BFS distance map from bright cells (energy/warm sources)
# - Shade block glow: close=▓, medium=▒, far=░
# - Color: dark variant of energy color (bright blue → dark blue glow)
# - Apply only to dark/black cells (preserve existing art)
```

This creates atmospheric depth and highlights without overwriting existing detail.

**Phase 5: Detail Recovery and Edge Enhancement**
Use the original image's edge information to recover fine lines and enforce contrast:

```bash
python3 phase5-detail.py <glow_hex> <original_image> [cols=120]
# Operations:
# - Sobel edge analysis from source image
# - Contrast enforcement: brighten/darken high-edge-density cells
# - Fine line recovery: detect top/bottom luminance splits for thin horizontal features
# - Stray pixel cleanup: remove isolated bright pixels in dark neighborhoods
# - Edge color consistency: smooth color transitions along horizontal runs
```

The result is publication-ready ANSI art with recovered detail, clean edges, and atmospheric depth.

**Recommended configuration:** Choose column width by aspect ratio — 120 cols for portrait images (1:1.5 or taller), 160–200 cols for landscape images (16:9, 1.7:1, etc.). **For very small sources (<500 px):** keep cols at 80–120 regardless of aspect ratio — wider means cells have <2 px of real info. Landscape images at 120 cols get too few rows for meaningful detail (e.g., a 16:9 image at 120 cols = only 34 rows / 4,080 cells; at 200 cols = 58 rows / 11,600 cells). The HTML viewer handles any width via responsive `max-width: 95vw` CSS. Use the retuned script variants (`*-tuned.py`) for best results — they use less destructive cleanup, catch finer edges and gradients, apply tighter glow, and include **shade expansion for solid-color cells** (critical for dark/moody images where 60%+ of cells would otherwise stay flat). For **depth-of-field images** (blurry background, sharp foreground), use the DOF variants (`*-dof.py`) with a sharpness mask from `phase05-sharpness-mask.py`.

**When to add Phase 2.5 (Feature Exaggeration):** Run `phase25-features.py` between Phase 2 and Phase 3 whenever the source has distinctive features that must read clearly at ANSI resolution — cartoon eyes, mouths, outlines, logos, text. The pass detects where high-contrast features exist in the source, dilates small ones to minimum readable size, and pushes contrast to maximum at feature boundaries. Skip for purely atmospheric scenes (landscapes, abstract art) where there are no specific features to preserve.

**Phase 6: Material-Aware Tone Correction**

Phase 6 introduces semantic understanding to the pipeline. Instead of treating every pixel with the same global color strategy, it recognizes what objects are in the scene and adjusts tone rendering based on how those materials interact with light.

The process requires three inputs:
1. The hex output from Phase 5
2. The source image
3. A **material segmentation mask** — a PNG where pixel colors indicate material type — paired with a **mask-key JSON** mapping those colors to optical properties

The segmentation mask is generated by an LLM-written per-image script. The LLM views the source image, identifies materials present (teeth, skin, metal, fabric, energy effects, etc.), and writes heuristic rules to classify regions. The mask doesn't need pixel-perfect accuracy — even rough region identification dramatically improves output quality.

Each material has optical properties defined in the `material-optics-reference.md`:
- **Ambient resistance**: How much the material resists the scene's dominant color tint (0.0 = fully follows ambient, 1.0 = ignores ambient entirely)
- **Transition sharpness**: Whether shade gradients should be smooth (skin, fabric), hard (metal, eyes), or shimmering (energy)
- **Brightness bias**: Whether the material should render brighter or darker than its raw luminance

The correction script compares each cell's current tone against what the source image shows for that material, then nudges cells toward the optically correct appearance. Teeth that were tinted blue get desaturated. Armor that was pushed too bright by feature exaggeration gets pulled back. Skin gets warmed relative to surrounding surfaces.

Keep material count low: 3-5 for simple cartoons, 5-8 for portraits, max 12 for complex scenes.

```
python3 phase6-material-correct.py <hex> <source_img> <mask_img> <mask_key.json> [cols=160]
# Reads material segmentation mask and corrects tone per-region
# Output: *-corrected.hex, *-corrected-preview.png
```

**The Material Optics Reference** (`material-optics-reference.md`) is a standalone document describing 10 material categories by their optical behavior — not by CGA color indices. It covers organic diffuse surfaces (teeth, bone), skin/flesh, hair, eyes, metal/reflective surfaces, transparent materials, fabric/cloth, emissive surfaces (fire, aura, magic), shadow/void, and atmospheric perspective. Each entry describes reflectance behavior, ambient resistance, shade transition character, and common rendering errors. The LLM consults this reference when writing the per-image mask generator and when setting optical properties in the mask-key JSON.

**Generating a mask for a new image:**
1. LLM views the source image
2. LLM identifies materials present and their approximate locations
3. LLM writes a mask generator script with per-pixel classification heuristics using color + position
4. Script outputs mask PNG + mask-key JSON
5. Phase 6 reads these and applies corrections

**Phase 7: LLM Visual Review Loop**

After Phase 6's automated correction, Phase 7 puts **you, the LLM** back in the loop. You will visually compare source vs output and make artistic corrections that no automated rule can catch.

**THIS PHASE REQUIRES YOU TO BE ACTIVELY IN THE LOOP.** You are not just running scripts — you are viewing images, making judgments, and writing correction files. Here is exactly what to do:

**Step 1: Generate comparison.**
```
python3 phase7-llm-review.py compare <hex> <source_img> [cols=160]
```
This outputs `*-comparison.png` (side-by-side: source left, ANSI right, with coordinate grid) and `*-review-prompt.md`.

**MANDATORY: Run the programmatic sanity check from AI-README Step 7a.5 BEFORE visual review.** This compares source vs output color distribution and catches temperature/brightness mismatches that visual review might miss. If any check fails, go back to Phase 2 — do not try to fix fundamental color mapping errors with Phase 7 corrections.

**Step 2: View the comparison image yourself.** Open the `*-comparison.png` with your vision capabilities. Compare source (left) vs ANSI output (right). Look for color temperature mismatches, brightness errors, lost features, material confusion.

**Step 3: Write a corrections JSON file.** If acceptable: `{"status": "ACCEPT", "iteration": N, "reason": "why"}`. If corrections needed: write regions with bounds, action, and intensity. See `*-review-prompt.md` for the exact format and available actions (desaturate, brighten, darken, warm, cool, max_contrast, set_color). Maximum 5 regions per round.

**Step 4: Apply corrections.**
```
python3 phase7-llm-review.py apply <hex> <corrections.json> [cols=160] [source_image]
```
**Always pass the source image** — enables source-aware mode where each cell's correction is proportional to how far it deviates from the source. This prevents visible rectangular patches from uniform corrections. Also applies edge feathering (cosine falloff at region borders) and automatic **global histogram anchoring** to prevent cumulative drift.

**Step 5: Coherence gate.** After apply finishes, view the `*-coherence.png` (zoomed-out full-image render). Check: does any corrected region look "pasted in"? Any style breaks or material color mismatches? If coherence is broken, write a ROLLBACK JSON to **re-map colors** (not restore from backup) — specify `from_colors` (wrong family, e.g. "gray") and `to_colors` (correct family, e.g. "cool"). This preserves all character shapes while shifting the hue. See AI-README Step 7e for format and available color families. If coherence is fine, continue.

**Step 6: Loop or stop.** ACCEPT → done. Below 50 cells modified → auto-accept. 5 iterations → forced stop. Otherwise → go to Step 1 with new hex.

```
python3 phase7-llm-review.py rebalance <hex> <source_img> [cols=160]
# Standalone: measure and correct global histogram drift
python3 phase7-llm-review.py status <hex> [cols=160]
# Show iteration history
```

This makes the pipeline **nine phases**: Phase 0 (Source Prep) → Phase 1 (Trace) → Phase 2 (Refine) → Phase 2.5 (Feature Exaggeration) → Phase 3 (Char Select) → Phase 4 (Glow/Atmosphere) → Phase 5 (Detail Recovery) → Phase 6 (Material Correction) → Phase 7 (LLM Visual Review).

**Why this beats freehand drawing:** The nine-phase pipeline captures the reference's exact shape, applies color theory automatically, understands material optics, and has an LLM visual quality loop. Each phase builds on the previous, refining the result without losing information. You get correct layout, optimal colors, feature preservation, artistic atmosphere, material-accurate tone rendering, and iterative visual refinement in one run. Manual freehand drawing requires solving all these problems simultaneously.

### Alternative: Browser-Based Interactive Tracing

If the ANSI editor is loaded in the browser with the v2 API, use the interactive overlay system instead:

### Step 0: Load the Reference and SIZE THE CANVAS

**CRITICAL: Always resize the canvas to match the reference image's aspect ratio.** The default 80×25 canvas has a 16:5 aspect ratio. A portrait image crammed into that becomes squat and distorted. This is the most common mistake — always fix it first.

```javascript
// Load the reference image
const info = await ansi.loadImage('data:image/png;base64,...'); // or URL
// Returns: { width, height, suggest: { halfblock, cell, compact } }

// ALWAYS resize to match the image's aspect ratio
const s = ansi.suggestCanvas();
// s.suggestions.halfblock = { cols: 80, rows: N, pixelRes: '80×M' }
// The rows value is calculated to preserve the image's aspect ratio

ansi.resize(s.suggestions.halfblock.cols, s.suggestions.halfblock.rows);
// Now the canvas matches the image proportions!

// Example: a 600×800 portrait image (3:4 ratio) → 80 cols × 54 rows (80×107 pixel res)
// Example: a 1920×1080 landscape (16:9) → 80 cols × 23 rows (80×45 pixel res)
// Example: a 500×500 square → 80 cols × 40 rows (80×80 pixel res)

// Enable the transparent underlay so you can trace over it
ansi.ref.show(0.4); // 40% opacity — visible but not overpowering

// Turn on the tracing grid to see exact cell boundaries
ansi.ref.grid('both'); // Shows both cell and half-block grids

// Side-by-side panel shows full reference next to canvas
ansi.ref.sideBySide(true);
```

**Why this matters:** In halfblock mode, each pixel is square (8×8 screen pixels). So for an image that's taller than wide, you need more rows. `suggestCanvas()` calculates this automatically — always use it.

**Switching between views for spatial awareness:** During drawing, toggle between overlay modes to check your work:
- `ansi.ref.show(0.3)` — light underlay while drawing, lets you see both reference and your art
- `ansi.ref.hide()` — hide overlay to see your art clearly, check overall impression
- `ansi.ref.show(0.6)` — heavier overlay for precise tracing of edges and boundaries
- `ansi.ref.zoom(true)` then `ansi.ref.zoomTo(px, py)` — zoom into specific regions for detail work
- `ansi.ref.sideBySide(true)` — full reference next to canvas for color comparison

**Using the tracing grid:** The grid shows you exactly where cell and half-block boundaries fall on the reference image. This is critical for understanding how much detail you can capture. Each grid cell is one character position; each half-block subdivision is one pixel in half-block mode.

### Step 1: Analyze the Reference

Study the image (with overlay active, you can see it directly on the canvas):
- What's the dominant color palette? Map it to CGA colors mentally
- Where is the light source? Identify shadow and highlight regions
- What's the main subject? What can be simplified or omitted?
- Use `ansi.suggestCanvas()` to see optimal canvas dimensions
- Use `ansi.ref.mapRegion(x, y, w, h)` to get CGA color analysis of specific regions

You will lose detail. That's expected and good. ANSI art is an interpretation, not a reproduction.

### Step 2: Plan the Composition

Decide framing and scale:
- Use `ansi.suggestCanvas()` to see recommended dimensions for the loaded image
- A face/portrait typically wants 20-40 columns wide, 15-25 rows tall
- Full body needs the entire canvas and will be very simplified
- Leave 1-2 rows/columns as margin when possible
- Center the subject or use rule-of-thirds

### Step 3: Background First

Always paint the background before the subject. Use `ansi.fillRect()` for solid areas and `ansi.shade()` for gradients. Dark backgrounds (blues, blacks) make foreground subjects pop.

Background techniques:
- Solid dark fill for focus on subject
- Gradient (dark to darker) for subtle depth
- Patterned (using shade characters or CP437 symbols) for texture

### Step 4: Block In Major Shapes Using Scanline Contours

For organic shapes (heads, bodies, animals), the most effective technique is **scanline contour rendering**. Instead of trying to plot individual pixels or use geometric primitives, define the shape as left-edge and right-edge arrays indexed by pixel row:

```javascript
// Define the silhouette as boundary arrays
const L = {2:33, 3:30, 4:27, 5:25, 6:23, ...}; // left edge per row
const R = {2:38, 3:41, 4:44, 5:46, 6:47, ...}; // right edge per row
```

Then fill between the boundaries with position-dependent color zones:

```javascript
for (let py = startRow; py <= endRow; py++) {
  if (!L[py]) continue;
  const left = L[py], right = R[py], w = right - left;
  for (let px = left; px < right; px++) {
    const t = (px - left) / w; // 0.0 = left edge, 1.0 = right edge
    let color;
    if (t < 0.05) color = 0;       // black outline
    else if (t < 0.20) color = 1;  // shadow zone
    else if (t < 0.65) color = 9;  // base tone
    else if (t < 0.90) color = 7;  // highlight zone
    else color = 0;                 // black outline (right edge)
    a.pixel(px, py, color);
  }
}
```

This approach is powerful because: it naturally produces organic curved silhouettes from simple numeric boundary data; the `t` parameter (normalized position within each row) lets you place shadow/highlight zones that follow the 3D form; and you can tune color zone boundaries without redrawing the shape.

Read `references/api-patterns.md` for the full scanline contour template and a worked portrait example.

### Step 5: Sculpt with Shadow and Light

With the base shape established, refine the color zones. Rather than re-rendering from scratch, overlay darker and lighter pixels in specific regions:

- **Shadow side:** Add darker pixels on the side away from light. Use the hue-appropriate shadow color (see Color-Appropriate Shadows above).
- **Under overhangs:** Brow ridge, nose, chin, beneath hair — these get the deepest shadow or black.
- **Between forms:** Where neck meets jaw, where arm meets body — ambient occlusion goes here.
- **Highlight side:** Add lighter pixels on surfaces facing the light. Forehead, cheekbone, nose bridge, chin.
- **Specular dots:** 1-2 white (15) pixels where light hits hardest. Use very sparingly.

Use highlights sparingly — too many and the piece looks washed out. Classic ANSI artists say: "never do straight line bright spots," meaning highlights should be organic shapes, not geometric lines.

### Step 6: Features and Details

Add eyes, mouth, distinctive markings, accessories. At this resolution, features need **proportional exaggeration**:

- **Eyes** need to be at least 4-5 pixels wide and 2-3 pixels tall to read as eyes. A realistic proportion would make them too small to see. Include a white specular highlight pixel in every eye — without it, eyes look dead.
- **Mouth/lips** should be at least 3-4 pixels wide. A single pixel of a different shade suggests lips.
- **Nose** is mostly implied by shadow placement — a darker vertical strip with a highlight pixel on the bridge.
- **Distinctive accessories** (headbands, earrings, scars, spikes) should be exaggerated in size. If they're the character's signature element, make them bigger than realistic.

### Step 7: Edge Cleanup and Anti-Aliasing

Review every edge where the subject meets the background:

- Add **dark gray (8) transition pixels** at the boundary between subject and black background. This creates a visual "softening" that prevents harsh aliasing. Place them at corners and along curves where the silhouette changes direction.
- Ensure the black outline is consistent — 1 pixel wide along the whole contour.
- Fix any stray pixels that broke through the outline during sculpting.

## Glow and Atmosphere Techniques

Glow effects create depth and atmosphere by placing darker variants of a color's shade in cells adjacent to bright/energy sources. This is automated in Phase 4 of the pipeline, but understanding the technique helps with manual refinement.

### The Glow Algorithm

**Input:** The current image (after Phase 3 character selection) and a list of bright/energy source cells.

**Step 1: Distance Map via BFS**
From each bright cell (energy/warm color, high luminance), compute the Manhattan distance to every other cell on the canvas. This creates a distance map where nearby cells have distance 1-2, medium-distance cells have 3-4, and far cells have 5+.

**Step 2: Shade Block Selection by Distance**
For each dark/black cell (background or shadow cells):
- Distance 1-2 from energy source: `▓` (dark shade, 75% filled)
- Distance 2-3: `▒` (medium shade, 50% filled)
- Distance 3-5: `░` (light shade, 25% filled)
- Distance 5+: No change (or very subtle effect)

**Step 3: Color Selection**
Use the *dark variant* of the energy source's hue:
- Bright blue (9) light source → glow in dark blue (1)
- Bright cyan (11) → glow in cyan (3)
- Bright yellow (14) or brown (6) fire → glow in dark red (4) or brown (6)
- Bright white (15) → glow in dark gray (8)

Example: A blue magical gem surrounded by darkness gets:
- Gem cell: bright blue `█` (219, color 9)
- Adjacent cells (distance 1-2): dark blue `▓` (178, color 1)
- Medium-far cells (distance 3-4): dark blue `▒` (177, color 1)
- Far cells (5+): might add dark blue `░` (176, color 1) very sparingly, or leave black

**Step 4: Preserve Existing Art**
Only place glow in cells that are currently dark (black 0, dark gray 8, or dark colors). Never overwrite bright areas, characters, or existing artwork. Glow should enhance, not replace.

### Typical Glow Radius

In practice, glow looks best with:
- **Tight glow (radius 2-3):** Energy source feels hot and intense, affects only immediate surroundings. **Recommended for 120-col canvases** — preserves detail while adding atmosphere.
- **Medium glow (radius 3-4):** Balanced, typical for magical or mysterious effects
- **Wide glow (radius 4-5):** Soft, diffuse light (large fire, moon, distant explosion). Default for 160+ col canvases.
- **Extreme glow (radius 5+):** Rare; typically used for dramatic effect or large light sources

### Advanced: Colored Glow

For multiple light sources (e.g., a blue flame and a red flame near each other):
1. Each light source computes its own distance map
2. For each dark cell, determine which light source is closest
3. Apply glow in that light's color
4. At boundaries between two light sources' influence, blend both colors with shade blocks

### When Not to Use Glow

- Outdoor/daylight scenes with diffuse natural light
- Solid-color backgrounds (glow requires dark areas to work)
- Pieces where contrast is already high (glow can muddy detail)
- When you want sharp, clean edges (glow softens boundaries)

## Detail Recovery and Edge Enhancement

After the six-phase pipeline completes, you can further refine the result using the original source image's edge information. This phase recovers fine details that might have been lost in color quantization and enhances contrast at boundaries.

### Sobel Edge Analysis

Use Sobel edge detection on the original reference image to create an edge map:
- High-value regions (0.5-1.0) indicate strong edges or rapid color transitions
- Medium-value regions (0.2-0.5) indicate gradual transitions or low-contrast details
- Low-value regions (0.0-0.2) indicate flat areas

### Contrast Enforcement

In regions of the ANSI art that correspond to high-edge areas in the source:
- **Brighten bright cells:** Push medium-bright cells toward white (15) or lighter variants
- **Darken dark cells:** Push medium-dark cells toward black (0) or darker variants
- **Saturate colors:** Shift neutral grays toward hue-matched colors (enhance blue on blue regions, etc.)

This makes edges pop without adding extra characters or dramatically altering the composition.

### Fine Line Recovery

Detect thin horizontal features in the source image that may have been lost:
1. Scan for rows where the top half and bottom half have significantly different luminance
2. This indicates a thin horizontal line (eyebrow, facial line, text edge, etc.)
3. In the ANSI art, use half-blocks strategically: if top is bright and bottom is dark, use `▀` (upper half-block) to emphasize the line

### Stray Pixel Cleanup

Remove isolated bright pixels in otherwise dark neighborhoods:
- Scan for bright cells (luminance > 180) surrounded mostly by dark cells
- If the bright cell has 6+ dark neighbors (out of 8), it's likely a stray pixel or noise
- Replace with a dark variant of the local average color

This prevents noise in the source image from creating visual artifacts in the ANSI art.

### Edge Color Consistency

Along horizontal runs of pixels, smooth color transitions to prevent jarring shifts:
- For each horizontal line, compute the average hue of the entire run
- Cells whose hue deviates significantly (more than 30° on the hue wheel) from neighbors should be adjusted toward the local average
- This preserves intentional color boundaries while smoothing unintended noise

### Step 8: Final Polish Pass

Elements drawn earlier may have been partially covered by later passes. In the final pass:

- Re-draw any elements that were occluded (e.g., spikes behind a collar, hair behind a headband)
- Add small accent details: texture on clothing, edge highlights on accessories
- Verify the overall read at 1:1 zoom — if something isn't visible at actual size, it's wasted effort

### Multi-Pass JavaScript Execution

Each step above should be a separate JavaScript execution block. This is important for practical reasons: if you try to do everything in one massive script and something goes wrong, you lose all the work. By executing in passes, each pass builds on the visible result of the previous one, and you can inspect and adjust between passes.

Typical pass sequence: (1) setup + background, (2) scanline contour fill, (3) shadow/highlight sculpting, (4) feature details, (5) edge cleanup and polish.

### Alternative Workflow: Grayscale Line Art First

The scanline contour approach above works well for programmatic drawing. But for maximum quality, the classic workflow used by top ANSI artists (zeroVision, Thrasher, enz0) follows a different order:

1. **Draw complete grayscale line art** — all shapes, features, details in black/gray/white only. Start with the eyes (they set the scale for everything), then work outward: eye sockets, nose, mouth, jawline, hair. Don't add color yet.
2. **Color area by area** — pick one region (nose, cheek, hair), fill with base color, add lighter highlight areas, add darker shadow areas, blend with shade characters (░▒▓). Repeat for each region.
3. **Refine** — check at 1:1 zoom, fix transitions between colored areas, add final details.

This produces the highest-quality results because you solve shape/proportion problems in grayscale before adding the complexity of color. See `community-techniques.md` for the full community-sourced coloring workflow.

### Critical Drawing Principles (from Community Tutorials)

These principles come from decades of ANSI art practice and are documented in tutorials on 16colo.rs:

- **"Draw BIG"** — More space = smoother shading gradients. 4 shade steps beat 1 sharp jump.
- **Break symmetry** — After mirroring an element (like eyes), add unique details to each side. Perfect symmetry looks mechanical and boring.
- **Consistent line segments** — When drawing diagonals, keep segment lengths uniform (e.g., 3,3,3 not 2,5,3). Random segment changes look like errors unless deliberately curving.
- **Fade line endings** — Don't stop lines abruptly. Use shade characters to fade: █ → ▓ → ▒ → ░ → nothing.
- **Black space matters** — "Pay attention to the black space as much as the grey." Negative space between elements (hair strands, facial lines) is as important as the elements themselves.
- **Selective detail** — "Don't try to catch every line, just the major ones." Capture the essence, not every detail.
- **Iterative revision** — The process is messy. Move features, delete and redo parts, try alternatives. "Any mistakes are happy mistakes."
- **1-cell spacing** — Leave 1 cell of black space between parallel lines for clean readability.

## New v2 Drawing Tools

### Pixel-Level Primitives

The v2 API adds drawing primitives that work directly in half-block pixel coordinates (80×N*2), so you don't have to manually manage upper/lower half-blocks:

```javascript
ansi.pixelLine(x0, y0, x1, y1, color);    // Bresenham line in pixel coords
ansi.pixelEllipse(cx, cy, rx, ry, color, true);  // Filled ellipse in pixel coords
ansi.pixelRect(x, y, w, h, color);         // Filled rectangle in pixel coords
ansi.peekPixel(px, py);                     // Read color at pixel coordinate
```

These are essential for drawing organic shapes — curves, diagonal lines, circles — at the half-block resolution. Previously you had to use `pixel()` one dot at a time.

### Dithering

Dithering blends two CGA colors visually by interleaving them in a pattern. This effectively gives you more than 16 perceived colors:

```javascript
// Ordered (Bayer matrix) dithering — structured, clean patterns
ansi.dither(x, y, w, h, color1, color2, threshold);
// threshold: 0.0 = all color1, 0.5 = 50/50 mix, 1.0 = all color2

// Random noise dithering — organic, noisy textures
ansi.noiseDither(x, y, w, h, color1, color2, ratio);

// Shade-character dithering at cell level (░▒▓)
ansi.shadeDither(x, y, w, h, fg, bg, density);
// density: 0=space, 0.25=░, 0.5=▒, 0.75=▓, 1.0=█
```

Use dithering for: smooth gradients between color pairs (e.g., cyan→light cyan), textured surfaces (skin, metal, fabric), atmospheric effects (fog, smoke), and any time you need a "color" that doesn't exist in the CGA palette.

### Sprite/Stamp System

For complex pixel patterns that you want to define once and stamp down:

```javascript
// 2D array stamp — each value is a CGA color index, -1 = transparent
ansi.stamp(startX, startY, [
  [0, 0, 11, 11, 11, 0, 0],
  [0, 11, 15, 11, 15, 11, 0],
  [11, 11, 11, 11, 11, 11, 11],
], -1);

// Hex string stamp — compact notation, '.' = transparent
ansi.stampHex(startX, startY, [
  '..BBB..',
  '.BFB FB.',
  'BBBBBBB',
]);
// Hex: 0=black, 1=blue, ..., B=lt cyan, F=white
```

Use stamps for: eyes, recurring details (studs, rivets), small icons, any repeated element.

## Composition at Low Resolution

### The Canvas Size

**Do NOT default to 80×25.** The canvas dimensions should match your subject. Some guiding principles:

- **Simplify aggressively.** A reference image might have thousands of colors and millions of pixels. You have 2,000 cells and 16 colors. Capture the *essence*, not the detail.
- **Exaggerate key features.** Eyes should be proportionally larger than realistic — at least 4-5 pixels wide even on a small face. Distinctive features (a character's signature color, a unique shape, signature accessories) should be amplified. If in doubt, make it bigger.
- **Use negative space.** Black or dark areas aren't wasted — they give the eye a place to rest and make lit areas more impactful.
- **One subject per canvas** usually works best at this resolution. Two subjects need careful balance.

### Framing Strategies

- **Close-up portrait:** Head fills most of canvas. Best for character detail. Use half-blocks.
- **Half-body:** Head + torso. Good balance of detail and context.
- **Full scene:** Multiple elements. Keep each simple. Use characters rather than half-blocks.
- **Logo/text piece:** Large stylized text with decorative elements. CP437 box-drawing characters shine here.

## Art Styles

### Blocky/Retro Style
Use full blocks (`█` 219), half blocks, and minimal character variety. Emphasize the grid. Think pixel art. Good for game sprites, icons, simple logos.

### Detailed/Painterly Style
Mix shade characters, half-blocks, and diverse CP437 glyphs. Anti-alias edges. Layer multiple passes of refinement. This is the style used for portraits and complex scenes.

### Text/Logo Style
Box-drawing characters (single `─│┌┐└┘`, double `═║╔╗╚╝`, and mixed), combined with shade blocks for fills and CP437 decorative characters. Good for title screens, banners, borders.

### Dithered/Textured Style
Heavy use of shade blocks (`░▒▓`) and textured characters (`≡`, `∞`, `≈`, `·`, `•`) to create surfaces that feel rough, organic, or atmospheric. Good for landscapes, abstract art, backgrounds.

## Community Techniques from 16colo.rs

These advanced techniques were identified from analyzing ~90 tutorials on 16colo.rs by top ANSI artists (Halaster, zeroVision, enzO, Thrasher, LDA, Zeus II). They represent hard-won community knowledge that goes beyond basic shading and character selection.

### 8.1 "The Plus" / Cross Texture (Halaster)

A shade-block texture pattern that creates a woven or crosshatch appearance using alternating `▒` blocks in a plus/cross arrangement. Place medium shade `▒` blocks at regular intervals (every 2-3 cells) in both horizontal and vertical directions, with full blocks `█` filling the gaps. The result reads as a fabric or mesh texture rather than a flat surface. Particularly effective for clothing, canvas, rough stone, and any surface that should feel textured rather than smooth.

**Pipeline application:** Phase 3 could detect low-variance regions (flat color areas) and apply plus-pattern dithering instead of uniform shade blocks, creating perceived texture without color changes.

### 8.2 Contrast Extremes (Halaster)

Deliberately push adjacent cells to maximum contrast — placing the brightest available color directly next to the darkest. Where conventional shading uses gradual ramps (dark → medium → light), contrast extremes skip the middle tones entirely: black `█` directly adjacent to white `█` or bright color `█`. This creates visual "pop" that draws the eye and defines focal points. Use sparingly — contrast extremes on every surface kills the effect. Reserve for: eyes, key facial features, light sources, metallic specular highlights.

**Pipeline application:** Phase 5 contrast enforcement already does a mild version. A "focal point boost" pass could identify the 5-10 highest-edge-strength cells and push them to maximum palette contrast.

### 8.3 Color Swirls — Organic Highlight Shapes (Halaster)

Instead of placing highlights in geometric patterns (rectangles, straight lines), scatter highlight pixels in organic, irregular clusters. A cheekbone highlight should be 3-5 pixels in an amoeba-like shape, not a straight horizontal line. Hair highlights should curve and taper, following the implied flow of strands. The principle: "never do straight line bright spots" — organic shapes read as natural light, geometric shapes read as rendering artifacts.

**Pipeline application:** Phase 5 fine line recovery could be extended with a "highlight scatter" pass that takes any straight-line highlight run (3+ bright pixels in a row) and perturbs 1-2 pixels to break the linearity.

### 8.4 Vertical Shading / "Chain Combo" (zeroVision, LDA)

Use vertical sequences of shade blocks within a single column to create chain-like or ribbed textures. Instead of horizontal shade transitions (the default in most pipelines), stack `░`, `▒`, `▓` vertically within 2-3 consecutive rows of the same column. This creates a visual rhythm that reads as chains, braided hair, ribbed fabric, or segmented armor. The key: maintain consistent vertical periodicity (same pattern repeats every N rows).

**Pipeline application:** Phase 3 currently analyzes luminance horizontally (left/right neighbors). A vertical gradient detection pass could identify columns where luminance changes vertically and apply vertical shade sequences instead of horizontal ones.

### 8.5 The Pnakotic Method — High-Intensity Color Variation (Halaster)

Named after artist Pnakotic, this technique uses maximum color variety within a small area by cycling through multiple CGA colors that share similar luminance but differ in hue. Example: a "white" highlight area might cycle through light cyan (11), light green (10), yellow (14), and white (15) — all high-luminance colors — creating a shimmering or iridescent effect. Works especially well for: magical effects, water reflections, metallic surfaces, gemstones, energy fields.

**Pipeline application:** Phase 2 refinement could detect very bright regions (luminance > 200) and, instead of mapping everything to white (15), distribute across the bright color family (11, 10, 14, 13, 15) based on source hue. This would replace flat white patches with shimmering color variety.

### 8.6 The Reversal (Halaster)

Swap foreground and background colors on shade blocks to invert the visual weight of a cell without changing the character code. A cell with `▒` in light cyan (11) fg on black (0) bg reads as "mostly dark with some cyan." Reverse to black fg on light cyan bg with the same `▒`, and it reads as "mostly cyan with some dark." This doubles the effective tonal range of each shade block character, giving you 6 intermediate tones per color pair instead of 3.

**Pipeline application:** Phase 3 shade block selection currently picks the shade density (░/▒/▓) based on luminance ratio. Adding fg/bg swap awareness would let it choose between `▓` with color-A-on-B versus `░` with color-B-on-A, picking whichever better matches the source luminance.

### 8.7 Hard Colors Handling (Halaster)

Some CGA color pairs create ugly visual artifacts when placed adjacent: red (4) next to green (2), magenta (5) next to green (2), brown (6) next to blue (1). These "impossible adjacencies" vibrate visually and distract from the composition. The fix: always insert a neutral buffer cell (black 0, dark gray 8) between clashing color pairs. If the composition requires them adjacent, use a shade block transition (e.g., `▒` with one color as fg and black as bg) to soften the boundary.

**Pipeline application:** Already partially implemented via the `IMPOSSIBLE_PAIRS` set in Phase 3's COLOR_RAMPS. Could be extended with a post-processing pass that scans for remaining impossible adjacencies and inserts buffer cells.

### 8.8 Shape Distortion (Halaster)

Intentionally distort proportions to compensate for CGA's non-square pixel aspect ratio and the viewer's perceptual biases. ANSI cells are taller than wide (roughly 2:1 in most renderers), so circles must be drawn as horizontal ellipses to appear round. Similarly, 45° diagonal lines need different segment lengths horizontally vs. vertically. The technique: always test compositions at actual display size, not zoomed in, because distortion effects are only visible at real scale.

**Pipeline application:** Phase 1 trace already handles aspect ratio correction. A refinement would add per-character aspect ratio compensation — ensuring that diagonal characters (`/`, `\`) are placed accounting for the cell's non-square shape.

### 8.9 Fadey Line Endings (zeroVision)

Never terminate a line or edge abruptly. Instead, fade it out using progressively lighter shade blocks: `█` → `▓` → `▒` → `░` → nothing. This applies to: hair strand endings, clothing edges, shadow boundaries, any organic line that should feel soft. The fade length depends on the line's visual weight — a thick shadow edge gets a 3-4 cell fade, a thin accent line gets 1-2 cells. The principle: hard line endings look mechanical; faded endings look natural.

**Pipeline application:** Phase 5 edge detection could identify line endpoints (cells where an edge run terminates) and apply a 1-2 cell shade fade instead of an abrupt cutoff.

### 8.10 Negative Space as Design Element (enzO/zerostar, zeroVision)

"Pay attention to the black space as much as the grey." Negative space (black or very dark areas) between elements is not empty — it's a deliberate design choice that defines shapes as much as the colored pixels do. Hair strands are defined by the black gaps between them, not just the highlighted strands. Facial features emerge from the shadow shapes between them. The technique: when drawing, periodically invert your mental model — instead of "where do I place color?", ask "where do I place darkness?" Both approaches should produce the same composition.

**Pipeline application:** Phase 2 refinement's outline cleanup pass already preserves black outlines. A "negative space audit" could verify that key negative space regions (between hair strands, around facial features) maintain consistent black/dark-gray cells and haven't been accidentally filled by glow or shade expansion.

### 8.11 Font/Logo Construction (Zeus II, zeroVision, Thrasher)

Building custom fonts and logos in ANSI uses a different technique set than portraiture. Key principles: use box-drawing characters (single `─│┌┐└┘` and double `═║╔╗╚╝`) for clean geometric edges; use full blocks for thick strokes and shade blocks for anti-aliased curves; maintain consistent stroke width across all letterforms; add drop shadows (offset by 1 cell in x and y, using dark gray 8) for depth; use the "1-cell rule" — leave exactly 1 cell of space between parallel strokes for readability. Advanced: create 3D letter effects by using three colors (dark shadow face, medium base face, bright highlight edge) and consistent light direction across all characters.

**Pipeline application:** Not directly applicable to the image conversion pipeline, but relevant when the pipeline output includes text elements or logos that need manual touch-up. The font construction principles could inform a future "text region detection" pass that applies different character selection rules to detected text areas.

## Common Mistakes

1. **Using too many colors at once.** A piece using 6-8 colors well beats one using all 16 poorly. Cohesion matters more than variety.

2. **Flat lighting.** Without shadow/highlight contrast, shapes look 2D. Always establish a light direction.

3. **Ignoring the background.** A solid black background is fine, but leaving the default color (or worse, inconsistent colors) looks unfinished.

4. **Straight-line highlights.** Organic highlight shapes read as natural; geometric lines read as errors.

5. **No anti-aliasing on curves.** Where a bright shape meets a dark area, insert transitional shade characters.

6. **Cramming too much in.** 80x25 is small. Focus on one strong element rather than many weak ones.

7. **Not stepping back.** Zoom out periodically. What reads well at 1:1 is what matters — fine details that only show at 4x zoom are wasted effort.

8. **Wrong shadow hue.** Using dark gray (8) as a universal shadow color desaturates strongly-colored subjects. A blue character with gray shadows looks gray, not blue. Match shadow hue to subject hue.

9. **Tiny features.** At 80x50 pixel resolution, proportionally accurate features become invisible. Eyes that would be 1-2 pixels in realistic proportion need to be 4-5 pixels to read clearly. Always exaggerate.

---
> Source: [sigma-zenko/ansi-skill](https://github.com/sigma-zenko/ansi-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
