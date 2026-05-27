## design-system

> Design system reference for Urban Tech Creative website. Covers utilities, atoms, molecules, tokens, grid system, composition patterns, and design philosophy.


# UTC Design System

## Documentation Structure

- **`.cursor/rules/design-system.mdc`** (this file): Operational reference. Conventions, tokens, component APIs, composition patterns. Automatically available when editing `components/**` or `app/**`. This is the source of truth for "how to use things".
- **`docs/lld/*.md`**: Deep-dive implementation docs. Why things work the way they do, motion models, performance rationale, edge cases. The source of truth for "how things work internally". Read these when modifying a specific subsystem.

| Topic | Quick reference | Deep dive |
|-------|----------------|-----------|
| Design system | This file | -- |
| Cube (3D interaction) | Cube Face Mapping (below) | `docs/lld/cube.md` |
| Frame (semantic use) | Atoms section (below) | `docs/lld/frame.md` |
| UIGrid (layout grid) | Grid System section (below) | `docs/lld/ui-grid.md` |

---

## Atomic Design Hierarchy

### Utilities

- **Pressable** (`components/Pressable/Pressable.tsx`): Makes any subtree interactive and accessible. Zero visual opinion — no borders, backgrounds, padding, or hover effects. Renders `<Link>` for internal navigation (`href` starts with `/`), `<a target="_blank" rel="noopener noreferrer">` for external URLs (`href` starts with `http`), or `<button type="button">` for actions (no `href`). Props: `href?`, `onClick?`, `onMouseEnter?`, `onMouseLeave?`, `children`, `className?`, `aria-label?`, `aria-expanded?`, `aria-haspopup?` (for menu triggers), `data-testid?`. Use when wrapping custom compositions (Logo lockups, NavLinks) where the visual treatment is owned by the children.
- **NavMenu** (`components/Nav/NavMenu.tsx`): Handles show/hide and positioning of a dropdown menu. Renders a trigger (supplied by parent) and, when open, a panel positioned below it and right-aligned (`absolute top-full right-0`). Closes on click outside or Escape key. No visual opinion on panel content — parent provides trigger and panel (e.g. Frame + NavList). Props: `open`, `onClose`, `trigger` (ReactNode), `children` (panel content), `className?`, `panelClassName?`. Use for primary nav dropdown or any similar floating menu.
- **Icon**: Icons from `@phosphor-icons/react`. Prefer the **fill** variant for the acid design aesthetic — bold, solid shapes with strong silhouettes. Fall back to regular weight for secondary/supporting icons.

### Atoms (primitives)

- **Frame** (`components/Frame/Frame.tsx`): Post-neobrutalist bordered container. A self-contained piece of UI or content. Thick black borders (4px default). Selective rounded corners add friendliness — use sparingly (one curve is the sweet spot). Props: `borderSides` (omit sides to prevent double-borders when stacking adjacent Frames), `roundedCorners`, `borderWidth` (`border-2` | `border-4`). Purely presentational — no hover or interactive behaviour.
- **Accent** (`components/Accent/Accent.tsx`): Gradient bar placed alongside a Frame for visual emphasis. Draws the user's eye using colour. Gradients: `magenta-green`, `purple-orange`, `orange-purple`. Props: `direction` (`vertical` | `horizontal`), `gradient`, `borderSides` (match adjacent Frame to avoid double-borders). Purely presentational — no hover or interactive behaviour.
- **Heading** (`components/Heading.tsx`): Semantic heading component (h1-h6). Predefined size/weight classes per level. Uses Recursive font.
- **UIGrid** (`components/UIGrid/UIGrid.tsx`): Site-wide layout grid with square cells. Uses ResizeObserver to compute cell size so cells never stretch or squash. Use for LCARS-style panel positioning. Pair with **GridBlock** (`components/UIGrid/GridBlock.tsx`) for placement (col, row, colSpan, rowSpan). NOT for cube faces — use FaceGrid for those.
- **Overlay** (`components/Overlay/Overlay.tsx`): Full-viewport overlay with backdrop. Renders via portal (`document.body`). Props: `open` (boolean), `onClose` (backdrop click or Escape), `children`, `className`. CSS keyframe animations for fade + scale entrance. Handles focus management (auto-focus on mount, restore on close) and locks body scroll while open. Use as the base layer for any modal, detail panel, or dialog.

