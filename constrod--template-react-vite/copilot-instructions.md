## template-react-vite

> # Design System & UI Guidelines


# bossROD TV Design System & UI Guidelines

## Core Aesthetic: "Minimalist Tech & Code"

The bossROD TV brand aesthetic sits at the intersection of **Programming, Technology, and Content Creation**. It is **"Clean Code meets Visual Clarity."**

The design must instantly communicate four key pillars:

1.  **Technical Excellence:** Clean, precise layouts that reflect code structure and logical thinking.
2.  **Minimalist Focus:** Black and white palette that removes distraction and emphasizes content.
3.  **Developer Identity:** Terminal aesthetics, monospace typography, and code-inspired UI patterns.
4.  **Content Creation:** Video frames, streaming UI elements, and creator-focused components.

**Key Visual Metaphors:**

- **The Terminal:** Dark backgrounds, monospace fonts, command-line aesthetics.
- **The IDE:** Syntax highlighting accents, tab-based navigation, panel layouts.
- **The Triangle:** Logo-inspired angular shapes, geometric precision, sharp edges.
- **The Stream:** Video frames, live indicators, recording UI elements.

---

## 1. Typography & Text Effects

### Font Stack

- **Primary:** `Inter` (via Tailwind default sans) for UI and body text.
- **Code/Accent:** `JetBrains Mono` or system monospace for technical elements and emphasis.

### Text Colors (Theme-Aware)

**CRITICAL:** ALWAYS use semantic theme colors (`text-foreground`, `text-muted-foreground`, `text-primary`, `bg-background`, `bg-card`). **NEVER** use hardcoded colors like `text-gray-900`, `text-gray-600`, or `bg-white` (unless intentionally fixed, e.g., on a dark overlay).

| Role           | Class                   | Description                              |
| -------------- | ----------------------- | ---------------------------------------- |
| **Headings**   | `text-foreground`       | Main titles, high contrast.              |
| **Body**       | `text-muted-foreground` | Descriptions and secondary text.         |
| **Code/Tech**  | `font-mono`             | Technical terms, code snippets.          |
| **Emphasis**   | `text-primary`          | Key brand moments or active states.      |

### The "Code Comment" Effect

Use this for section subtitles to emphasize the **developer/tech** identity. It simulates code comments.

```tsx
<p className="font-mono text-sm text-muted-foreground">
  <span className="text-muted-foreground/60">// </span>
  Building the future, one commit at a time
</p>
```

### The "Terminal Prompt" Effect

Use for CTAs or interactive elements to create a command-line feel.

```tsx
<span className="font-mono text-sm">
  <span className="text-primary">$</span> start learning
  <span className="animate-pulse">_</span>
</span>
```

---

## 2. Color System

### Primary Palette: Black & White

The core palette is strictly monochromatic for maximum clarity and focus.

| Token              | Light Mode          | Dark Mode           | Usage                    |
| ------------------ | ------------------- | ------------------- | ------------------------ |
| `--background`     | `#ffffff`           | `#0a0a0a`           | Page backgrounds         |
| `--foreground`     | `#0a0a0a`           | `#fafafa`           | Primary text             |
| `--card`           | `#ffffff`           | `#111111`           | Card backgrounds         |
| `--card-foreground`| `#0a0a0a`           | `#fafafa`           | Card text                |
| `--muted`          | `#f4f4f5`           | `#1a1a1a`           | Muted backgrounds        |
| `--muted-foreground`| `#71717a`          | `#a1a1aa`           | Secondary text           |
| `--border`         | `#e4e4e7`           | `#27272a`           | Borders and dividers     |
| `--primary`        | `#0a0a0a`           | `#fafafa`           | Primary actions          |
| `--primary-foreground`| `#fafafa`        | `#0a0a0a`           | Text on primary          |

### Accent Colors (Sparingly Used)

Only use accents for functional feedback, never for decoration.

| Color    | Usage                                    |
| -------- | ---------------------------------------- |
| `green`  | Success states, "live" indicators        |
| `red`    | Error states, recording indicators       |
| `yellow` | Warning states, pending actions          |
| `blue`   | Links, information (use minimally)       |

---

## 3. Component Patterns

