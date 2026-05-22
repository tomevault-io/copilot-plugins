## hero-mobile-image-containment-aspect-ratio

> Hero section mobile image containment using aspect-ratio matching — the definitive technique for containing images of varying dimensions (portrait, landscape, ultra-wide) inside responsive containers without cropping or gaps


# Hero Section Mobile: Aspect-Ratio Image Containment (March 2026)

## Overview

The homepage hero section (`src/components/HeroSection.tsx`) has a 4-section split layout. Each section contains an image with **different dimensions and aspect ratios**. On mobile (`@media (max-width: 767px)`), we need every image fully visible (zero crop) AND filling its container (zero gaps). This rule documents the **aspect-ratio matching technique** that solved this across all three image sections.

All mobile overrides live in `src/app/globals.css` inside the `@media (max-width: 767px)` block.

## The Problem

Each hero section image has a different aspect ratio:

| Section | Image | Recommended Size | Aspect Ratio | Shape |
|---------|-------|-----------------|--------------|-------|
| 1 | Kerala portrait | 1000 x 1200 px | **5:6** | Tall portrait |
| 2 | Event slideshow | 2000 x 800 px | **5:2** | Wide landscape |
| 3 | Unite India banner | 1000 x 200 px | **5:1** | Ultra-wide strip |

Naive approaches fail:
- **`object-cover`** fills the container but **crops** the image (top/bottom or left/right)
- **`object-contain`** shows the full image but leaves **letterbox gaps** (purple/background bars)
- **Fixed `height` / `min-height`** either crops tall images or leaves empty space below short ones

## The Core Technique: Aspect-Ratio Matching

**The solution**: Set the CSS `aspect-ratio` on the container to match the image's native proportions. When container and image have the same ratio, `object-contain` fills the container exactly — zero crop AND zero gaps.

```
Container aspect-ratio  ===  Image aspect-ratio
         +
    object-contain
         =
  Zero crop + Zero gaps
```

### Why This Works

- `aspect-ratio: 5 / 2` on a `width: 100%` container → browser auto-calculates height proportionally
- Image with `object-contain` scales to fit → since ratios match, it fills 100% of the container
- `height: auto` lets the container shrink/grow with viewport width → fully responsive
- `background: #1a0a2e` (dark purple matching hero bg) hides any sub-pixel rounding gaps

## Section-by-Section Implementation

### Section 1 — Kerala Portrait (5:6, `object-contain` + `max-height` cap)

Section 1 is a **tall portrait** image. Using `aspect-ratio: 5/6` on mobile would make the container extremely tall (taller than the viewport). Instead, we use `object-contain` with a `max-height` viewport cap so the image is fully visible but doesn't dominate the screen.

**Why not `aspect-ratio` here**: A 5:6 portrait on a 390px-wide phone → 468px tall container, which is too much. The `max-height: 22vh` cap keeps it proportional.

**Why not `object-cover`**: Cover fills width but crops top/bottom. The Kerala image has text/content at the edges that gets cut off.

```css
@media (max-width: 767px) {
  /* Container: full-bleed, no decorations */
  .hero-left-panel {
    width: 100%;
    flex: 0 0 auto;
    min-height: 0;
    padding: 0;
    margin: 0;
  }

  .hero-left-image {
    width: 100%;
    max-width: 100%;
    height: auto;
    min-height: 0;
    aspect-ratio: auto;       /* NOT locked — height driven by max-height cap */
    padding: 0;
    border-radius: 0;
    box-shadow: none;
    border: none;
    background: #1a0a2e;      /* Blends side letterbox bars with hero bg */
  }

  /* Kill decorative pseudo-elements */
  .hero-left-image::before,
  .hero-left-image::after {
    display: none;
  }

  .hero-left-image img {
    border-radius: 0 !important;
    object-fit: contain !important;          /* Full image — no crop */
    object-position: center center !important;
    width: 100% !important;
    height: auto !important;
    max-height: 22vh !important;             /* Cap prevents portrait from being too tall */
    display: block !important;
  }
}
```

**Trade-off**: `object-contain` on a portrait image in a landscape container creates **side gaps**. The `background: #1a0a2e` makes these gaps blend with the hero's dark purple background, making them invisible.

**Tuning**: Increase `max-height` to show more of the image (e.g., `28vh`), decrease to show less (e.g., `18vh`).

---

### Section 2 — Event Slideshow (5:2, `aspect-ratio` match)

