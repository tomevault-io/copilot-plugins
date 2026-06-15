## slide-image-to-editable-pptx

> >


# Slide Image → Editable PPTX

You are converting slide screenshots into editable PowerPoint files.
The goal is **visual fidelity** — the output PPT, when rendered, should look
nearly identical to the source image — while making text and geometric shapes
editable.

## Core Principle: Three-Layer Decomposition

Every pixel in the source image belongs to exactly one of three layers:

| Layer | What it contains | How to implement | Editable? |
|-------|-----------------|------------------|-----------|
| **A — Visual Asset** | Complex illustrations, decorative backgrounds, photos, scientific figures, icons with intricate detail, maps, radar scenes, waveforms, heatmaps, campus sketches, device drawings, textured decorations | Clean PNG generated with `$imagegen` (no text!) or original figure preserved as image | Replaceable (move/resize/delete), not internally editable |
| **B — Structure** | Solid color rectangles, rounded rectangles, circles, lines, arrows, dividers, panel frames, card backgrounds, badges, gradient bars, simple geometric decorations | PPT-native shapes via `$slides` / `presentation-skill` (PptxGenJS) | Fully editable |
| **C — Content** | All readable text: titles, subtitles, body text, labels, captions, page numbers, footer text, formula text, table text, card headings, legend labels | PPT-native text boxes via `$slides` / `presentation-skill` (PptxGenJS) | Fully editable |

**The key insight**: Never bake text into a generated image. Never use crude PPT shapes to approximate a complex visual. Never use a full-slide screenshot as background.

## Workflow

### Phase 1: Pixel-Level Analysis

For EACH source image, produce a structured analysis. Do not skip any slide. Do not infer from one slide what another contains — inspect each one.

#### Step 1.1: Observe and catalog

Look at the image carefully. Identify every distinct visual element. For each element, determine:

```
{
  "element_id": "s01_e01",
  "description": "distributed radar network illustration with 6 antennas around a central target on a coordinate grid",
  "bbox_percent": {"x": 45, "y": 10, "w": 55, "h": 80},
  "layer": "A",
  "implementation": "image2-generate",
  "z_order": 1,
  "notes": "Must NOT include any text labels. The title, axis labels, and page number are separate text elements."
}
```

The `bbox_percent` uses percentages of slide width/height (0-100) to specify position. This avoids pixel-count errors across different image resolutions.

#### Step 1.2: Classify elements strictly

Apply these rules in order:

1. **Is it readable text?** → Layer C (text box). This includes ALL text: titles, labels inside diagrams, axis labels, page numbers, footer text, card headings, bullet points, formula text. Even if text overlaps a complex visual, the text itself goes to Layer C.

2. **Is it a simple geometric shape?** (solid rectangle, rounded rectangle, circle, line, arrow, triangle, chevron, trapezoid — with solid fill or simple border) → Layer B (PPT shape). This includes panel backgrounds, card frames, title bars, dividers, badges, progress bars, simple decorative strips.

3. **Is it visually complex?** (illustration, photo, texture, gradient background with imagery, scientific figure, diagram with intricate line work, icon with detail beyond basic geometry) → Layer A (generated PNG or preserved figure).

#### Step 1.3: Identify shared vs. unique elements

- **Shared across slides**: header bar style, footer style, logo, page number format, background color
- **Unique per slide**: main content visuals, specific diagrams, data figures

Create a `slide_master_elements` list for shared elements, and per-slide element lists for unique content.

#### Step 1.4: Completeness self-check (required)

After finishing the element list for ALL slides, go back and re-examine each source image ONE MORE TIME with fresh eyes. This second pass focuses specifically on **small or easy-to-miss visual elements** that the first pass may have overlooked.

For each slide, ask yourself these questions:

1. **Small icons**: Are there any small icons (university seals, bullet-point icons, award badges, folder icons, chip icons, person icons, book icons, etc.) that I classified as Layer B (PPT shape) but are actually too detailed for simple shapes? If the icon has more than 3-4 visual features (gradients, shadows, internal detail, curves), reclassify it as Layer A.

