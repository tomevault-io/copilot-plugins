## juicy

> This project is the official landing page for **Juicy Hamburgers**, a premium burger brand from **Villa Carlos Paz**, with **2 existing locations** and a **future Buenos Aires launch**.

# AGENTS.md — Juicy Hamburgers Landing

## 1. Project Context

This project is the official landing page for **Juicy Hamburgers**, a premium burger brand from **Villa Carlos Paz**, with **2 existing locations** and a **future Buenos Aires launch**.

The site must feel:

- award-worthy
- modern
- fast
- premium
- highly visual
- conversion-oriented

This is **not** a generic restaurant site.
It must communicate **desire**, **brand attitude**, and **place identity**.

Primary slogan:

- **Taste the Difference**

Primary business goals:

- drive traffic to physical locations
- support delivery intent
- capture leads for Buenos Aires waitlist
- strengthen brand perception

---

## 2. Core Stack

- **Next.js** (App Router)
- **TypeScript**
- **Tailwind CSS**
- **GSAP** for premium motion and scroll-based storytelling

Preferred implementation style:

- server-first where possible
- client components only when interactivity/animation requires it
- clean composition
- reusable sections and UI primitives
- strong visual consistency
- minimal dependencies

---

## 3. Product Principles

Every contribution to this codebase must follow these principles:

1. **Brand before decoration**
   - visuals must reinforce Juicy’s identity
   - never add trendy UI just for the sake of it

2. **Motion with intention**
   - animation must support hierarchy, reveal, delight, or conversion
   - avoid noisy, random, or excessive movement

3. **Product-first storytelling**
   - the burgers are the heroes
   - content should make the food feel desirable immediately

4. **Fast by default**
   - performance is part of the design
   - heavy effects must be justified and optimized

5. **Mobile-first experience**
   - most users will likely discover the brand on mobile
   - every section must work beautifully on small screens first

6. **Premium minimalism**
   - bold, memorable, retro-american inspiration
   - but always polished, modern, and restrained

---

## 4. Visual Direction

The brand aesthetic combines:

- retro-american burger culture
- premium editorial presentation
- clean contemporary layout systems
- warm, tactile materials and subtle texture

### Brand Colors

- juicy-red: `#C41E1E`
- juicy-red-dark: `#8B1010`
- juicy-red-light: `#E84040`
- juicy-cream: `#FAF7F0`
- juicy-cream-dark: `#F0EBE0`
- juicy-black: `#1A1008`
- juicy-gray: `#8A8070`
- juicy-white: `#FFFFFF`

### Typography

- Display / logo vibe: `Pacifico` or `Satisfy`
- Headlines: `Bebas Neue`
- Body/UI: `DM Sans`
- Accent phrases: `Playfair Display Italic`

### Visual Motifs

- red/white checkerboard
- subtle grain
- kraft/paper-inspired warmth
- bold food photography
- red illustrated dog mascot (“Juicy Dog”)

---

## 5. Section Strategy

The landing is composed around these sections:

1. Hero
2. Philosophy
3. Menu Showcase
4. Immersive Gallery / The Vibe
5. Locations + Buenos Aires Waitlist
6. Social Proof / Reviews
7. Footer

When building or editing sections:

- preserve narrative flow
- maintain strong contrast between sections
- avoid repetitive layouts
- each section must have a clear visual idea

---

## 6. Motion Rules (GSAP)

GSAP is a core part of the experience, but it must follow strict rules:

### Use GSAP for:

- hero entrances
- staggered reveals
- scroll-triggered parallax
- marquee motion
- mascot reactions
- hover interactions on food/product cards
- clip-path reveals when they clearly improve perceived quality

### Avoid:

- animating everything on screen at once
- long blocking intro animations
- motion that delays access to content
- excessive bouncing or cartoon-like easing unless it fits the mascot

### Motion style

- confident
- premium
- playful in controlled amounts
- smooth, high-end, tactile

### Technical rules

- initialize GSAP only on client
- always clean up contexts/timelines
- use `prefers-reduced-motion`
- avoid layout thrashing
- use transforms and opacity first
- scroll-based animations must feel performant on mid-range mobile devices

---

## 7. Performance Rules

Performance is a product requirement.

Always prefer:

- `next/image`
- `next/font`
- lazy loading where appropriate
- optimized image sizes and formats
- minimal client-side JS
- section-level code separation if justified
- transform/opacity animations over layout-changing animations

Avoid:

- oversized hero images without optimization
- unnecessary animation libraries beyond GSAP
- large third-party widgets unless clearly valuable
- shipping decorative code with no business or UX value

---

