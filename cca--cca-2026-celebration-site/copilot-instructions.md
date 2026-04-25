## cca-2026-celebration-site

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```sh
bun install          # install dependencies
bun run dev          # dev server at localhost:4321
bun run build        # production build to ./dist/
bun run preview      # preview production build
bunx astro check     # type-check .astro files
```

There are no lint or test commands — the project relies on `astro check` for type safety.

### Build arguments

`PUBLIC_ENV` (`production` | `staging`) must be set at Docker build time — it controls the generated `robots.txt`. Cloud Build triggers inject this automatically via the `_ENV` substitution variable. For local builds: `PUBLIC_ENV=production bun run build`.

## Architecture

This is an Astro 5 static site. All content lives in `src/content/` as JSON files, typed via Zod schemas in `src/content.config.ts`. There is no database, no server rendering, and no external CMS.

### Phase System

The most important architectural concept. A single variable in `src/config/phases.ts` controls what content is visible across the entire site:

```ts
// Four phases in order: 'save-the-date' | 'pre-event' | 'during-event' | 'post-event'
export const CURRENT_PHASE: Phase = 'pre-event';
```

Two utility functions gate visibility:
- `isPhaseAtLeast(current, minimum)` — true if current phase is at or after minimum
- `isPhaseBefore(current, threshold)` — true if current phase is earlier than threshold

In templates, use either direct utility calls or the `PhaseGate` component:
```astro
<PhaseGate visibleIn={['post-event']}>...</PhaseGate>
<PhaseGate hiddenIn={['save-the-date']}>...</PhaseGate>
```

The dev toolbar (Astro dev UI) lets you toggle phases locally without editing `phases.ts`. The override flows through `src/integrations/phase-toolbar/` and is read by `src/lib/utils/phase.ts → getCurrentPhase()`.

### Content Collections

Seven collections defined in `src/content.config.ts`, all loaded from `src/content/{collection}/*.json`:

- **students** — profiles with program refs, ceremony refs, links, photos
- **works** — media entries linked to students and events
- **events** — ceremonies, showcases, thesis shows with schedule/program data
- **programs** — academic programs with division, degreeTypes
- **people** — faculty/speakers/honorees linked to ceremonies
- **commencement-info** — logistics, statistics, downloadable assets
- **video-interviews** — linked to students, hosted on YouTube/Vimeo

All cross-references use Astro's `reference()` helper. Utility functions in `src/lib/utils/collections.ts` handle filtering and grouping (e.g., `getStudentsByEvent()`, `getCandidatesGroupedByDivision()`).

### Theming

No CSS framework. Design tokens are CSS custom properties defined in `src/styles/themes.css`. The active theme is set via a `data-theme` attribute on the `<html>` element (passed as `themeKey` prop to `BaseLayout`).

Theme variants: `commencement`, `commencement-undergrad`, `commencement-grad`, `showcase`, `thesis-undergrad`, `thesis-grad`, `celebration`.

Each theme overrides the full token set: `--color-bg`, `--color-text`, `--color-accent`, `--color-surface`, `--color-border`, plus RGB channels, shimmer colors, gradients, and overlay opacities.

Typography uses fluid sizing via `clamp()` on `--text-xs` through `--text-5xl`. Spacing uses `--space-1` (0.25rem) through `--space-24` (6rem).

### Scroll Animations

Add `.reveal`, `.reveal-scale`, `.reveal-left`, `.reveal-right`, or `.reveal-clip` to any element. `src/scripts/scroll-reveal.ts` runs an `IntersectionObserver` that adds `.is-visible` on viewport entry. For staggered children, add `.stagger-1` through `.stagger-8`.

### Layout Hierarchy

- `BaseLayout.astro` — root wrapper (head, GTM, global scripts, Header, Footer)
- `EventLayout.astro` — extends Base, adds `ParticleField` and theme support
- `StudentLayout.astro` — student profile with photo header, links, news section
- `WorkLayout.astro` — work detail with media gallery
- `ThesisEventLayout.astro` — thesis exhibition variant of EventLayout

### Key Utility Files

- `src/lib/utils/phase.ts` — `getCurrentPhase()`, `isContentVisible()`
- `src/lib/utils/collections.ts` — all content filtering and grouping helpers
- `src/lib/utils/format.ts` — `fullName()`, `formatDate()`, `formatTime()`, `formatDateRange()`
- `src/lib/utils/frame-data.ts` — frame/border styling data
- `src/lib/utils/calendar.ts` — calendar link utilities

### Event Hero Images

Two display modes are available for event hero images, controlled by which field is set in the event JSON:

**Poster/plain image** — use the `image` field. Renders a clean, unmasked image (no frame). Supports `aspectRatio` and `heroOnly: true` to suppress the image from appearing elsewhere.

