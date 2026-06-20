## designing-better-maps-skills

> Guide map design from scratch using established cartographic principles (color, projection, type, symbols, visual hierarchy, layout). Use this whenever the user is making or planning a map or map figure and needs design decisions — choosing a color scheme, picking a projection, deciding class breaks, setting up labels, or laying out a figure for a paper, poster, slide, or web. Trigger it even when the user just says "make a map of X," "what colors for this map," "how should I map this data," or asks for cartographic feedback, and even when they don't say the word "cartography." Especially relevant for publication-quality figures where grayscale-robustness and colorblind-safety matter.


# Designing Better Maps

Guide the user through map design as a sequence of dependent decisions. Each decision constrains the next: you cannot choose a color scheme until you know the data's measurement level, and you cannot choose a projection until you know the map's purpose and extent. Work the chain in order. Resist the urge to jump straight to "here's a nice color palette" — a good palette on the wrong scheme type is still a bad map.

The goal is a concrete **design spec** the user can hand to their plotting code (GEE, matplotlib, R, QGIS, ArcGIS), not finished pixels. End every session with that spec.

## Critique mode — if you have an existing map

When the user shares an existing map rather than starting fresh, run a rapid audit before proposing changes. Work the same decision chain, but as a checklist:

1. **Map type** — does the type match the data's measurement level? Flag: counts on a choropleth, qualitative scheme on ordered data, symbols not scaled to area.
2. **Color** — does the scheme type match the data? Flag: rainbow/jet for sequence, ternary RGB when a simpler encoding would serve, wrong diverging midpoint, red–green pair on a colorblind-sensitive figure.
3. **Symbols** — are sizes area-scaled? Is the range wide enough to discriminate? Is overplotting masking the data?
4. **Projection** — is it equal-area for any comparison of quantities across areas? Appropriate compromise for the extent?
5. **Visual hierarchy** — is the thematic content dominant? Is the basemap quiet enough? Does the most important layer have the highest contrast?
6. **Type** — is there a legible size hierarchy? Are all labels above the minimum legibility floor for the medium?
7. **Layout** — is every marginalia element earning its place? Is the legend readable and correctly ordered?

Name each problem with a one-sentence diagnosis and a one-sentence fix. Don't recount what the map already does right unless it's non-obvious. Prioritize: color scheme errors first (they mislead the data), then symbol errors, then hierarchy and type, then layout polish.

## Step 0 — Get the brief before recommending anything

A map's design follows from its job. Elicit these before proposing decisions; if the user already stated some, don't re-ask. Keep it to the few that actually change the design:

- **Purpose / message** — What is the one thing a reader should take away? A map that tries to say everything says nothing. Push for a single dominant message; everything else is supporting context.
- **Audience & medium** — Print (paper figure, poster) vs screen (slide, web, interactive)? Print needs higher contrast and can't rely on zoom; screen tolerates more layers. This sets resolution, type size floors, and detail budget.
- **Data** — What variable(s), and what is each variable's **measurement level**? This is the single most consequential fact for color and symbol choice. Sort each into: nominal/categorical, ordinal, or numeric (interval/ratio), and note whether numeric data has a **meaningful midpoint** (e.g. anomaly, change, divergence from a reference).
- **Geographic extent & location** — Continental, national, city, neighborhood? Governs projection and how much generalization the basemap needs.

If the user is vague on purpose, that is the problem to fix first — most weak maps are weak because they have no decided message, not because of palette.

## The decision chain

Work top to bottom. For the bulky lookups, read the referenced file only when you reach that decision.

### 1. Map type (follows from data measurement level)

Match the thematic map type to what the data *is*. Common pairings:

- **Nominal / categorical area data** → categorical fill (one hue per class). Not a sequence.
- **Numeric data tied to areas, normalized** → **choropleth**. Critical rule: choropleth requires data normalized to area or population (rates, densities, per-capita), never raw counts — raw counts on a choropleth just re-draw the population/area map. If the user has raw counts, either normalize or switch to proportional symbols.
- **Numeric data, raw magnitude / counts** → **proportional (graduated) symbols** scaled by value, or dot density.
- **Continuous surfaces (raster, interpolation)** → isarithmic / continuous fill (this is the usual case for remote-sensing and climate fields — LST, NDVI, precipitation).
- **Flows / movement** → flow lines weighted by magnitude.

State the chosen type and *why the data forced it*. If the user asked for a choropleth of counts, flag the normalization trap explicitly.

### 2. Classification (only for class-based maps)