## 8. Accessibility Rules

Accessibility is mandatory, not optional.

Always ensure:

- semantic HTML
- visible focus states
- keyboard navigability
- sufficient contrast
- descriptive alt text
- accessible forms
- reduced-motion handling
- links and buttons are clearly distinguishable

No visual change should break:

- keyboard navigation
- focus visibility
- reading order
- screen reader understanding

---

## 9. SEO & Content Rules

This is a local-business website, so SEO must support discovery.

Priorities:

- strong metadata per page
- local relevance
- structured, readable headings
- location-based copy
- optimized Open Graph
- sitemap/robots when applicable
- schema markup if useful for restaurant/local business context

Tone of copy:

- short
- bold
- sensory
- brand-aware
- never generic corporate restaurant copy

Avoid:

- filler text
- cliché burger marketing language
- exaggerated claims without support
- overly long paragraphs

---

## 10. Code Style Rules

### TypeScript

- strict typing
- avoid `any`
- type props explicitly
- prefer narrow, descriptive types
- extract reusable interfaces when repeated

### React / Next.js

- default to Server Components
- use Client Components only when needed for interactivity, browser APIs, or GSAP
- keep components focused and composable
- separate content/data/config from view logic when possible

### Tailwind

- prefer clean utility composition
- avoid unreadable class explosions
- extract repeated patterns into reusable components/helpers
- preserve spacing rhythm and visual consistency

### File naming

- components: `PascalCase.tsx`
- hooks: `useSomething.ts`
- utils/config: `camelCase.ts`
- route files follow Next.js conventions

---

## 11. Preferred Architecture

Suggested structure:

- `app/`
- `components/sections/`
- `components/ui/`
- `components/animations/`
- `lib/`
- `data/`
- `types/`
- `public/images/`

### Architecture preference

- sections are top-level composable blocks
- reusable UI lives separately from sections
- animations live in isolated hooks/helpers where possible
- static content/config should be centralized
- avoid mixing content, animation, and layout logic in giant files

---

## 12. Content & Assets Expectations

The project depends on real brand assets.
Use placeholders only in a way that is easy to replace later.

Expected assets:

- burger photography
- location imagery
- Juicy logo
- Juicy Dog illustrations
- reviews
- maps/location links
- delivery links
- Buenos Aires launch content

When assets are missing:

- create structure that degrades gracefully
- never hardcode fake production content without clearly marking it as placeholder-safe

---

## 13. Definition of Done

A task is only done when it is:

- visually aligned with Juicy’s brand
- responsive
- performant
- accessible
- type-safe
- easy to maintain
- consistent with the section narrative
- polished enough to feel premium

If a solution is technically correct but visually generic, it is **not done**.

---

## 14. How the Agent Should Behave

When working in this repo, the agent should:

- prioritize brand consistency over generic patterns
- propose premium UI solutions, not boilerplate layouts
- preserve performance while adding motion
- think in sections, storytelling, and conversion
- challenge weak design decisions
- avoid unnecessary libraries
- prefer maintainable abstractions
- keep copy concise and brand-aligned
- treat the mascot, burger imagery, and typography as key brand assets

When suggesting changes, the agent should evaluate:

1. visual quality
2. conversion impact
3. performance cost
4. accessibility impact
5. maintainability

---

## 15. Skill Routing Guidance

Use these skills when relevant:

- `frontend-design`  
  For visual hierarchy, layout refinement, spacing, typography, and section composition.

- `gsap`  
  For timelines, ScrollTrigger, parallax, hover motion, reveal systems, and animation cleanup.

- `nextjs-seo`  
  For metadata, sitemap, robots, JSON-LD, OG, and local-business SEO improvements.

- `web-accessibility`  
  For WCAG review, keyboard flow, semantic structure, and reduced-motion support.

- `nextjs-performance`  
  For Core Web Vitals, image/font optimization, caching, and reducing unnecessary client-side code.

- `skill-creator`  
  For creating new skills.

---

## 16. Project-Specific Custom Skills To Add

Create and maintain these custom skills for this repo:

1. `juicy-brand-system`
   - codifies colors, type hierarchy, spacing, checker motifs, textures, mascot usage, and premium retro rules

2. `juicy-section-composer`
   - guides section creation/editing using Juicy’s storytelling, conversion goals, and section rhythm

3. `juicy-motion-direction`
   - codifies how GSAP motion should feel for Juicy specifically, including hero motion, scroll reveals, mascot behavior, and hover rules

These custom skills should always reinforce this AGENTS.md, not replace it.

---
> Source: [NicoEspin/Juicy](https://github.com/NicoEspin/Juicy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
