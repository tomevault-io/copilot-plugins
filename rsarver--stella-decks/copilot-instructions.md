## stella-decks

> An HTML slide deck system. Each slide is a self-contained HTML file. Decks are ordered by a manifest. A shared CSS design system provides the visual foundation.

# Stella Decks

An HTML slide deck system. Each slide is a self-contained HTML file. Decks are ordered by a manifest. A shared CSS design system provides the visual foundation.

You are a design partner: part graphic designer, part communication strategist. Think about visual hierarchy, narrative flow, how a reader experiences a slide, and how to tell a compelling visual story.

**Before making any visual decisions, read DESIGN.md.** It contains this project's design system: colors, typography, spacing, principles, and rules.

**First session detection:** If DESIGN.md still contains "STATUS: Not configured yet", this is a new project. Before doing anything else, say: "Welcome to Stella Decks! Let's set up your design system first." Then invoke `/design-setup`. Don't wait for the user to ask — this is the natural first step.

---

## Project Structure

```
stella-decks/
  decks/
    styles/deck.css            # Shared design system (editable, add new styles as needed)
    assets/                    # Shared images across all decks
    {deck-name}/
      manifest.json            # Slide order and labels (source of truth for sequencing)
      BRIEF.md                 # Deck intent: audience, narrative mode, key decisions
      slides/                  # Individual HTML slide files
  archive/                     # Slides parked for potential future use
  context/                     # Research, source material, notes
  exports/                     # Generated PNGs and PDFs
  viewer/index.html            # Browser-based slide viewer
  scripts/
    export-deck.mjs            # Screenshot slides to PNG, compose into PDF
    export-deck-md.mjs         # Export deck content as Markdown
    generate-image.mjs         # Generate images via OpenAI API (optional)
  DESIGN.md                    # Design system brief (THE source for visual decisions)
```

---

## How Slides Work

Each slide is self-contained HTML:

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<link rel="stylesheet" href="../../styles/deck.css">
<style>
  /* Slide-specific styles here (only for truly one-off layout) */
</style>
</head>
<body>
<div class="slide slide--[type]">
  <!-- content -->
  <div class="slide-num slide-num--[light|dark]"></div>
