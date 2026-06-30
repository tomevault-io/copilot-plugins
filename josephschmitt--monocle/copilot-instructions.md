## monocle

> Single-page site in `website/index.html` — all CSS, HTML, and JS in one file. No build step; served statically.

# Website Agent Instructions

## Structure

Single-page site in `website/index.html` — all CSS, HTML, and JS in one file. No build step; served statically.

## Design System

- **Theme**: Catppuccin Mocha. All colors via CSS custom properties (`--blue`, `--peach`, `--surface0`, etc.). Never use raw hex values.
- **Fonts**: Instrument Serif (headings, decorative `<em>`), Plus Jakarta Sans (body), JetBrains Mono (code, monospace UI).
- **Decorative emphasis**: Use `<em>` with Instrument Serif italic + `--blue` for accent words in headings and UI text.
- **Sections**: Separated by `.divider` (gradient 1px line). Each section uses `.section-pad` for vertical rhythm and `.container` for max-width.
- **Animations**: Scroll-reveal via `data-reveal` attribute + IntersectionObserver. Stagger delays with `.dr1`–`.dr4` classes.

## Feature Demos

The feature section has two states: a **grid** of feature cards and an expanded **demo view** with a sidebar + GIF player.

- Feature cards in `.feat-grid` have `data-demo` (GIF path), `data-demo-desc` (player description), and `data-color` (accent color) attributes.
- Cards without `data-demo` show a "Demo Soon" badge via CSS `::after` pseudo-element.
- The JS builds sidebar items from the grid cards automatically — no need to duplicate content.
- GIF dimensions are stored in a `gifSizes` JS object for `width`/`height` attributes on `<img>` tags to prevent layout collapse during load.
- The player body uses the padding aspect-ratio trick (`padding-bottom: 58.78%`) to reserve space before the GIF loads.

### Adding a new demo

1. Add the GIF to `website/assets/`.
2. Add `data-demo="assets/your-demo.gif"` and `data-demo-desc="..."` to the feature card's `<div>`.
3. Add the GIF dimensions to the `gifSizes` object in the JS.
4. The sidebar, player, and "Demo Soon" badge update automatically.

## Demo Descriptions (`data-demo-desc`)

When writing descriptions for demo GIFs in the player view, follow this tone and style:

### Tone
- **Present tense, imperative mood** — "Browse diffs", "Flag an issue", "Submit a review"
- **Reinforce the feature**, don't narrate the GIF frame-by-frame. This isn't closed captioning.
- Describe what the user **can do**, grounded in what the demo actually shows.
- Keep it to 2–3 sentences. Punchy, not breathless.

### What to reference
- Read `DEMO-SCRIPTS.md` at the repo root to understand what's actually happening in each recording.
- Pull out the key capabilities demonstrated, not the step-by-step actions.
- Mention specific UI features by name when they're shown (e.g., "version picker", "visual select", "focus mode").

### Examples (current descriptions)

**Diff review:**
> Browse multi-file diffs with full syntax highlighting and intra-line char diffs that show exactly what changed. Toggle between unified, side-by-side, and raw file views to read the code however you prefer.

**Structured feedback:**
> Select a range of lines to leave praise, flag a bug as an issue, or add a file-level note. Toggle comments between resolved and unresolved, then submit a structured review the agent picks up automatically.

**Plan review:**
> Review the agent's plan as rendered markdown before any code is written. Leave inline comments to request changes, then see the updated plan arrive as a diff between versions. Use the version picker to compare any revision against the latest.

**Instant review loop:**
> Claude writes code while Monocle shows the diffs live. Flag an issue, submit a review, and the agent receives your feedback instantly via push notification. It fixes the code and the diff updates — a tight loop without leaving the terminal.

### Anti-patterns
- Don't use "-ing" gerunds as the main verb ("Browsing", "Scrolling", "Watching") — that reads as narration
- Don't describe what's "on screen" — describe what the user does and gets
- Don't reuse the generic feature card `<p>` text — the demo desc should be specific to what the GIF shows

## Sneak Peek Section

Expandable teaser near the bottom for upcoming features (currently the desktop app). Uses CSS `grid-template-rows: 0fr → 1fr` for the expand animation. The carousel is scroll-snap based with dot indicators tracked via IntersectionObserver.

## Responsive

Breakpoint at `960px` switches to single-column layout. The feature sidebar becomes a horizontal scrollable row. The player body drops its fixed aspect ratio. A second breakpoint at `480px` handles small phones.

---
> Source: [josephschmitt/monocle](https://github.com/josephschmitt/monocle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-29 -->
