## davalerio-com

> Personal portfolio for David Valerio as an essayist and writer. Showcases published work — primarily Discern Earth (newsletter on faith, work, technology, and nature) — plus academic publications and external writing.

# AGENTS.md — davidavalerio.com

## What This Is

Personal portfolio for David Valerio as an essayist and writer. Showcases published work — primarily Discern Earth (newsletter on faith, work, technology, and nature) — plus academic publications and external writing.

## Tech Stack

- Pure static HTML + CSS (no JavaScript, no build system)
- Google Fonts: Cormorant Garamond (headings), EB Garamond (body)
- GitHub Pages deployment

## Project Structure

```
davidavalerio.com/
├── index.html        # Complete site (~3KB)
├── styles.css        # Separate stylesheet
├── CNAME             # GitHub Pages custom domain
├── favicon.svg       # SVG icon
├── headshot.jpg      # Profile photo
└── CLAUDE.md         # Brief project notes
```

## Page Sections

1. **Header** — Centered name "David Valerio"
2. **Headshot** — Circular profile image
3. **Bio** — Introductory paragraph, links to Discern Earth
4. **Writing** — Two subsections: Discern Earth essays + External Publications
5. **Contact** — Email link (david.a.valerio@gmail.com)
6. **Footer** — Copyright with dynamic year

## Design Tokens

- Background: `#f8f5f0` (cream)
- Text: `#1a1a1a` (near-black)
- Accent: `#722f37` (rust/maroon — headings and links)
- Gold: `#b8860b` (decorative accents, subtle borders)
- Max-width: 640px (narrow reading column)
- Base font size: 18px
- Mobile breakpoint: 600px

## Aesthetic

Intentionally quiet, text-focused, and thoughtful. Serif typography throughout. This is a writer's site, not a business site — the design reflects that. The restraint is the point.

## What Codex Should Know

- This is a very small site. Two files do all the work: `index.html` and `styles.css`.
- Unlike valeriosafety.com, there's no `content.md` source-of-truth pattern — content is edited directly in `index.html`.
- No JavaScript, no forms, no external services beyond Google Fonts.
- Deploy by pushing to main. No build step.
- The aesthetic is deliberately minimal. Don't add visual complexity without being asked.

---
> Source: [davidavalerio/davalerio.com](https://github.com/davidavalerio/davalerio.com) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