If using classed data (choropleth, classed symbols), decide the class scheme and count:

- **Number of classes**: 3–7 is the readable range for most maps. More classes carry more information but become indistinguishable, especially in print or grayscale. Default to 5 unless there's reason otherwise.
- **Class breaks method**: match to data distribution and message.
  - *Quantile* — equal counts per class; good for ordinal comparison, but can hide/inflate by grouping unlike values.
  - *Equal interval* — equal value ranges; intuitive, good for evenly spread data, wasteful on skewed data.
  - *Natural breaks (Jenks)* — minimizes within-class variance; good default for clustered/skewed real-world data.
  - *Standard deviation* — for showing distance from the mean; pairs naturally with a diverging scheme.
  - *Manual / meaningful* — when breaks have domain meaning (regulatory thresholds, zero).
- Consider **unclassed (continuous)** for screen/interactive or when preserving fine variation matters more than legend simplicity.

Name the method and the rationale tied to the data's distribution and the message.

### 3. Color (the decision people get wrong most often)

Color scheme **type** must match data type. This is the core of the whole skill. Read `references/color-schemes.md` for the full selection logic, named palettes, class-count notes, and tool hooks (matplotlib/R/GEE). The short version:

- **Sequential** (light→dark, low→high) for ordered numeric data with no meaningful midpoint.
- **Diverging** (two hues, light/neutral center) for data with a meaningful midpoint — change, anomaly, deviation. This is where standard-deviation breaks belong.
- **Qualitative** (distinct hues, equal lightness) for nominal categories. Never use a sequence for categories — it implies an order that isn't there.

### Multivariate, bivariate, and ternary color encoding

When a map encodes **two or three simultaneous numeric variables**, standard single-variable schemes don't apply. These approaches exist, in order of increasing cognitive cost:

- **Dominant-factor + saturation** (recommended default): extract *which factor dominates* (nominal → hue, colorblind-safe qualitative palette) and *how strongly it dominates* (saturation or lightness). A city that is 60/30/10% reads as the dominant factor's hue at medium saturation; 90/5/5% reads as the same hue at high saturation. Mixed cities near the ternary center become desaturated neutrals — honest and readable. Works with standard qualitative palettes.
- **Bivariate choropleth**: two sequential ramps combined on a 2×2 or 3×3 grid. Powerful when the *interaction* between two variables is the message, but hard to read without a 2D legend; limit to 9 cells (3×3 max).
- **RGB / ternary color mixing**: three component proportions mapped to R, G, B channels (common in geology, petrology, remote sensing). Serious problems: (1) colors near the ternary center are gray/desaturated and indistinguishable; (2) the red–green axes are the exact pair lost in deuteranopia — fails colorblind safety by design; (3) the 2D triangular legend is among the hardest to read in cartography. Use only for specialist audiences trained in ternary diagrams.

**Decision rule by audience and purpose**:
- *General audience / journal figure for non-specialist readers*: prefer dominant-factor + saturation — the hue is immediately interpretable without training.
- *Specialist / expert audience where the full gradient of factor combinations is the message* (e.g. a geochemistry, remote sensing, or multi-factor priority map read by domain experts): RGB ternary can be retained. The spectrum of in-between colours carries real information that discrete categories lose. In this case: (1) flag the colorblind limitation explicitly in the figure caption; (2) keep the ternary legend at a readable size with clearly labelled corners; (3) consider adding one or two annotated anchor cities/points to help readers calibrate the colour space.
- When the gradient itself is not the message — when the reader only needs to know *which* factor drives each location — prefer the simpler encoding regardless of audience.