```json
"image": {
  "src": "/images/cca-photography/my-photo.jpg",
  "alt": "Description",
  "aspectRatio": "4/5",
  "heroOnly": true
}
```

**Framed image** — use the `heroImages` array. Each entry gets an SVG clip-mask applied from `public/images/scanned-graphics/frames/frame-*.svg` (frames 01–23 available). Pass `heroImageSize="large"` to `EventHero` to scale up to 680px wide.

```json
"heroImages": [
  {
    "src": "/images/cca-photography/my-photo.jpg",
    "alt": "Description",
    "frameId": "frame-04"
  }
]
```

Both modes are handled by `src/components/events/EventHero.astro`. If neither field is set, the hero renders with no image.

### Event Photo Galleries

Some thesis event pages include a photo gallery section (e.g., Architecture Studio Conversations). These are implemented directly in the page `.astro` file — not a shared component — as a `<section class="gallery-section">` with a `.gallery-grid`. The first image gets `.gallery-item--featured` (spans 2 columns, 16:9 ratio); remaining images use 4:3. Photos are stored in `public/images/cca-photography/`.

### Photography Demo Page

`src/pages/demo/photography.astro` is an asset inventory page listing all images used across the site. To add new photos, add their filenames to the `ccaPhotography` array at the top of the file. Each image must be present in `public/images/cca-photography/`.

### Static Assets

Remote images from `cca.edu` are allowed in `astro.config.mjs`. Sharp handles image optimization. Fonts are self-hosted WOFF2 files in `public/fonts/` declared in `src/styles/fonts.css`.

## Common Tasks

### Adding a new event

Three files are always required — complete all three before considering the task done:

1. **Data file** — `src/content/events/{slug}.json`  
   Model it on an existing event. The `slug` field must match the filename.

2. **Detail page** — `src/pages/thesis/{slug}.astro`  
   Copy the closest existing page as a template and update the content query and layout accordingly.

3. **Bento grid** — `src/components/landing/BentoEvents.astro`  
   Add a `{ slug, size, area }` entry to the `layout` array, then add the `area` name to **all three** `grid-template-areas` blocks (mobile, tablet, desktop). The grid layout is manually designed — always flag that visual confirmation is needed before committing.

### Updating event details from the CCA portal

When new details arrive (schedule, presenters, admission info), update two files:

**1. Data file** — `src/content/events/{slug}.json`

Update `description` and add any schema-supported fields. The `schedule` field is optional (`Array<{ time, label }>`):

```json
"description": "Updated description. Admission is free and open to the public.",
"schedule": [
  { "time": "11:00 AM", "label": "Welcome and Opening Remarks" },
  { "time": "11:10 AM", "label": "Panel 1" }
]
```

**2. Detail page** — `src/pages/thesis/{slug}.astro`

Each thesis event page is custom — there's no single template. The right sections depend on what content is available. Use the building blocks that match:

| Content type | How to render |
|---|---|
| Narrative text / about copy | `EventContextSection` with `paragraphs` prop |
| Program schedule (from `event.data.schedule`) | `CeremonySchedule` component, null-guarded |
| Presenting students / speakers (not in schema) | Hardcoded inline array, rendered as a styled list |
| Photo gallery | Inline `<section class="gallery-section">` with `.gallery-grid` (see VCS or Architecture pages for reference) |

**`CeremonySchedule`** (`src/components/events/CeremonySchedule.astro`) — accepts `Array<{ time: string; label: string }>`, renders a vertical timeline card. Calls `formatTime()` internally, so times pass through as-is from JSON. Always null-guard since the schema field is optional:

```astro
{event.data.schedule && (
  <section id="schedule" class="content-section reveal">
    <h2 class="section-heading">Program Schedule</h2>
    <CeremonySchedule schedule={event.data.schedule} />
  </section>
)}
```

Data not supported by the schema (presenters, speakers, etc.) lives as a hardcoded array in the page frontmatter — not in the JSON, not in a shared component:

```astro
const presenters = ['Name One', 'Name Two', 'Name Three'];
```

Look at existing pages like `vcs-spring-symposium.astro` (schedule + presenters + gallery) and `mfa-writing-25th-anniversary.astro` (schedule only) as reference implementations.

## Developer Documentation

Developer docs are served as a live site at `/docs/` and authored in `src/content/docs/` (markdown) and `src/pages/docs/` (structured reference pages).

Key docs:
- `/docs/getting-started/` — local setup and key concepts
- `/docs/phase-system/` — Phase system deep dive
- `/docs/content-collections/` — Collections, Zod schemas, and utility layer
- `/docs/component-guide/` — Component taxonomy and layout hierarchy
- `/docs/styling-and-animation/` — Fluid tokens, themes, scroll-reveal
- `/docs/schema-reference/` — Full field reference for all 7 collections
- `/docs/component-catalog/` — All components with props and usage
- `/docs/route-index/` — All site routes cataloged

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/cca) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
