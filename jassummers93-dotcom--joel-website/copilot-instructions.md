## joel-website

> A professional single-page portfolio website for Joel Summers, Senior Audio Artist at Codemasters (EA).

# Joel Summers Portfolio

A professional single-page portfolio website for Joel Summers, Senior Audio Artist at Codemasters (EA).

## Tech Stack

- **Astro** - Static site generator
- **Tailwind CSS** - Utility-first styling
- **TypeScript** - Type safety
- **Cloudflare Pages** - Deployment target

## Commands

```bash
npm run dev      # Start dev server at http://localhost:4321
npm run build    # Build for production to dist/
npm run preview  # Preview production build
```

## Project Structure

```
src/
├── components/
│   ├── Header.astro       # Fixed navigation with mobile menu
│   ├── Hero.astro         # Full-screen hero with waveform animation
│   ├── About.astro        # Bio section with photo and skills
│   ├── Projects.astro     # AAA game projects grid
│   ├── ProjectCard.astro  # Video thumbnail card with modal
│   ├── Credits.astro      # Two-column work history timeline
│   ├── Testimonials.astro # Client testimonials (placeholder)
│   ├── Contact.astro      # Email and social links
│   └── Footer.astro       # Copyright and back-to-top
├── data/
│   ├── projects.json      # Game project metadata
│   └── credits.json       # Sound Designer & Recordist credits
├── layouts/
│   └── Layout.astro       # Base layout with SEO/Open Graph
├── pages/
│   └── index.astro        # Main page combining all sections
└── styles/
    └── global.css         # Design system and animations
public/
└── images/
    ├── game_thumbnails/   # Project thumbnails
    ├── hero/              # Background and logo
    ├── icons/             # IMDB, MobyGames icons
    └── profile/           # Joel's headshot
```

## Design System

### Colors
- Background: `#0a0a0f` (near black)
- Surface: `#141419` (elevated elements)
- Accent: `#e63946` (red for CTAs)
- Secondary: `#4361ee` (blue for links)
- Text: `#f8f9fa` (primary), `#9ca3af` (muted)

### Fonts
- Display: Space Grotesk
- Body: Inter

## Content Updates Needed

1. **Contact email** - Update in `src/components/Contact.astro` line 3
2. **Testimonials** - Replace placeholder content in `src/components/Testimonials.astro`

## Deployment

Configured for Cloudflare Pages via `wrangler.toml`. The build output directory is `dist/`.

To deploy:
1. Connect repository to Cloudflare Pages
2. Set build command: `npm run build`
3. Set output directory: `dist`

## Reference

- Original Wix site: https://joel43jas.wixsite.com/jsummersaudio
- Content source: `website_notes.txt`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jassummers93-dotcom) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