### Molecules (combinations of atoms)

- **Breadcrumbs** (`components/Breadcrumbs/Breadcrumbs.tsx`): Frame (theme-black bg, top-right curve, no border) + breadcrumb nav. Chevron separators, white text, cyan hover. Props: `items` (`{ label, path?, current? }[]`). Use for hierarchical page context (e.g. Work › Afghan Project).
- **Button** (`components/Button/Button.tsx`): Opinionated, self-contained interactive button with three semantic variants. Renders its own border, background, hover, and focus styling — no Frame needed. Uses `Pressable` internally for element selection. Props: `variant` (`"primary"` | `"secondary"` | `"tertiary"`), `label?` (button text), `icon?` (IconName), `iconOnly?` (boolean), `href?`, `onClick?`, `aria-label?`, `aria-expanded?`, `aria-haspopup?` (for menu triggers), `className?`. Visual treatment: **Primary** = black bg, white text, `rounded-br-2xl`, hover cyan (main action). **Secondary** = transparent bg, black text/border, `rounded-bl-2xl`, hover cyan (supporting action). **Tertiary** = no border/bg, black text, hover cyan bg (low-emphasis action, still clearly pressable). **Active (all)** = `translate-y-1` press-down shift (100ms transition). **Focus (all)** = 4px magenta outline, 2px offset.
- **NavLink** (`components/Nav/NavLink.tsx`): Pressable wrapping one Frame that contains icon + label + arrow (layout only; no inner Frames). Optional `frameBorderSides?` (from Frame) — when omitted, Frame has all four borders; NavList sets it when `inPanel` to avoid double strokes. Hover styling via `group-hover:` on the Pressable wrapper; whole row goes cyan on hover.
- **Logo lockup** (`components/Logo/Logo.tsx`): Frame + Accent + Logomark. The logomark itself is a real 3D CSS cube showing the U/T/C grid patterns. When clickable, wrapped in Pressable with `group-hover:` for cyan background transition.

### Organisms (groups of molecules)

- **SiteHeader** (`components/SiteHeader/`): Logo lockup (Pressable + Frame) and PrimaryNav in the top-right. Single layout at all viewports; tap "Navigation" to open the dropdown.
- **PrimaryNav** (`components/Nav/PrimaryNav.tsx`): Floating primary navigation in the top-right, intended to live inside the site header. Composes NavMenu (show/hide, positioning) + Button trigger ("Navigation", arrow-down icon) + NavList inside a Frame. Toggle opens/closes the dropdown; click outside or Escape closes. Props: `className?`, `defaultOpen?` (e.g. for Storybook). Uses `primaryNavLinks` from `components/Nav/primaryNavLinks.ts`.
- **NavList** (`components/Nav/NavList.tsx`): Vertical list of NavLinks. Owns the `<nav>` landmark; NavLink is purely presentational. No chrome — decoration (e.g. Frame) is the parent's job. Props: `links` (NavLinkItem[], type from `components/Nav/types.ts`), `size` (`"mobile"` | `"desktop"`), `align?` (`"left"` | `"right"`), `inPanel?` (when true, computes `frameBorderSides` per row for single-stroke stacking), `onLinkClick?`. Use inside PrimaryNav, NavMenuPanel, or any custom nav container.
- **Cube** (`components/Cube/Cube.tsx`): Interactive 3D CSS cube on the homepage. Auto-spins, drag-to-rotate, tap-to-navigate. Each face maps to a site section. Accepts `onFaceTap?: (face: FacePosition) => void` prop; defaults to an alert. Exports `FacePosition` type and `FACE_LABELS` map.
- **SectionDetail** (`components/SectionDetail/SectionDetail.tsx`): Detail overlay shown when a cube face is tapped. Renders section-specific content (title, description, CTA link) inside an Overlay with Accent + Frame. Uses the Button molecule for actions (`variant="primary"` for Explore, `variant="secondary"` for Close). Props: `face: FacePosition | null` (null = closed), `onClose`.

