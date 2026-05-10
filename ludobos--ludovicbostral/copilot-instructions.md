## ludovicbostral

> Personal site for Ludovic Bostral, founder of Streaming Radar.

# Claude Code Configuration

## Project Overview
Personal site for Ludovic Bostral, founder of Streaming Radar.
Hub showcasing intelligence reports (Lens), essays, MCP infrastructure, and inbound advisory.
Two languages: FR (default at /) and EN (at /en/).

## Tech Stack
- **Framework**: Astro 5 (static output)
- **Hosting**: Vercel (project `ludovicbostral`)
- **CSS**: Vanilla CSS with custom properties (Lens palette, inverted light mode)
- **Fonts**: Self-hosted Space Grotesk + JetBrains Mono (woff2 in public/fonts/)
- **Analytics**: GA4 (G-VXBFRGGZV3) with cookie consent, no ContentSquare
- **Booking**: /call redirects to Cal.com with language detection

## File Structure
```
/
├── astro.config.mjs       # Astro config: i18n, sitemap, Vercel adapter, redirects
├── src/
│   ├── content/
│   │   └── home/
│   │       ├── fr.json    # All FR text content (edit here to change copy)
│   │       └── en.json    # All EN text content
│   ├── content.config.ts  # Content collection schema
│   ├── layouts/
│   │   └── Base.astro     # HTML shell, meta, fonts, analytics, consent
│   ├── components/        # One .astro file per section
│   │   ├── Header.astro   # Sticky header with nav + FR/EN switch
│   │   ├── Footer.astro   # Single-line footer
│   │   ├── Hero.astro
│   │   ├── Now.astro
│   │   ├── Reports.astro
│   │   ├── Writing.astro
│   │   ├── TrackRecord.astro
│   │   ├── Stack.astro
│   │   ├── Advisory.astro
│   │   ├── Contact.astro
│   │   └── CookieConsent.astro
│   ├── pages/
│   │   ├── index.astro    # FR home
│   │   ├── privacy.astro
│   │   ├── terms.astro
│   │   └── en/
│   │       ├── index.astro  # EN home
│   │       ├── privacy.astro
│   │       └── terms.astro
│   ├── styles/
│   │   ├── fonts.css      # @font-face declarations
│   │   ├── variables.css  # CSS custom properties (Lens palette)
│   │   ├── base.css       # Reset + typography
│   │   └── components.css # All section styles
│   └── utils/
│       └── i18n.ts        # Content loader helper
├── public/
│   ├── fonts/             # 8 woff2 files
│   ├── call/index.html    # Cal.com redirect (preserved from old site)
│   ├── essais/            # Essay HTML pages (preserved from old site)
│   ├── assets/            # OG image, favicons, logos
│   └── robots.txt
└── scripts/
    └── build-essai.py     # Essay build tool (Markdown to HTML)
```

## Key Patterns

### Content Editing
All text is in JSON data files, not in components:
- Edit `src/content/home/fr.json` for French content
- Edit `src/content/home/en.json` for English content
- Components import and render this data via props

### CSS Variables (Lens palette, inverted)
```css
--bg-page: var(--cream-light);    /* #F5EDE1 */
--text-primary: var(--navy-deep); /* #0F1C26 */
--accent: var(--terracotta);      /* #A65D37 */
--accent-secondary: var(--gold);  /* #D4A944 */
```

### Sections Order (home page)
1. Hero (70vh, two-line H1)
2. Now (current activities)
3. Reports (5 Lens report cards)
4. Writing (essays, newsletter, Ludo Tries Things)
5. Track Record (timeline, 7 entries)
6. Stack (tools powering Streaming Radar)
7. Advisory (3 pillars, prose only)
8. Contact (CTA + secondary links)

## Common Tasks

### Change Text Content
Edit the corresponding JSON file in `src/content/home/`. No need to touch components.

### Add a New Section
1. Create `src/components/NewSection.astro`
2. Add styles in `src/styles/components.css`
3. Add data to both `fr.json` and `en.json`
4. Import and place in both `src/pages/index.astro` and `src/pages/en/index.astro`

### Add a New Essay
1. Place essay HTML/assets in `public/essais/[slug]/`
2. Add entry to the `writing.essays.items` array in both JSON files

## Development
```bash
npm install        # Install dependencies
npm run dev        # Dev server at localhost:4321
npm run build      # Build static site
npm run preview    # Preview build locally
```

## Deployment
Auto-deployed on Vercel on push to main. Static output to `.vercel/output/static/`.

## External Links
- Lens: https://lens.streaming-radar.com
- MCP: https://lens.streaming-radar.com/mcp
- Newsletter: https://www.streaming-radar.com
- LinkedIn: https://linkedin.com/in/ludovicbostral
- Calendar: https://www.bostral.com/call
- Email: ludovic@streaming-radar.com

---
> Source: [ludobos/ludovicbostral](https://github.com/ludobos/ludovicbostral) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-29 -->
