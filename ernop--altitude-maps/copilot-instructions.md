## altitude-maps

> UI components, axes indicator, and visual design patterns


# UI Essentials

## Region Selector
- Three groups in order: Countries -> Areas -> US States
- Visible HR-style divider between groups
- Alphabetical within each group
- Non-selectable dividers not focusable

## Text & Buttons
- Primary HUD/meta/status: White text on dark backgrounds
- All buttons: White text (#ffffff) in normal and hover states
- On hover: Change background only, never text color
- Smooth transitions (0.2s)

## HUD Overlay
- Elevation/slope/aspect data in bottom-left corner
- Hidden by default; toggle via "Show HUD Overlay" checkbox
- Draggable; position persists in localStorage
- Minimize button (-) in top-right collapses and unchecks checkbox
- Settings button opens config panel
- Tabular layout prevents value jitter

## Controls Pattern
- Conceptual labels ("Resolution" not "Bucket Size")
- MAX/DEFAULT quick access buttons
- Bidirectional controls: Two columns (HIGHER DETAIL vs LOWER DETAIL)
- Color-coded headers (green = higher detail, orange = lower detail)
- Vertical stacking within columns

## Flat Lighting
- Checkbox "Flat lighting (no shading)" under Color Scheme
- Default off; URL `flat=1` turns it on
- Uses unlit materials and ambient-only lighting

## Axes Indicator
3D helper showing world orientation.

### Coordinate System (Roblox Convention)
- +X = right (red arrow)
- +Y = up (green arrow)
- +Z = back/toward camera (blue arrow)

### Rotation Mapping (Roblox BasePart.Orientation)
- X-axis rotation = Pitch
- Y-axis rotation = Yaw
- Z-axis rotation = Roll
- Order: X (pitch), then Y (yaw), then Z (roll)

### Position & Behavior
- Fixed screen-space: Bottom middle (8% from bottom, centered)
- World-space object rotates with terrain
- Rotates with `terrainGroup.quaternion`

### Visual Design
- Origin: White sphere (radius: `size * 0.18`)
- Arrows: 2/3 length (`size * 0.333`), start outside sphere
- Arrowheads: 60% of arrow length
- Negative directions: Gray semi-transparent lines (no arrows)
- Colors: RGB = XYZ (X=Red, Y=Green, Z=Blue)

### Labels
- X/Y/Z: Large (`size * 1.5`), gap after arrow tip
- Pitch/Yaw/Roll: Match axis colors, positioned after X/Y/Z labels

## Color Schemes
- `auto-stretch`: Percentile bounds computed client-side
- Global Scale toggle for cross-region comparison
- Located below "Show Scale" checkbox
- Applies to elevation-based schemes, not slope/aspect

## Removed UI
- Shading controls (sun/azimuth/altitude/contrast)
- Water simulation UI
- Contour lines (no controls, no rendering)
- Region navigation arrows (hidden by default)
- Render mode selector (hidden; bars only)

## Recent Regions
- Max 12 items, most recent first
- On region load, move to front (auto-deduplicates)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ernop)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/ernop)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