2. **Decorative details**: Are there small decorative elements I skipped entirely? Look in corners, edges, between cards, along dividers, and in footer/header areas. Things like: small wave patterns, dot grids, circuit-trace textures, line-art campus buildings, faint background motifs.

3. **In-card visuals**: For each card/panel on the slide, does it contain a small illustration or diagram inside it? These are frequently missed because the analyst focuses on the card frame (Layer B) and card text (Layer C) but forgets the small visual inside the card.

4. **Chart/figure decorations**: Around charts or data figures, are there small visual elements like target icons, antenna icons, signal-path illustrations, or comparison diagrams that I might have grouped with the chart but are actually separate Layer A elements?

5. **Count check**: Count the total Layer A elements per slide. A visually rich slide typically has 3-8 distinct visual assets. If a complex-looking slide has only 1-2 Layer A elements, something is likely missing — re-examine it.

If this second pass finds any missed elements, add them to the element list with a note: `"found_in": "completeness_check"`. Do NOT remove or modify any existing elements — only add.

### Phase 2: Visual Asset Generation

For each Layer A element, generate a clean PNG using `$imagegen`.

#### `$imagegen` Prompt Rules

1. **Be specific about content**: Don't say "a radar scene." Say "A bird's-eye view of a distributed radar network: 6 satellite dish antennas arranged in a hexagonal pattern around a central target (marked with a star), connected by dashed lines, on a dark blue coordinate grid background with axis markings from -200 to 200 km. The style is technical/scientific, with a dark blue (#0A1628) background and light blue/cyan (#31D7FF) elements. No text, no labels, no numbers, no letters."

2. **Always end with "No text"**: Every prompt must include "No text, no labels, no numbers, no letters anywhere in the image."

3. **Specify exact style**: Reference the color palette, line style, and visual mood from the source image. Example: "Dark technical blueprint style with navy background (#061426), cyan (#31D7FF) glowing lines, subtle grid pattern."

4. **Specify aspect ratio**: Match the aspect ratio of the target bounding box. "Aspect ratio approximately 16:9" or "Aspect ratio approximately 1:1".

5. **Specify transparency when needed**: "Transparent background (PNG with alpha channel)" for overlay elements. "Solid dark background" for background scenes.

6. **One element per image**: Don't combine unrelated visual elements. Generate separate images for separate visual regions.

#### Asset Naming Convention

```
s{slide_number}_{semantic_role}.png
```

Examples:
- `s01_radar_network_scene.png` — cover slide's main radar illustration
- `s01_bottom_wave_decoration.png` — cover slide's bottom decorative wave
- `s03_system_diagram_center.png` — slide 3's central system model illustration
- `s05_rmse_chart_figure.png` — slide 5's RMSE result plot (if preserved as figure)

### Phase 3: PPT Construction

Use the **`$slides` system skill** or the **`presentation-skill`** (both based on PptxGenJS) to build the deck. If a presentation skill is installed, prefer it. Do NOT write raw python-pptx code — use whichever PptxGenJS-based skill is available, which provides bundled helpers, rendering, and validation.

Follow this exact layer stacking order for every slide (add elements in this sequence so z-order is correct):

```
Z-order (bottom to top):
1. Slide background color (set via slide.background)
2. Full-width structural bars (header band, footer band)
3. Large visual assets (background scenes, decorative motifs)
4. Panel/card frame shapes (rounded rectangles, boxes)
5. Smaller visual assets (icons, figures inside panels)
6. Lines, arrows, connectors
7. Native charts and tables
8. All text boxes (titles, labels, body text, captions)
9. Brand elements (logo on top)
```

#### Coordinate Conversion

Source images are typically at a known pixel resolution (e.g., 1920×1080 or 1672×941). Convert to PowerPoint inches using:

```javascript
const SLIDE_W = 13.333; // inches (16:9)
const SLIDE_H = 7.5;

function pxToInches(px_x, px_y, img_w, img_h) {
  return { x: px_x / img_w * SLIDE_W, y: px_y / img_h * SLIDE_H };
}
```

Measure element positions from the source image precisely. Don't guess.

#### Font Handling

- Use "Microsoft YaHei" for Chinese text
- Use "Cambria Math" for mathematical formulas
- Match font size, weight (bold), and color from the source image
- Default to these approximations if unsure:
  - Main title: 28-32pt bold
  - Section title: 20-24pt bold
  - Card heading: 14-16pt bold
  - Body text: 10-12pt regular
  - Caption/label: 8-10pt regular
  - Page number: 10-12pt

#### Color Extraction

Extract exact colors from the source image. Provide hex values. Build a color palette object at the top of the script:

```javascript
const COLORS = {
  bg_dark: "0A1628",
  accent_cyan: "31D7FF",
  accent_gold: "C59A4A",
  text_white: "F2FBFF",
  text_muted: "A9C6D8",
  // ... etc
};
```

#### Rendering and Validation

After building each slide (or the full deck), use the `$slides` built-in render and validation scripts to:
- Render each slide to PNG for visual comparison
- Check for text overflow
- Check for element overlap issues
- Check for font substitution problems

Build one slide at a time. Render and visually compare against the source image before proceeding to the next slide. Fix any significant layout differences before moving on.

### Phase 4: Validation

After building the PPTX:

1. **Structural check**: Count objects per slide. Each slide should have:
   - Multiple text frames (not zero)
   - Multiple shapes (not just one big picture)
   - No picture covering >85% of slide area (that would be a full-slide screenshot)

2. **Visual comparison**: If possible, render the PPTX to images and compare with source. Flag any slide where:
   - Major elements are missing
   - Layout is significantly different
   - Text is baked into images instead of text boxes

3. **Editability check**: Verify that opening the PPTX and clicking on text allows editing.

## Non-Negotiable Rules

1. **NEVER** use a full-slide screenshot as the background
2. **NEVER** bake readable text into generated images
3. **NEVER** use a single generic visual motif across many slides when the source slides have distinct visuals
4. **NEVER** merge unrelated elements (text + visual + shapes) into one image
5. **NEVER** skip the analysis phase — inspect every source image before writing any code
6. **NEVER** use crude PPT shapes to approximate complex illustrations when `$imagegen` can generate a proper visual
7. **ALWAYS** generate each visual asset without any text
8. **ALWAYS** match the source image's color palette, not a generic theme
9. **ALWAYS** place text as PPT-native text boxes, even if it overlaps a visual asset
10. **ALWAYS** process slides one at a time — each slide gets its own analysis, assets, and build function

## Common Failure Modes to Avoid

### Failure: "New template" instead of "reconstruction"
**Symptom**: The output looks like a professional PPT on the same topic, but doesn't match the source image's specific layout.
**Cause**: Skipping detailed analysis, using generic layouts.
**Fix**: Measure exact positions from the source image. Match the source's specific element arrangement.

### Failure: "Generic motif spam"
**Symptom**: The same background image appears on every slide.
**Cause**: Generating one or few generic background images and reusing them.
**Fix**: Each slide's visual assets should be unique and match that specific slide's source image content.

### Failure: "Over-native crude shapes"
**Symptom**: Complex illustrations (radar dishes, waveforms, network diagrams) rendered as ugly PPT shape approximations.
**Cause**: Trying to rebuild everything with native shapes when IMAGE2 would produce a much better result.
**Fix**: Use `$imagegen` for any visual element that PPT shapes can't faithfully reproduce.

### Failure: "Baked text"
**Symptom**: Text visible in the PPTX but not editable — it's part of an image.
**Cause**: Generated images include text that should be editable.
**Fix**: `$imagegen` prompts must always specify "no t

---
> Source: [w1163222589-coder/slide-image-to-editable-pptx](https://github.com/w1163222589-coder/slide-image-to-editable-pptx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-15 -->