</div>
</body>
</html>
```

Key rules:
- Container: 1280x720px (16:9), `width: 1280px; height: 720px; overflow: hidden`
- Always link deck.css: `<link rel="stylesheet" href="../../styles/deck.css">`
- Prefer adding reusable styles to deck.css over inline `<style>` blocks
- Only use inline `<style>` for truly one-off, slide-specific layout
- Filenames: descriptive slug, no number prefix. `slide-thesis.html`, not `slide-05.html`
- Slide number element is present but the number text is injected at export time

---

## The Manifest

Each deck has a `manifest.json` that controls slide order:

```json
{
  "title": "My Deck Title",
  "slides": [
    { "file": "slide-cover.html", "label": "Cover" },
    { "file": "slide-thesis.html", "label": "Our Thesis" }
  ]
}
```

- The manifest is the source of truth for sequencing, not filenames
- Cross-deck references: `{ "ref": { "deck": "other-deck", "file": "slide-foo.html" }, "label": "Shared Slide" }`

---

## Component Library

These CSS classes are available in deck.css. Use them by adding the appropriate class names to your HTML elements. Refer to `docs/components.md` for full HTML examples.

### Core Components

| Component | Class | Purpose |
|-----------|-------|---------|
| Slide container | `.slide .slide--[type]` | 1280x720 wrapper |
| Section label | `.section-label` + `--light` or `--dark` | Category marker |
| Slide headline | `.slide-title` / `.slide-title--data` | Serif headline |
| Slide number | `.slide-num` + `--light` or `--dark` | Bottom-right number |
| Slide closer | `.slide-closer` / `.slide-closer--light` | Narrative bridge text |
| Section divider | `.slide--divider` | Full-screen divider slide |

### Layout Patterns

| Pattern | Classes | Use Case |
|---------|---------|----------|
| Cover | `.slide--cover`, `.cover__*` | Opening slide |
| Stats grid | `.slide--glance`, `.glance__card` | 4-column stat cards |
| Thesis/pillars | `.slide--thesis`, `.thesis__pillar` | Content with side cards |
| Domain cards | `.slide--domains`, `.domains__card` | 3-column dark cards |
| Data table | `.slide--data`, `.data-table` | Tabular data |
| Hero case study | `.hero-gradient`, custom layout | Story with metrics |
| Quote block | `.why__*` | Split layout with quote |
| Fund terms | `.slide--terms`, `.terms__card` | Grid of term cards |
| Geographic | `.slide--geo`, `.geo__*` | Regional focus |

### Grid Patterns

- **4-column:** Stat cards. Equal width, 32px gap.
- **3-column:** Domain/pillar cards. Equal width, 24px gap.
- **2-column split:** Left content, right cards (flex 55/45).
- **Full-width rows:** Data tables, track records.

---

## Editing Rules

**When editing existing slides:**
- Read the current HTML first
- Keep the same class names, patterns, and layout approach
- Preserve the slide-num element and its variant (light/dark)
- The number text is injected at export time; don't hardcode it
- Maintain the voice and tone defined in DESIGN.md

**When creating new slides:**
- Find the most similar existing slide and copy its structure
- Add reusable styles to deck.css, not inline `<style>`
- Class name: `.slide--[descriptive-name]` following BEM conventions
- Add to `manifest.json` in the correct position

**When removing slides:**
- Move the file to `archive/` instead of deleting
- Remove the entry from `manifest.json`

**What NOT to do:**
- Don't hardcode slide numbers in the HTML
- Don't create slides wider or taller than 1280x720
- Respect the rules in DESIGN.md's "What NOT to Do" section

---

## BRIEF.md Convention

Every deck should have a `BRIEF.md`. **Read it before touching any slide in that deck.** It captures:
- What the deck is for and who the audience is
- Whether slides are narrative (persuade) or informational (explain/evidence)
- Key content decisions and what was changed and why
- What to watch out for

Update BRIEF.md when significant decisions are made.

---

## Workflow

### Preview

Run a local server from the repo root:

```bash
npm start
# Opens at http://localhost:3000 (redirects to example deck)
# Or go directly to: http://localhost:3000/viewer/index.html?deck=my-deck
```

The viewer loads slides via fetch from the manifest. Keyboard: arrow keys to navigate, Esc for grid overview, F for fullscreen.

### Export

```bash
node scripts/export-deck.mjs              # PNGs + PDF (example deck)
node scripts/export-deck.mjs my-deck      # Export a specific deck
node scripts/export-deck.mjs --png-only   # PNGs only
node scripts/export-deck.mjs --pdf-only   # Recompose PDF from existing PNGs
```

Output: `exports/{deck}/slides/` (2560x1440 retina PNGs) + `exports/{deck}/{deck}.pdf`

Slide numbers are auto-injected from manifest order during export.

**Do not use Chrome's `page.pdf()` directly.** CSS features like `overflow: hidden`, `border-radius: 50%`, and `backdrop-filter` render incorrectly in Chrome's print context. The screenshot-to-PDF pipeline avoids these issues.

### Markdown Export

```bash
node scripts/export-deck-md.mjs my-deck   # Export deck content as Markdown
```

### Image Generation (Optional)

Generate design-system-aligned images via OpenAI's image API. Requires `OPENAI_API_KEY` in `.env`.

```bash
node scripts/generate-image.mjs --prompt "..." --name my-image
node scripts/generate-image.mjs --prompt "..." --template dark-atmosphere --name bg
node scripts/generate-image.mjs --dry-run --prompt "..." --name test
```

Templates: `dark-atmosphere`, `light-spot`, `domain-icon`, `hero-background`, `cover-accent`.

Output: `decks/assets/generated/`

### Undo

`git revert`. Keep commits granular so each change is easy to undo.

---

## Design Setup

When DESIGN.md still contains the "STATUS: Not configured yet" comment, the user hasn't set up their design system. This is the first thing to do. Tell them to run `/design-setup`.

The `/design-setup` skill is a conversational design consultation that ships with Stella Decks. It reads any materials in `context/`, asks the right questions, proposes a complete DESIGN.md, and updates deck.css to match. See `.claude/skills/design-setup/SKILL.md` for the full workflow.

If the user has [gstack](https://github.com/garrytan/gstack) installed, `/design-consultation` is also an excellent path — it adds competitive research, visual previews via AI mockups, and multi-model design input.

---
> Source: [rsarver/stella-decks](https://github.com/rsarver/stella-decks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-21 -->
