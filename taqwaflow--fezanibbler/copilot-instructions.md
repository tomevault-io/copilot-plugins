## fezanibbler

> Multi-page construction company website. Full site with multiple pages/routes.

# Ustadh Ahmad Truck — Construction Website (Gemini Instructions)

## Project

Multi-page construction company website. Full site with multiple pages/routes.
Design reference and site skeleton will be provided separately.

## Core Rules

- Do not make changes until you have 95% confidence in what to build. Ask follow-up questions until you reach that confidence.
- Avoid conflicting edits. Work on your assigned section(s) only.
- Save before the user switches to another agent.

## Multi-Agent Setup

Three agents share this workspace: **Claude Code**, **Codex**, **Gemini**.
All see the same files. The user coordinates — agents do not interact directly.

### Workflow Modes

**Assigned Sections:** Each agent owns specific `<!-- SECTION: name -->` blocks in `index.html`. Only edit your assigned block.

**Bake-Off Mode:** When the user says "bake-off" or "all three try this":
1. Create your output in `drafts/gemini/` (e.g., `drafts/gemini/hero.html`)
2. Do NOT edit `index.html` directly
3. The user will compare all three drafts and pick the winner to merge

## Tech Stack

- Multi-page site (index.html + additional pages)
- Tailwind CSS via CDN (`<script src="https://cdn.tailwindcss.com"></script>`)
- Placeholder images from `https://placehold.co/` when source images aren't provided
- Mobile-first responsive design
- Shared components (nav, footer) consistent across pages

## Design Philosophy

- Elite, pixel-perfect execution — match the reference exactly
- All complex backgrounds/patterns/visuals: generated procedurally through code/math/CSS
- No external image assets unless explicitly provided
- Code-drawn decorative elements where the design calls for it
- Procedural patterns > stock assets, always
- 60fps performance target for any animations/generative visuals
- Color palette from the design reference (construction theme)

## Screenshot Comparison Workflow

After building or editing any section:

1. **Generate** the section in `index.html` or draft file
2. **Screenshot** the rendered page (any browser automation available)
3. **Compare** against reference image — check spacing, fonts, colors, alignment, radii, shadows, responsive behavior
4. **Fix** every mismatch
5. **Re-screenshot** and compare again
6. **Repeat** until within ~2-3px of reference everywhere

Minimum 2 comparison rounds. Be specific about mismatches (e.g., "heading is 32px but reference shows ~24px").

### Comparison Rules
- Do not add features/sections not in the reference
- Match the reference exactly — do not "improve" the design
- If CSS classes or style tokens are provided, use them verbatim

## Memory & Rules

Gemini-specific memory and rules live in `.gemini/memory/` and `.gemini/rules/`.
Same structure as Claude: MEMORY.md index + topic files, path-scoped rules.

## Project Structure

```
.
├── CLAUDE.md              # Claude Code instructions
├── AGENTS.md              # Codex instructions
├── GEMINI.md              # Gemini instructions (this file)
├── index.html             # Homepage
├── *.html                 # Additional pages
├── drafts/                # Bake-off outputs
│   ├── claude/
│   ├── codex/
│   └── gemini/
```

## Content Guidelines

- Construction industry — professional, trustworthy, modern
- Colour theme will differ from reference template (to be provided)
- Site skeleton and design template to be provided separately

---

**Do not make any changes until you have 95% confidence in what you need to build. Ask follow-up questions until you reach that confidence.**

---
> Source: [TaqwaFlow/fezanibbler](https://github.com/TaqwaFlow/fezanibbler) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