Section 2 is a **wide landscape** image. This is the canonical use of the aspect-ratio technique.

```css
@media (max-width: 767px) {
  /* Panel: strip all spacing, content-driven height */
  .hero-right-panel {
    width: 100%;
    flex: 0 0 auto;
    min-height: 0;          /* Was 54vh — caused massive gap below image */
    padding: 0;
    margin-top: 0.125rem;   /* Tiny gap between Section 1 and 2 */
    align-items: stretch;
    justify-content: stretch;
  }

  /* Wrapper: aspect-ratio locks to image proportions */
  .hero-slideshow-wrapper {
    padding: 0;
    border-radius: 0;
    box-shadow: none;
    border: none;
    background: #1a0a2e;
    aspect-ratio: 5 / 2;    /* Matches 2000x800 image → zero crop, zero gaps */
    width: 100%;
    height: auto;
  }

  /* Kill decorative top-edge highlight */
  .hero-slideshow-wrapper::before {
    display: none;
  }

  .hero-slideshow-wrapper img {
    border-radius: 0 !important;
    object-fit: contain !important;   /* Full image visible */
    width: 100% !important;
    height: 100% !important;          /* Fill the aspect-ratio container */
  }

  /* Ticket/Register overlay: pinned bottom-right ON the image */
  .hero-ticket-overlay {
    bottom: 0.75rem;
    right: 0.75rem;
    z-index: 30;
  }
}
```

**Key insight**: `min-height: 54vh` was the original culprit — it made the panel taller than the image, and `align-items: center` (from base styles) vertically centered the image, creating a huge purple gap above it. Removing `min-height` and using `aspect-ratio` instead makes the container exactly image-sized.

---

### Section 3 — Unite India Banner (5:1, `aspect-ratio` match + text overlay)

Section 3 is an **ultra-wide strip** image with "About" / "Our Mission" text overlaid. Uses the same `aspect-ratio` technique as Section 2, plus absolute-positioned text.

```css
@media (max-width: 767px) {
  /* Card: aspect-ratio locks to banner proportions */
  .hero-bottom-card-mission {
    min-height: 0;           /* Was 160px — didn't match image ratio */
    overflow: hidden;
    position: relative;
    border-radius: 0.75rem;
    aspect-ratio: 5 / 1;    /* Matches 1000x200 image → zero crop, zero gaps */
    width: 100%;
    height: auto;
    background: #1a0a2e;
  }

  /* Stronger gradient so white text reads over the image */
  .hero-bottom-card-mission .hero-mission-overlay {
    background: linear-gradient(
      to top,
      rgba(0,0,0,0.65) 0%,
      rgba(0,0,0,0.3) 50%,
      transparent 100%
    );
  }

  /* Text: absolutely positioned inside the card, overlaying the image */
  .hero-bottom-card-mission .hero-mission-content {
    margin-top: 0;
    position: absolute;
    bottom: 0.75rem;
    left: 0;
    right: 0;
    padding: 0.25rem 0.75rem;
    text-shadow: 0 1px 4px rgba(0,0,0,0.7);   /* Extra readability */
  }

  /* Smaller text to fit within the ultra-wide card */
  .hero-bottom-card-mission .hero-card-label {
    font-size: 0.625rem;
    margin-bottom: 0.125rem;
  }
  .hero-bottom-card-mission .hero-card-title {
    font-size: 0.8125rem;
  }

  /* Background image: contained inside the 5:1 card */
  .hero-bottom-card-mission .hero-mission-bg {
    position: absolute;
    inset: 0;
  }
  .hero-bottom-card-mission .hero-mission-bg img {
    object-fit: contain !important;
    object-position: center center !important;
    width: 100% !important;
    height: 100% !important;
  }
}
```

**Key insight**: The bg image uses `position: absolute; inset: 0` (fills the card), so the card's `aspect-ratio: 5/1` directly controls the image's display box. When both match 5:1, contain fills perfectly.

---

## Desktop (768px+) — Different Strategy

On desktop, the hero uses a side-by-side layout (Section 1 left, Section 2 right). The technique differs:

| Section | Desktop Strategy | Why |
|---------|-----------------|-----|
| 1 | `aspect-ratio: 5/6` on container + `object-cover` | Container ratio matches portrait; cover fills it exactly |
| 2 | `object-cover` + `border-radius: 0.75rem` | Landscape images fill the large panel; slight edge crop is acceptable at desktop size |
| 3 | Same as mobile (unchanged) | Bottom row cards are the same width on desktop |