**The hue-normalization magnitude trap, and the hue × opacity fix.** RGB/ternary composites are almost always *hue-normalized* (each point's colour divided by its max channel) so that composition reads regardless of magnitude. The hidden cost: this throws away total magnitude entirely — a location weak on all factors looks just as vivid as a strong one, so the map shows *why* each place ranks but not *which places rank highest*. The fix is a **two-channel encoding**: keep **hue = composition** (the normalized RGB) and add a **second visual variable = total magnitude** — opacity or lightness, computed from the un-normalized combined value (vector norm, sum, or better, a model output that physically combines the factors). On a dark canvas, opacity-as-magnitude is especially effective: low-magnitude points fade into the background and high-magnitude points render opaque and pop, so the map answers "where is it strongest" and "why" simultaneously. Implement with a per-point RGBA array (hue in RGB, magnitude in A); give it its own opacity-ramp legend alongside the composition legend. The same logic restores magnitude to any normalized multivariate scheme.

Always check two robustness conditions for any map that might be printed or read by anyone: **colorblind-safety** and **grayscale legibility** (does it survive being photocopied / printed B&W / shown on a bad projector?). Sequential and good diverging schemes pass grayscale because lightness carries the signal; qualitative schemes usually fail grayscale, which is fine on screen but a problem for print categories — note it.

### 4. Symbols & visual variables

Encode each variable with the visual variable that suits its measurement level (Bertin's logic):

- **Quantity / magnitude** → size (proportional symbols) or lightness/value (choropleth). Size reads as "how much"; hue does not.
- **Order** → lightness/value, or size.
- **Category / difference in kind** → hue or shape. These read as "different from," not "more than."

Scale proportional symbols by **area, not radius** (or use a perceptual scaling) so a value twice as large looks twice as large, not four times. Keep symbol counts and the symbol legend honest with a few representative sizes.

**Dense point data and overplotting**

When many point symbols overlap, overlapping markers hide spatial structure and make size/color comparisons unreliable. Decision rule by count and density:

- **< ~500 points, moderate density**: alpha transparency (0.4–0.7) usually sufficient; overlap is evident but the pattern readable.
- **500–5,000 points, global/continental extent**: alpha + a size floor so the smallest symbols stay visible; plot highest-value points last (on top) so important symbols aren't buried.
- **> ~5,000 points, or dense clusters dominate**: switch to a **binned representation** — hexagonal bins, kernel density surface, or dot-density raster — to show spatial density rather than individual points. Retain individual symbols only for labeled top-N features.
- **When the message is "which specific features rank highest"**: keep individual symbols but cap the plotted set to a defensible top-N (e.g., top 500) and note the cutoff explicitly in the caption.

Never silently overlap symbols of different sizes — a large symbol buried under small ones is invisible and misleading.

For detailed symbol design (point/line/area treatment, halos, hierarchy of symbol weight), see `references/type-and-symbols.md`.

### 5. Projection & scale

Choose projection from **purpose + extent + the property that must be preserved**. Full selection table in `references/projections.md`. The decisive rule:

- **Any statistical / thematic map** (choropleth, density, anything where area comparison matters) → **equal-area** projection. A non-equal-area projection silently distorts the very quantities a thematic map is comparing. This is the most common projection error in published figures.
- **Navigation / direction / web tiles** → conformal (Mercator family) — but never for area comparison.
- Small extents (city/neighborhood) → local projected CRS (UTM zone, national grid); projection choice matters little for distortion but matters for using meters.

Include a scale indicator appropriate to medium: scale bar (survives resizing — prefer for digital and anything that may be rescaled), or representative fraction only when print size is fixed. Note projection in the caption for any published figure.

### 6. Visual hierarchy & figure-ground

Decide what should dominate and recede *before* styling. The thematic content (the message) sits on top; basemap and context recede. Tools for hierarchy:

- **Contrast** — the most important layer gets the most contrast against its surroundings; context layers are muted (low saturation, grayed, thinner).
- **Figure-ground** — make the area of interest read as "figure" against a quiet "ground": crisp boundary, slight value difference, optional muting/desaturating of everything outside the study area.
- **Detail budget** — strip basemap detail that doesn't serve the message. Every line competes for attention.

A reader should know within a second where to look. If everything is equally bold, nothing is.

**Basemap design**

The basemap is the ground; it must recede. Standard choices for print figures:

- **Land fill**: light neutral gray (#f0f0f0–#e8e8e8) or very light warm tan (#f5f0e8). Pure white makes country boundaries invisible; saturated fills compete with the thematic layer.
- **Ocean/water**: a very light desaturated blue (#dde8f0) gives subtle figure-ground contrast without competing with blue thematic data. Leaving the ocean white (matching page background) is also clean for journal figures.
- **Country/admin boundaries**: 0.2–0.4 pt, mid-gray (#a0a0a0). If boundaries aren't part of the message, reduce them to minimum visibility or drop below a size threshold.
- **Coastlines**: slightly heavier than admin boundaries (0.4–0.6 pt) since they define land/water; still subordinate to the thematic layer.
- **Detail budget**: include only the admin level the map's scale and message require. A world map does not need state/province outlines unless they're the unit of analysis; a city map doesn't need country outlines.
- **Tile basemaps (screen/interactive)**: choose a muted/grayscale tile style (Carto Positron, Stamen Toner Lite, Mapbox Light) and optionally reduce opacity to 70–85% to keep it subordinate.

**Light vs dark basemap.** Default to a light basemap for print and for dark/saturated thematic layers (dark dots on light land is the classic, safe figure-ground). But switch to a **dark basemap when the thematic layer is luminous or saturated point data — especially when the brightest points are the most important** (e.g. a glowing-dot map where opacity or brightness encodes magnitude). A dark canvas maximizes figure-ground contrast for bright marks, where a light basemap would mute them. Construction that works: keep the ocean/background the *darkest* element (near-black), lift the land just enough to read continents (land clearly lighter than ocean, e.g. dark grey on near-black, with a faint coastline), drop dot outlines (dark outlines vanish and black ones look dirty on dark fill), and let the data be the only bright thing on the page. Carto "Dark Matter" is the reference style. Tradeoff to weigh: dark figures cost more ink in print and can look different on projectors — confirm the medium before committing.

### 7. Type & labels

Type is a design element, not an afterthought; it often makes or breaks figure polish. See `references/type-and-symbols.md` for placement conventions and the hierarchy. Essentials:

- **Hierarchy by size/weight**: title > region labels > feature labels, with clear, limited steps. Two type families at most (commonly one serif/one sans), or one family with weight variation.
- **Conventions readers expect**: italics for hydrography/water features; label placement that hugs its feature (point labels upper-right by default, then resolve collisions); curved labels along linear features; avoid labels crossing boundaries or other labels.
- **Legibility floors**: set a minimum type size for the medium (print figures need a real floor — tiny labels vanish at journal column width). Add halos/casings only where labels sit on busy fill, and keep them subtle.

### 8. Layout & marginalia

Assemble the frame. Include only elements that do work:

- **Title** — present and informative when the map stands alone; often omitted/handled by the caption for an in-text journal figure (don't double up with the figure caption).
- **Legend** — explains every symbol/scheme used; order it to match the data's logic (sequential legends low→high); label units. Cut legend entries for anything self-evident.
- **Scale bar** — include for any map where distance matters; prefer over stated scale for digital.
- **North arrow** — include only when orientation is non-obvious or non-north-up. On a standard north-up regional map it's clutter; omit it.
- **Credits / source / projection / date** — small, in a corner; essential for published and standalone maps.
- **Balance** — distribute visual weight; avoid a heavy element marooning empty space. Align elements to an implicit grid.
- **Legends inside vs outside the map frame** — when the map has large empty regions (open ocean, uninhabited polar areas, sparse continents), place legends *inside* the map frame in those empty zones rather than in a separate side panel. This keeps the full figure width for data, avoids wasting whitespace, and keeps the reader's eye near the relevant area. Draw legends at map data coordinates so they stay anchored to a geographic region. Reserve a separate side panel only when the map is uniformly dense and there is no safe empty zone to occupy.

## Output: the design spec

Close by writing the decisions back as a compact spec the user can implement. Use this shape:

```
MAP DESIGN SPEC — <title/purpose>
Message:        <the one takeaway>
Medium/size:    <print|screen, dimensions, dpi>
Map type:       <choropleth|prop. symbol|continuous|...>  (because <data reason>)
Classification: <method, N classes>  | or "unclassed"
Color scheme:   <sequential|diverging|qualitative> — <named palette>
                colorblind-safe: <y/n>   grayscale-legible: <y/n>
Symbols:        <visual variable → variable mapping; scaling note>
Projection:     <name / EPSG>  (equal-area: <y/n>, because <reason>)
Hierarchy:      <what dominates; how context recedes>
Type:           <families, hierarchy, conventions applied>
Layout:         <which marginalia in/out; why>
Tool hook:      <matplotlib cmap / R palette / GEE viz params, if useful>
```

If the user wants, translate the spec into starter code for their tool — but the spec is the deliverable; code is optional.

## Working style

- Ask the brief questions together, not one at a time, then proceed through the chain making recommendations the user can veto.
- Always give the *reason* a decision follows from the data or purpose — the user should learn the logic, not just receive a palette.
- When the user's stated wish conflicts with a principle (counts on a choropleth, rainbow scheme for sequence, non-equal-area for a density map), say so plainly and offer the fix; don't silently comply.
- Calibrate detail to the user's expertise. A GIS-fluent user wants the decisive rule and the EPSG code, not a tutorial.

---
> Source: [kangning-huang/designing-better-maps-skills](https://github.com/kangning-huang/designing-better-maps-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
