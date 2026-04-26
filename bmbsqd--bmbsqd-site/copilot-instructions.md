## bmbsqd-site

> Modern Astro site with Tailwind CSS v4, Biome, and CloudFlare Pages deployment.

# bmbsqd-site

Modern Astro site with Tailwind CSS v4, Biome, and CloudFlare Pages deployment.
Premium "Material-Minimalism Showroom" design aesthetic with stone-tone palette, warm wood accents, and subtle WebGL effects.

## Tech Stack

- **Astro** - Static site generator with React component islands
- **Tailwind CSS v4** - Utility-first CSS with `@theme` blocks and CSS custom properties
- **Biome** - Fast linter and formatter (replaces ESLint + Prettier)
- **Bun** - Fast JS runtime and package manager (note: `bun` may not be in PATH, use `npx` as fallback for builds)
- **CloudFlare Pages** - Edge deployment platform
- **TypeScript** - Type-safe development
- **Sharp** - High-performance image optimization
- **Framer Motion** - Animation library for React components
- **OGL** - Lightweight WebGL library for shader effects

## Development Setup

```bash
# Install dependencies
bun install

# Start development server
bun run dev

# Build for production
bun run build
# or if bun not in PATH:
npx astro build

# Preview production build
bun run preview
```

## Deployment

```bash
# Deploy to CloudFlare Pages
bun run deploy
```

## Linting & Formatting

```bash
# Lint
bun run lint

# Lint + auto-fix
bun run lint:fix

# Format
bun run format
```

