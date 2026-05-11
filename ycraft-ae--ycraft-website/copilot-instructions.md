## ycraft-website

> Marketing website for YCraft — professional After Effects extensions and scripts.

# YCraft Tools Website

## Project Overview
Marketing website for YCraft — professional After Effects extensions and scripts.
Products: STRUCT (layer management), TextFlow Pro (text styling), ExpressionKit (expression editor).

## Tech Stack
- **Pure HTML/CSS/JS** — no framework, no build step
- Open `index.html` directly in browser or serve with any static server
- Single page with show/hide navigation (no routing library)

## File Structure
```
ycraft-project/
├── index.html              # Main HTML — all sections & pages
├── css/
│   └── style.css           # All styles (1700+ lines)
├── js/
│   └── main.js             # Navigation, scroll effects, canvas BG
├── assets/
│   ├── images/             # Product screenshots, OG image (TODO)
│   └── pdf/
│       └── ycraft-install-guide.pdf
├── CLAUDE.md               # This file
└── README.md               # Project readme
```

## Architecture

### Navigation System
The site uses DOM show/hide instead of routing. All "pages" exist in the same HTML:
- `#pg-home` — Main landing page (hero, products, about, FAQ, etc.)
- `.detail-page` — Individual product pages (STRUCT, TextFlow Pro, ExpressionKit)
- `showPage(id)` / `goHome()` functions in main.js handle visibility

### CSS Organization (style.css)
Sections are marked with comment headers: `/* ═══════════ SECTION NAME ═══════════ */`
- CSS Variables (`:root`) — colors, fonts, spacing
- Background atmosphere
- Navigation
- Hero section (split layout: text left, panel right)
- Hero background (canvas-rendered)
- Buttons
- Product cards
- Stats, About, FAQ, Changelog
- Compatibility, Newsletter
- Footer
- Detail page styles
- Responsive breakpoints (900px, 600px)

### JS Organization (main.js)
- `showPage(id)` — Navigate to product detail page
- `goHome()` — Return to home
- Scroll observer (reveal animations)
- Nav scroll effect
- FAQ accordion
- Newsletter form handler
- Hero background canvas renderer
- Smooth scroll for anchor links

### Color Palette (CSS Variables)
```css
--neon-cyan: #00D4FF       /* Primary accent */
--neon-purple: #9999FF     /* Secondary */
--neon-orange: #FF9E64     /* CTA, warnings */
--neon-pink: #FF6AC1       /* Highlights */
--neon-green: #5DE8B2      /* Success, ExpressionKit */
--neon-yellow: #E8D44D     /* STRUCT brand color */
--neon-magenta: #D291FF    /* TextFlow brand color */
```

### Product Brand Colors
- **STRUCT** → Yellow (#E8D44D)
- **TextFlow Pro** → Purple (#9999FF / #D291FF)
- **ExpressionKit** → Green (#5DE8B2)

## Key Conventions
- No external JS dependencies
- All animations use CSS `@keyframes` or JS canvas
- Gradient stroke borders on product cards (via `::before` pseudo-element with `conic-gradient`)
- `reveal` class + IntersectionObserver for scroll-triggered fade-in
- Mobile: nav links hidden, grid collapses to single column

## Common Tasks

### Add a new product page
1. In `index.html`, duplicate a `.detail-page` section
2. Update `id="pg-{slug}"` and all internal references
3. Add nav link in `#navHome` list
4. Product card in `#pg-home` products section

### Change hero panel mockup
The right side of the hero contains a static mockup of TextFlow Pro.
Edit the `.hero__right > .hero__panel` HTML in index.html.

### Update hero background
The background is rendered via Canvas in `main.js` (search for `heroBgCanvas`).
Modify the radial gradient colors/positions in the `render()` function.

### Deployment
Static site — deploy to Netlify, Vercel, or any static host:
```bash
# Netlify
netlify deploy --prod --dir=.

# Or just upload the folder to any hosting
```

## TODO
- [ ] Replace mockup panels with real AE screenshots
- [ ] Create 30-60s video demos for each product
- [ ] Add blog section for SEO content
- [ ] Connect email signup to Mailchimp/Gumroad
- [ ] Implement license verification in extensions
- [ ] Purchase domain (ycraft.dev)
- [ ] Create OG image (assets/images/og-image.png)
- [ ] Add Google Analytics

---
> Source: [ycraft-ae/ycraft-website](https://github.com/ycraft-ae/ycraft-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-30 -->