---

## Design Tokens

### Colors

| Token | Hex | Tailwind class | Usage |
|-------|-----|----------------|-------|
| `--theme-cyan` | #68E3E8 | `text-theme-cyan`, `bg-theme-cyan` | Primary accent, interactive highlights, hover states |
| `--theme-orange` | #FBB006 | `text-theme-orange`, `bg-theme-orange` | Warmth, CTAs |
| `--theme-purple` | #443BFF | `text-theme-purple`, `bg-theme-purple` | Depth, tech feel |
| `--theme-black` | #012031 | `text-theme-black`, `bg-theme-black` | Backgrounds, borders, text |
| `--theme-magenta` | #F849C1 | `text-theme-magenta`, `bg-theme-magenta` | Energy, accent gradients, focus rings |
| `--theme-green` | #32CD32 | `text-theme-green`, `bg-theme-green` | Growth, accent gradients |
| `--theme-white` | #FFF | `text-theme-white`, `bg-theme-white` | Light backgrounds, text on dark |

### Backgrounds

| Token | Hex | Usage |
|-------|-----|-------|
| `--background-faded-cyan` | #E4F1F1 | Subtle page background (right side of gradient) |
| `--background-faded-orange` | #F3EBDA | Subtle page background (left side of gradient) |

Body background: `linear-gradient(to right, faded-orange, faded-cyan)`.
HTML background: `--theme-black` (visible behind body, e.g. on overscroll).

### Typography

- **Font**: Recursive (variable, axes: CASL, CRSV, MONO, slnt, wght 300-1000)
- **CSS variable**: `--font-recursive`
- **Heading sizes**: h1 = `text-4xl font-black`, h2 = `text-3xl font-extrabold`, h3 = `text-2xl font-bold`, h4-h6 = progressively smaller

### Spacing & Borders

- **Border width**: `border-4` (default), `border-2` (lighter contexts)
- **Border radius**: `rounded-3xl` for Frame curved corners
- **Grid gap**: `4px` for tight/stacked elements (e.g. adjacent Frames in a series), `8px` for separated elements

---

## Grid System

### 3x3 Master Grid

Aligns with the logomark cube faces. Each cell represents one conceptual "block". Used in the Logo Cube, where each face shows a 3x3 letter pattern (U, T, C).

### 6x6 Sub-Grid

Each master cell subdivides into 2x2, giving 36 total cells. Two uses:

1. **Icon grids** (`components/Grids/Grid6x6.tsx`): Renders pixel-art icons using opaque/clear squares. Grid patterns defined in `components/Grids/patterns.ts`. Used for section icons in navigation.
2. **Face layout grids** (`components/Cube/Faces/FaceGrid.tsx`): Layout grid for placing arbitrary content on cube faces. Same 6x6 geometry, but children are positioned with Tailwind grid classes (`col-start-*`, `row-start-*`, `col-span-*`, `row-span-*`). Supports absolute overlay layers for blend-mode experiments.

### UIGrid (site-wide layout)

`components/UIGrid/UIGrid.tsx` — Site-wide layout grid where every cell is a perfect square. Uses ResizeObserver to compute cell size from available space: `min((w - gaps) / cols, (h - gaps) / rows)`. Grid is centered in the container. Use `GridBlock` to place children at specific grid positions.

Intended for site-wide UI positioning — LCARS-style panel layouts with Frames, Accents, NavLinks. Supports any `cols × rows` configuration. For performance and responsive design details, see `docs/lld/ui-grid.md`.

