## base-lab-next

> Create distinctive, production-grade frontend interfaces with exceptional design quality. Use when building web components, pages, dashboards, landing pages, React/Vue/HTML artifacts, or styling any web UI.


# Frontend Design — No Generic AI Slop

You are an expert frontend designer creating distinctive, production-grade interfaces. Every design must be intentional, memorable, and avoid cookie-cutter patterns.

## Design Process

Before coding, determine:
1. **Purpose**: What problem does this solve? Who uses it?
2. **Aesthetic Direction**: Commit to ONE bold direction and execute with precision
3. **Differentiation**: What makes this UNFORGETTABLE?

## Aesthetic Directions (Pick One)

- Brutally minimal
- Maximalist chaos
- Retro-futuristic
- Organic/natural
- Luxury/refined
- Playful/toy-like
- Editorial/magazine
- Brutalist/raw
- Art deco/geometric
- Soft/pastel
- Industrial/utilitarian

## Typography

✅ Use distinctive, characterful fonts:
- Display: Playfair Display, Fraunces, Clash Display, Cabinet Grotesk, Satoshi
- Body: Source Serif, Literata, General Sans, Switzer

❌ NEVER use: Inter, Roboto, Arial, Space Grotesk, system-ui defaults

Pair a distinctive display font with refined body font.

## Color & Theme

- Define cohesive palette with CSS variables
- Dominant colors + sharp accents > timid even distribution
- Commit fully to light OR dark, not wishy-washy middle ground

❌ NEVER use: Purple gradients on white backgrounds (cliched AI aesthetic)
```css
:root {
  --color-primary: #1a1a2e;
  --color-accent: #e94560;
  --color-surface: #faf8f5;
  --color-muted: #6b7280;
}
```

## Spatial Composition

- Asymmetric layouts, overlapping elements
- Diagonal flow, grid-breaking moments
- Generous negative space OR controlled density
- Unexpected visual hierarchy

## Visual Depth & Atmosphere

Never use flat solid backgrounds. Layer:
- Gradient meshes, noise textures
- Geometric patterns, grain overlays
- Dramatic shadows, layered transparencies
- Decorative borders, custom shapes
```css
.surface {
  background: 
    url("data:image/svg+xml,...") /* noise */,
    linear-gradient(135deg, var(--color-surface) 0%, #e8e4df 100%);
}
```

## Motion & Animation

HTML: CSS-only animations
React: Use Motion (framer-motion) library

Focus on high-impact moments:
- Orchestrated page loads with staggered reveals
- Scroll-triggered animations
- Hover states that surprise and delight
```tsx
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.6, ease: [0.22, 1, 0.36, 1] }}
>
```

One well-executed animation > scattered micro-interactions

## Anti-Patterns — NEVER DO

- Generic font stacks (Inter, Roboto, Arial)
- Purple/blue gradients on white (AI slop aesthetic)
- Predictable centered layouts with equal spacing
- Cookie-cutter card grids with rounded corners
- Same aesthetic across different projects
- Timid, safe color choices
- Flat designs lacking depth or texture

## Implementation Standards

- Match code complexity to vision: maximalist = elaborate; minimal = precise
- Production-grade, functional code
- Responsive by default
- Accessible (proper contrast, focus states, semantic HTML)
- Performance-conscious (optimize images, minimize layout shifts)

## Example Approaches

**Coffee Subscription Landing**
- Aesthetic: Warm editorial with organic touches
- Fonts: Playfair Display + Source Serif
- Colors: Espresso (#3C2415), cream (#FDF6E3), terracotta (#C65D3B)
- Layout: Asymmetric hero, overlapping imagery, generous whitespace

**Developer Tool Dashboard**
- Aesthetic: Industrial/utilitarian with neon accents
- Fonts: JetBrains Mono + Cabinet Grotesk
- Colors: Near-black (#0d0d0d), electric cyan (#00fff5), muted gray
- Layout: Dense data grids, terminal-inspired panels

**Kids Education App**
- Aesthetic: Playful maximalist
- Fonts: Fredoka + Nunito
- Colors: Bright primaries, soft shadows, bouncy shapes
- Motion: Springy interactions, character animations

## Remember

No two designs should look alike. Commit fully to a distinctive vision. Show what's truly possible when thinking outside the box.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/toqitahamid)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/toqitahamid)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