### A. The "Code Block" Card

Used for: _Features, tutorials, technical content._ Represents the **Developer Identity** aspect.

- **Container:** Dark background with subtle border, monospace accents.
- **Header:** Simulates a terminal/IDE title bar with dots.

```tsx
<div className="overflow-hidden rounded-lg border border-border bg-card">
  {/* Terminal Header */}
  <div className="flex items-center gap-2 border-b border-border bg-muted px-4 py-2">
    <div className="h-3 w-3 rounded-full bg-red-500/80" />
    <div className="h-3 w-3 rounded-full bg-yellow-500/80" />
    <div className="h-3 w-3 rounded-full bg-green-500/80" />
    <span className="ml-2 font-mono text-xs text-muted-foreground">
      component.tsx
    </span>
  </div>
  {/* Content */}
  <div className="p-4">
    <pre className="font-mono text-sm text-foreground">
      {/* Code content */}
    </pre>
  </div>
</div>
```

### B. The "Panel" Card (Clean Container)

Used for: _General content, stats, info blocks._ Represents the **Minimalist Focus** aspect.

- **Style:** Clean borders, no shadows (or very subtle), sharp corners or minimal rounding.
- **Spacing:** Generous padding, clear hierarchy.

```tsx
<div className="rounded-lg border border-border bg-card p-6">
  <h3 className="text-lg font-semibold text-foreground">Panel Title</h3>
  <p className="mt-2 text-muted-foreground">
    Clean, focused content without distraction.
  </p>
</div>
```

### C. The "Stat Block" (Dashboard Widget)

Used for: _Metrics, numbers, key statistics._ Represents the **Technical Excellence** aspect.

```tsx
<div className="rounded-lg border border-border bg-card p-4">
  <p className="font-mono text-xs uppercase tracking-wider text-muted-foreground">
    Total Views
  </p>
  <div className="mt-1 flex items-baseline gap-2">
    <span className="font-mono text-3xl font-bold text-foreground">
      1.2M
    </span>
    <span className="font-mono text-xs text-green-500">+12%</span>
  </div>
</div>
```

### D. The "Video Frame" (Content Showcase)

Used for: _Video previews, thumbnails, media content._ Represents the **Content Creation** aspect.

```tsx
<div className="group relative overflow-hidden rounded-lg border border-border bg-black">
  {/* Live/Recording Indicator */}
  <div className="absolute left-3 top-3 z-10 flex items-center gap-1.5 rounded bg-red-600 px-2 py-0.5">
    <div className="h-2 w-2 animate-pulse rounded-full bg-white" />
    <span className="font-mono text-xs font-medium text-white">LIVE</span>
  </div>
  {/* Video Content */}
  <div className="aspect-video bg-muted">
    <img className="h-full w-full object-cover" />
  </div>
  {/* Progress Bar */}
  <div className="h-1 w-full bg-muted">
    <div className="h-full w-1/3 bg-primary" />
  </div>
</div>
```

### E. The "Triangle Accent" (Brand Element)

Used for: _Decorative accents, section dividers._ Represents the **Logo/Brand Identity**.

```tsx
{/* Decorative triangle inspired by logo */}
<div className="relative">
  <div className="absolute -left-4 top-0 h-0 w-0 border-l-[16px] border-r-[16px] border-t-[24px] border-l-transparent border-r-transparent border-t-primary opacity-10" />
</div>

{/* Or as a section divider */}
<div className="flex items-center gap-4">
  <div className="h-px flex-1 bg-border" />
  <div className="h-0 w-0 border-l-8 border-r-8 border-t-12 border-l-transparent border-r-transparent border-t-foreground" />
  <div className="h-px flex-1 bg-border" />
</div>
```

### F. The "Command Palette" (Interactive Element)

Used for: _Search, navigation, quick actions._ Represents the **Developer Tools** aspect.

```tsx
<div className="mx-auto w-full max-w-lg overflow-hidden rounded-lg border border-border bg-card shadow-2xl">
  <div className="flex items-center gap-2 border-b border-border px-4 py-3">
    <span className="font-mono text-muted-foreground">&gt;</span>
    <input
      className="flex-1 bg-transparent font-mono text-foreground outline-none placeholder:text-muted-foreground"
      placeholder="Type a command..."
    />
    <kbd className="rounded border border-border bg-muted px-2 py-0.5 font-mono text-xs text-muted-foreground">
      ⌘K
    </kbd>
  </div>
  {/* Results */}
</div>
```