```css
@media (min-width: 768px) {
  .hero-left-panel { width: 35%; flex: none; padding: 0; }
  .hero-left-image { width: auto; height: 100%; aspect-ratio: 5 / 6; max-width: 100%; }
  .hero-left-image img { object-fit: cover !important; }

  .hero-right-panel { padding: 0; }
  .hero-slideshow-wrapper { padding: 0; }
  .hero-slideshow-wrapper img { object-fit: cover !important; border-radius: 0.75rem; }
}
```

---

## Decision Tree: Which Technique to Use

```
Is the image a PORTRAIT (taller than wide)?
  YES → object-contain + max-height cap + dark background
        (aspect-ratio would make container too tall on mobile)

Is the image LANDSCAPE or ULTRA-WIDE?
  YES → aspect-ratio matching + object-contain
        Container ratio = Image ratio → zero crop, zero gaps

Does the image have TEXT OVERLAY?
  YES → Add: position: relative on card
        Add: position: absolute; bottom: Xrem on text
        Add: gradient overlay for readability
        Add: text-shadow for extra contrast
```

## Common Pitfalls & Lessons Learned

### 1. `min-height` creates phantom gaps
**Problem**: `min-height: 54vh` on a panel + `align-items: center` = image centered with huge gap above.
**Fix**: Remove `min-height`, use `aspect-ratio` or content-driven height instead.

### 2. `object-cover` crops differently at every viewport width
**Problem**: A 5:2 landscape image in a 36vh-tall container is wider than needed on narrow phones → left/right edges get cropped.
**Fix**: `aspect-ratio: 5/2` container + `object-contain` = container and image are always the same shape.

### 3. `object-contain` on portrait images creates side gaps
**Problem**: A 5:6 portrait on a landscape mobile screen (wider than tall) → bars on left and right.
**Fix**: `background: #1a0a2e` blends the bars into the hero's dark purple background. Visually invisible.

### 4. Fixed `height` values don't scale across devices
**Problem**: `height: 160px` looks fine on iPhone SE but wrong on iPad mini.
**Fix**: `aspect-ratio` + `width: 100%` + `height: auto` scales proportionally at every width.

### 5. Flexbox alignment fights content-driven sizing
**Problem**: Base styles with `align-items: center; justify-content: center` on panels redistribute empty space around images.
**Fix**: Override to `align-items: stretch; justify-content: stretch` on mobile, or remove `min-height` so there's no empty space to distribute.

### 6. `position: absolute` bg images need explicit dimensions
**Problem**: `.hero-mission-bg` uses `position: absolute; inset: 0` — the image fills whatever the card is. If the card has wrong dimensions, the image overflows.
**Fix**: Card's `aspect-ratio` controls the box → bg image inherits correct dimensions via `inset: 0`.

---

## File References

- **CSS overrides**: [`src/app/globals.css`](mdc:src/app/globals.css) — `@media (max-width: 767px)` hero block (lines ~974–1144)
- **Desktop overrides**: [`src/app/globals.css`](mdc:src/app/globals.css) — `@media (min-width: 768px)` hero blocks
- **Component**: [`src/components/HeroSection.tsx`](mdc:src/components/HeroSection.tsx) — 4-section split layout
- **Layout diagram**: `public/images/hero_section/hero_section_split_excalidraw.png`
- **General containment rule**: [image_containment_prevention_hero_images_cards.mdc](mdc:.cursor/rules/image_containment_prevention_hero_images_cards.mdc)

## Quick Reference Checklist

Before modifying hero image CSS on mobile:

- [ ] Identify the image's native aspect ratio (width:height)
- [ ] If landscape/ultra-wide: set `aspect-ratio` on container to match
- [ ] If portrait: use `object-contain` + `max-height` cap instead
- [ ] Always `background: #1a0a2e` on container (blends letterbox gaps)
- [ ] Always `object-fit: contain !important` on `img`
- [ ] Always `width: 100% !important` on `img`
- [ ] Remove `min-height` from containers (causes alignment gaps)
- [ ] Remove decorative styles (`border-radius`, `box-shadow`, `border`) for full-bleed
- [ ] If text overlay: `position: absolute; bottom: Xrem` + gradient + `text-shadow`
- [ ] Test on multiple viewport widths (320px, 390px, 428px, 768px)

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-22 -->