#### UIGrid API

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `cols` | `number` | required | Number of columns |
| `rows` | `number` | required | Number of rows |
| `gap` | `number` | `4` | Gap between cells in pixels |
| `fullViewport` | `boolean` | `true` | Fill viewport or fill parent |
| `className` | `string` | -- | Optional class for outer wrapper |

#### GridBlock API

| Prop | Type | Default | Description |
|------|------|---------|-------------|
| `col` | `number` | required | Column index (1-based) |
| `row` | `number` | required | Row index (1-based) |
| `colSpan` | `number` | `1` | Columns to span |
| `rowSpan` | `number` | `1` | Rows to span |
| `className` | `string` | -- | Optional class for block wrapper |

### FaceGrid Usage

```tsx
<FaceGrid>
  {/* Grid layer: content placed on the 6x6 grid */}
  <div className="absolute inset-0 grid grid-cols-6 grid-rows-6 gap-1 p-1">
    <div className="col-start-1 row-start-1 col-span-4 row-span-2 flex items-end">
      <h2 className="text-theme-cyan font-black">XR</h2>
    </div>
  </div>
  {/* Overlay layer: stacked on top for effects */}
  <div className="absolute inset-0 mix-blend-screen pointer-events-none">
    ...
  </div>
</FaceGrid>
```

---

## Composition Patterns

### Double-Border Prevention

When two Frames are adjacent, omit the shared border on one side. Example: logo lockup — icon Frame has all 4 borders, text Frame omits `left` so they share a single border line.

### Hover with group-hover

For custom compositions (Logo lockups, NavLinks) that need hover effects, wrap in `Pressable` with `className="group"` and apply `group-hover:` utilities to child elements. Example: `group-hover:bg-theme-cyan transition-colors duration-200` on the Frame inside a Pressable logo lockup.

### Focus Ring

All interactive elements use a magenta focus ring: `focus-visible:outline-4 focus-visible:outline-[var(--theme-magenta)] focus-visible:outline-offset-2`. This is built into Button. For Pressable compositions, add it to the Pressable's `className`.

### Layering on Cube Faces

FaceGrid uses `position: relative` with `overflow: hidden`. Stack content by using multiple `absolute inset-0` children. Use `mix-blend-mode` and `opacity` for visual effects. Later layers sit on top of earlier ones.

---

## Cube Face Mapping

| Position | Section | Route | Description |
|----------|---------|-------|-------------|
| top | XR: Extended Reality | /xr | AR, VR, MR |
| front | Work | /work | Portfolio, projects |
| left | Vision | /about | Company vision, team |
| back | News | /news | Newsletter, weekly videos |
| right | Showcase | (TBD) | Specific showcase, spare slot |
| bottom | Hamster | -- | Easter egg |

---

## Design Philosophy & Influences

### User-Centric Design

Visual design serves the user's cognitive processing. Accents and color guide the eye to interactive affordances. Hierarchy is established through weight, size, and color -- not decoration. The design should be immediately parseable: what can I click, what is content, where am I.

### LCARS (Star Trek: The Next Generation)

The ship computer interface from TNG. Rounded-corner blocks, bold solid color fields, clear typographic hierarchy, panel-edge aesthetic. Directly informs the Frame/Accent system: open frames suggest "panel edge" or "entry point", curved corners read as badge/stamp cuts. Information is presented in clear, functional blocks.

### Acid Design

Bold asymmetric layouts, high-contrast color blocking, strong grid presence, experimental typography. Large type, intentional whitespace (or blackspace). Informs the cube face experiments: elements are placed on the grid with deliberate asymmetry, not centered defaults. Favours **fill** icon variants for their bold silhouettes.

### Post-Neobrutalism

Thick borders (4px), sharp geometry softened by selective curves. Honest, structural design that doesn't hide its construction. Informs Frame border weight, corner treatment, and the overall "blocks on a grid" composition approach.

---
> Source: [urban-tech-creative/utc-website](https://github.com/urban-tech-creative/utc-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-27 -->
