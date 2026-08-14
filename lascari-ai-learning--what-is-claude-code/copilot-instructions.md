## what-is-claude-code

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Slidev presentation about Claude Code. Slidev is a Vue.js-based presentation framework that uses markdown files for slides.

## Common Commands

```bash
# Install dependencies (uses pnpm)
pnpm install

# Start development server (http://localhost:3030)
pnpm dev

# Build for production
pnpm build

# Export presentation to PDF/PNG/PNGs
pnpm export

# Auto-generate index.md from slides directory
pnpm build:slides
```

## Architecture

### Slide System
- Individual slides are stored as markdown files in `slides/` directory
- Slides are numbered (e.g., `01-about-me.md`, `02-introduction.md`) for automatic ordering
- `scripts/build.ts` generates `index.md` by concatenating all slides in numerical order
- Each slide can have frontmatter for theme and layout configuration

### Key Directories
- `slides/` - Individual slide markdown files
- `layouts/` - Custom Vue layouts for different slide types (speaker, cover, etc.)
- `ai_docs/` - AI-related documentation including speaker profile and brand guidelines
- `public/` - Static assets accessible in slides

### Theming
- Uses custom fonts: Styrene A (headings) and Styrene B (body text)
- Custom color palette defined in styles
- Light color scheme by default
- Tailwind/Windi CSS for styling

### Custom Layouts
- `speaker.vue` - Speaker introduction slide with image and name slots
- `cover.vue` - Cover slide layout
- Custom slots: `::speaker-name::`, `::image::`

## Development Workflow

1. Add/edit slides in `slides/` directory following the numbering convention
2. Run `pnpm build:slides` to regenerate index.md
3. Use `pnpm dev` to preview changes
4. Slides support standard markdown, Vue components, and custom layouts

## Deployment
- Configured for both Netlify and Vercel
- Build command: `pnpm build`
- Output directory: `dist`

---
> Source: [Lascari-AI-Learning/what-is-claude-code](https://github.com/Lascari-AI-Learning/what-is-claude-code) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