Biome handles both linting and formatting. Config in `biome.json`.
Note: `.astro` files are excluded from Biome (it can't parse Astro template syntax).

## Design System

### Color Palette (Stone Tones)

Defined as CSS custom properties in `src/styles/global.css`:

| Token | Hex | Usage |
|-------|-----|-------|
| `--stone-50` | `#FAF7F5` | Page background |
| `--stone-100` | `#F5F5F4` | Card backgrounds, secondary bg |
| `--stone-200` | `#E7E5E4` | Borders, dividers |
| `--stone-300` | `#D6D3D1` | Hover borders, subtle accents |
| `--stone-400` | `#A8A29E` | Muted text, labels |
| `--stone-500` | `#78716C` | Body text |
| `--stone-900` | `#1C1917` | Headings, primary text, button bg |
| `--warm-wood` | `#A67B5B` | Accent color (checkmarks, highlights, button shine) |

### Typography

Fonts loaded via **Fontshare CDN** in `BaseLayout.astro`:
- **Cabinet Grotesk** (800, 700, 500) - Headings (`--font-heading`)
- **Satoshi** (400, 500, 700) - Body text (`--font-sans`)

Headings use `letter-spacing: -0.02em` globally. The `tracking-tighter` Tailwind token is also set to `-0.02em`.

### Key CSS Classes

Defined in `src/styles/global.css` `@layer utilities`:

- **`.showroom-card`** - White card with 2px transparent border, 1.5rem radius, hover scale + shadow
- **`.btn-showroom`** - Black pill button with animated shiny conic-gradient border (warm-wood highlight), uses `@property` CSS animations
- **`.section-label`** - Uppercase label with horizontal line decoration
- **`.material-badge`** - Frosted glass badge with backdrop-filter
- **`.hover-lift`** - Hover translateY + shadow animation

### Button Shine Effect

The `.btn-showroom` class uses CSS `@property` for animated gradient borders:
- Conic gradient rotates continuously around button border
- Warm-wood colored shine (`--warm-wood` / `#c4956b`)
- On hover: gradient spreads wider, animation speeds up
- Pure CSS, no JS hydration needed

## Project Structure

```
src/
├── layouts/
│   └── BaseLayout.astro        # Base layout with Fontshare CDN, meta, structured data
├── pages/
│   └── index.astro             # Homepage with nav, sections, footer
├── components/
│   ├── ui/                     # React component islands
│   │   ├── scroll-reveal.tsx   # Framer Motion scroll-triggered fade-in
│   │   ├── parallax-stars-bg.tsx # CSS box-shadow particle animation (hero bg)
│   │   ├── plasma.tsx          # WebGL plasma shader effect (OGL)
│   │   ├── shiny-text.tsx      # Framer Motion gradient text animation
│   │   ├── client-logo-marquee.tsx # Auto-scrolling logo strip
│   │   └── rainbow-button.tsx  # Rainbow gradient button
│   ├── HeroSection.astro       # Hero with ParallaxStarsBg wrapper
│   ├── GlobalPresence.astro    # City cards (LA/Stockholm/Singapore) with photos
│   ├── TrackRecord.astro       # Editorial layout with stat cards + photo
│   ├── TechnologyExpertise.astro # Expertise cards with Plasma bg
│   ├── ServiceOfferings.astro  # Service cards grid
│   └── ContactSection.astro    # CTA section with Plasma bg
├── styles/
│   └── global.css              # Theme, color system, component classes, animations
└── lib/
    └── utils.ts                # Utility functions (cn helper)
public/
├── images/
│   ├── cities/                 # City photos (Unsplash, free license)
│   │   ├── los-angeles.jpg
│   │   ├── stockholm.jpg
│   │   └── singapore.jpg
│   └── track-record.jpg        # Strategy meeting photo
astro.config.ts                 # Astro configuration
biome.json                      # Biome linter/formatter config
wrangler.toml                   # CloudFlare deployment config
```

## Page Sections (index.astro)

1. **Sticky Nav** - Frosted glass header with ShinyText "BOMBSQUAD" logo (warm-wood shine), nav links, shiny CTA button
2. **Hero** - ParallaxStarsBg (stone-colored particles), centered headline + CTA
3. **Client Logo Marquee** - Auto-scrolling logo strip
4. **Global Presence** - 3 showroom-cards with city photos (LA, Stockholm, Singapore)
5. **Track Record** - 2-column: editorial text + photo left, staggered stat cards right
6. **Technology Expertise** - 3 white cards on Plasma WebGL background, bordered section
7. **Service Offerings** - 5 showroom-cards in responsive grid
8. **Contact** - CTA section with Plasma WebGL background
9. **Footer** - ShinyText logo, city names, copyright

## React Component Islands

Components use Astro's hydration directives:
- `client:load` - Hydrate immediately (nav logo ShinyText, ParallaxStarsBg)
- `client:visible` - Hydrate when scrolled into view (ScrollReveal, Plasma, footer ShinyText)

### Plasma Component (`ui/plasma.tsx`)
- WebGL shader via OGL (`Renderer`, `Program`, `Triangle`, `Mesh`)
- Supports children (wraps content, canvas renders behind)
- Used on TechnologyExpertise and ContactSection
- Current settings: `color="#A8A29E"`, `speed={0.3}`, `opacity={0.2}`, `mouseInteractive={false}`

### ParallaxStarsBg (`ui/parallax-stars-bg.tsx`)
- CSS box-shadow based particle layers (1200 small, 400 medium, 150 large)
- Stone-colored particles (#D6D3D1, #A8A29E, #78716C)
- Wraps hero section content

### ShinyText (`ui/shiny-text.tsx`)
- Framer Motion `useAnimationFrame` for gradient sweep animation
- Used for "BOMBSQUAD" logo in nav and footer
- Nav: dark text with warm-wood shine, Footer: stone-400 with stone-300 shine

## Code Style

- Use **2-space indentation** (not tabs)
- Prefer **camelCase** for JavaScript/TypeScript
- Use **kebab-case** for file names and directories
- Prefer **functional style** with ternary operators and `.map()`
- Use **real functions** instead of arrow functions where applicable
- Use **single quotes**, no semicolons, trailing commas
- Use **CSS custom properties** (`var(--stone-*)`) for colors
- Avoid unnecessary comments, use descriptive names instead
- Import from `framer-motion` (not `motion/react`)

## CloudFlare Pages Notes

- Static output optimized for edge deployment
- Automatic compression and sitemap generation
- Image optimization with responsive styles
- Prefetch strategy for improved performance

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/bmbsqd) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