---

## 4. Backgrounds & Textures

### The "Grid" Pattern (Subtle Structure)

Used for: _Hero sections, backgrounds._ Represents **Technical Precision**.

```tsx
<div className="absolute inset-0 -z-10 bg-[linear-gradient(to_right,#80808008_1px,transparent_1px),linear-gradient(to_bottom,#80808008_1px,transparent_1px)] bg-[size:32px_32px]" />
```

### The "Dot Matrix" (Minimal Texture)

Used for: _Section backgrounds._ Represents **Digital/Tech**.

```tsx
<div className="absolute inset-0 -z-10 bg-[radial-gradient(#71717a_0.5px,transparent_0.5px)] [background-size:24px_24px] opacity-30" />
```

### The "Gradient Fade" (Depth)

Used for: _Edge fading, focus areas._ Creates depth without color.

```tsx
<div className="pointer-events-none absolute inset-0 bg-gradient-to-t from-background via-transparent to-transparent" />
```

---

## 5. Buttons & Interactive Elements

### Primary Button (Solid)

Clean, high-contrast, no frills.

```tsx
className="inline-flex items-center justify-center rounded-md bg-primary px-4 py-2 font-medium text-primary-foreground transition-colors hover:bg-primary/90 focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring"
```

### Secondary Button (Outline)

Subtle, for secondary actions.

```tsx
className="inline-flex items-center justify-center rounded-md border border-border bg-transparent px-4 py-2 font-medium text-foreground transition-colors hover:bg-muted focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-ring"
```

### Ghost Button

Minimal, for tertiary actions.

```tsx
className="inline-flex items-center justify-center rounded-md px-4 py-2 font-medium text-muted-foreground transition-colors hover:bg-muted hover:text-foreground"
```

### Code-Style Button

For tech-focused CTAs.

```tsx
<button className="group inline-flex items-center gap-2 rounded-md border border-border bg-card px-4 py-2 font-mono text-sm transition-colors hover:bg-muted">
  <span className="text-primary">$</span>
  <span className="text-foreground">npm start</span>
  <span className="text-muted-foreground group-hover:text-foreground">↵</span>
</button>
```

---

## 6. Animation Guidelines

- **Transitions:** Keep it subtle. `transition-colors duration-200` for most interactions.
- **Hover States:** Prefer color/opacity changes over transforms.
- **Loading:** Use simple pulse or skeleton animations, avoid spinners when possible.
- **Cursor Blink:** `animate-pulse` for terminal cursor effects.

```tsx
// Blinking cursor
<span className="animate-pulse">_</span>

// Subtle fade in
<div className="animate-in fade-in duration-300">

// Skeleton loading
<div className="h-4 w-32 animate-pulse rounded bg-muted" />
```

---

## 7. Spacing & Layout

### Grid System

Use 8px base unit. Prefer `gap-4`, `gap-6`, `gap-8` for consistency.

### Card Spacing

- Internal padding: `p-4` (compact), `p-6` (standard), `p-8` (spacious)
- Card gaps: `gap-4` minimum

### Section Spacing

- Between sections: `py-16` to `py-24`
- Content max-width: `max-w-6xl mx-auto px-4`

---

## 8. Implementation Checklist for Agents

When generating new UI components:

1.  [ ] **Color Check:** Using only `text-foreground`, `bg-background`, `border-border`, etc.?
2.  [ ] **Contrast Check:** Is there enough contrast? Black/white should dominate.
3.  [ ] **Font Check:** Is `font-mono` used for technical elements?
4.  [ ] **Minimal Check:** Can anything be removed? Less is more.
5.  [ ] **Border Check:** Using borders over shadows for separation?
6.  [ ] **Spacing Check:** Is spacing consistent with 8px grid?
7.  [ ] **No Decorative Colors:** Accents only for functional feedback (success/error/warning).
8.  [ ] **Theme Support:** Does it work in both light and dark modes?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/constROD) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
