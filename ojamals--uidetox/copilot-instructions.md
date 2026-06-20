## uidetox

> Eliminates AI slop from frontend code. Combines design taste enforcement, anti-pattern detection, and structured remediation. Use this skill whenever generating or reviewing HTML, CSS, React, Vue, Svelte, or any frontend UI code.


# UIdetox — Combined Design Skill

## 1. ACTIVE BASELINE CONFIGURATION

* DESIGN_VARIANCE: 8 (1=Perfect Symmetry, 10=Artsy Chaos)
* MOTION_INTENSITY: 6 (1=Static/No movement, 10=Cinematic/Magic Physics)
* VISUAL_DENSITY: 4 (1=Art Gallery/Airy, 10=Pilot Cockpit/Packed Data)

**AI Instruction:** The standard baseline is strictly set to these values (8, 6, 4). Do not ask the user to edit this file. Adapt these values dynamically based on what the user explicitly requests. Use these values to drive the logic in Sections 3 through 8.

---

## 2. DESIGN DIRECTION

Commit to a BOLD aesthetic direction before writing a single line of code:
- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick an extreme — brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, industrial/utilitarian. There are infinite flavors.
- **Constraints**: Technical requirements (framework, performance, accessibility).
- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work — the key is intentionality, not intensity.

Then implement working code that is:
- Production-grade and functional
- Visually striking and memorable
- Cohesive with a clear aesthetic point-of-view
- Meticulously refined in every detail

---

## 3. DESIGN ENGINEERING DIRECTIVES (Bias Correction)

LLMs have statistical biases toward specific UI cliché patterns. Override them with these rules:

### Rule 1: Deterministic Typography

→ *Consult [typography reference](reference/typography.md) for scales, pairing, and loading strategies.*

* **Display/Headlines:** Default to large, tight tracking, reduced line-height. Headlines should feel heavy and intentional.
  * **ANTI-SLOP:** `Inter`, `Roboto`, `Arial`, `Open Sans`, system defaults are BANNED for creative or premium vibes. Force unique character using `Geist`, `Outfit`, `Cabinet Grotesk`, `Satoshi`, or a distinctive display font.
  * **TECHNICAL UI RULE:** Serif fonts are strictly BANNED for Dashboard/Software UIs. Use exclusively high-end Sans-Serif pairings (`Geist` + `Geist Mono` or `Satoshi` + `JetBrains Mono`).
* **Body/Paragraphs:** Readable sizes (14-16px body), limit paragraph width to ~65 characters, generous line-height.
* **Weight Spectrum:** Use Medium (500) and SemiBold (600), not just Regular and Bold.
* **Numbers:** Use monospace or enable `font-variant-numeric: tabular-nums` for data interfaces.
* **Letter-spacing:** Negative tracking for large headers, positive tracking for small caps or labels.
* **Don't** put large icons with rounded corners above every heading — they rarely add value and make sites look templated.

### Rule 2: Color Calibration

→ *Consult [color reference](reference/color-and-contrast.md) for OKLCH, palettes, and dark mode.*

* **Constraint:** Max 1 Accent Color. Saturation < 80%.
* **THE AI PALETTE BAN:** Purple/blue gradients, cyan-on-dark, neon accents on dark backgrounds — all BANNED. These are the fingerprints of AI-generated work. Use absolute neutral bases (Zinc/Slate) with high-contrast, singular accents (Emerald, Electric Blue, or Deep Rose).
* **COLOR CONSISTENCY:** Stick to one palette for the entire output. Never mix warm and cool grays. Tint all neutrals toward your brand hue.
* **GRAY ON COLOR:** Never put gray text on colored backgrounds — it looks washed out. Use a shade of the background color instead.
* **NO PURE BLACK:** Never use `#000000`. Use off-black, zinc-950, or tinted dark.
* **NO GRADIENT TEXT:** Do not use text-fill gradients for "impact" — especially on metrics or headings.
* Use modern CSS color functions (oklch, color-mix, light-dark) for perceptually uniform palettes.
* **Color Priority Order:**
  1. Use existing colors from the user's project (search for them by reading config files)
  2. Get inspired from the curated palettes in [color-palettes reference](reference/color-palettes.md)
  3. Never invent random color combinations

### Rule 3: Layout Diversification

→ *Consult [spatial reference](reference/spatial-design.md) for grids, rhythm, and container queries.*

* **ANTI-CENTER BIAS:** Centered Hero/H1 sections are BANNED when DESIGN_VARIANCE > 4. Force "Split Screen" (50/50), "Left Aligned content/Right Aligned asset", or "Asymmetric White-space" structures.
* **3-COLUMN CARD BAN:** The generic "3 equal cards horizontally" feature row is BANNED. Use a 2-column zig-zag, asymmetric grid, horizontal scroll, or masonry layout.
* **Container Constraint:** Always use max-width (1200-1440px) with auto margins.
* **Grid over Flex-Math:** Never use complex flexbox percentage math. Always use CSS Grid for reliable structures.
* **Viewport Stability:** Never use `h-screen` for full-height sections. Always use `min-h-[100dvh]`.
* Create visual rhythm through varied spacing — tight groupings, generous separations. **NO SPACING REPETITION:** Avoid overusing identical spacing utilities (like repeating `p-4` or `gap-4` five times). Mix scales to create rhythm.
* Use asymmetry and unexpected compositions. Break the grid intentionally for emphasis.

### Rule 4: Materiality & Surfaces

* **CARD OVERUSE BAN:** For VISUAL_DENSITY > 7, generic card containers are BANNED. Use `border-top`, `divide-y`, or negative space. Data should breathe without being boxed.
* Use cards ONLY when elevation communicates hierarchy. When a shadow is used, tint it to the background hue.
* **NO GLASSMORPHISM DEFAULT:** Don't use blur effects, glass cards, or glow borders decoratively. If glassmorphism is needed, go beyond `backdrop-blur` — add a 1px inner border (`border-white/10`) and a subtle inner shadow for physical edge refraction.
* Don't wrap everything in cards. Don't nest cards inside cards.
* Don't use rounded rectangles with generic drop shadows.
* **NO OPACITY ABUSE:** Avoid excessive layering of continuous transparent elements (like stacking multiple `opacity-50` or `bg-white/10`). Use solid surface colors; reserve transparency solely for overlays and modals.

### Rule 5: Interactive UI States

→ *Consult [interaction reference](reference/interaction-design.md) for forms, focus, and loading patterns.*

* **Mandatory Generation:** LLMs naturally generate "static" successful states. You MUST implement full interaction cycles:
  * **Hover:** Subtle scale, color shift, or shadow change.
  * **Focus:** Visible keyboard focus indicators (accessibility requirement).
  * **Active:** `-translate-y-[1px]` or `scale-[0.98]` to simulate a physical push.
  * **Loading:** Skeletal loaders matching layout sizes (never generic circular spinners).
  * **Empty States:** Composed states indicating how to populate data.
  * **Error States:** Clear, inline error reporting.
* Progressive disclosure — start simple, reveal sophistication through interaction.
* Make every interactive surface feel intentional and responsive.

### Rule 6: Data & Form Patterns

* Label MUST sit above input. Helper text optional. Error text below input.
* Use standard gap between input blocks.
* Make every button primary hierarchy explicit — use ghost buttons, text links, secondary styles.

---

## 4. ANTI-PATTERN CATALOG (Consolidated Ban List)

→ *Full reference at [anti-patterns reference](reference/anti-patterns.md).*

These are the fingerprints of AI-generated UI. If you see yourself reaching for any of these, **stop and pick the harder, cleaner option.**

### Keep It Normal (The Uncodixfy Standard)

Think Linear. Think Raycast. Think Stripe. Think GitHub. They don't grab attention. They just work.

- Sidebars: 240-260px fixed width, solid background, simple border-right. No floating shells, no rounded outer corners.
- Headers: Simple text hierarchy. No eyebrows, no uppercase labels, no gradient text.
- Sections: Standard padding (20-30px). No hero blocks inside dashboards. No decorative copy.
- Navigation: Simple links, subtle hover states. No transform animations, no badges unless functional.
- Buttons: Solid fills or simple borders, 8-10px radius max. No pill shapes, no gradient backgrounds.
- Cards: 8-12px radius max, subtle borders, shadows under 8px blur. No floating effect.
- Inputs: Solid borders, simple focus ring. No animated underlines, no morphing shapes.
- Transitions: 100-200ms ease. No bouncy animations. Simple opacity/color changes.
- Icons: 16-20px, consistent stroke width, monochrome or subtle color. No decorative icon backgrounds.

### Hard No — Banned Patterns

- ❌ No oversized rounded corners (20-32px range on everything)
- ❌ No pill overload
- ❌ No floating glassmorphism shells as default visual language
- ❌ No soft corporate gradients used to fake taste
- ❌ No decorative sidebar blobs
- ❌ No "control room" cosplay unless explicitly requested
- ❌ No serif headline + system sans fallback as shortcut to "premium"
- ❌ No `Segoe UI`, `Trebuchet MS`, `Arial`, `Inter`, `Roboto` unless the product already uses them
- ❌ No sticky left rail unless information architecture truly needs it
- ❌ No metric-card grid as first instinct
- ❌ No fake charts that exist only to fill space
- ❌ No random glows, blur haze, frosted panels, or conic-gradient donuts as decoration
- ❌ No "hero section" inside internal UI unless there is a real product reason
- ❌ No overpadded layouts
- ❌ No ornamental labels like "live pulse", "night shift" unless from product voice
- ❌ No generic startup copy
- ❌ No style decisions made because they are easy to generate
- ❌ No `<small>` eyebrow headers
- ❌ No rounded `<span>` tags for decoration
- ❌ No colors trending toward blue by default — dark muted colors are best
- ❌ No neon/outer glows or auto-glows
- ❌ No oversaturated accents
- ❌ No custom mouse cursors
- ❌ No bounce or elastic easing — they feel dated and tacky
- ❌ No hero metric layout templates (big number, small label, gradient accent)
- ❌ No modals unless there's truly no better alternative
- ❌ No sparklines as decoration — tiny charts that convey nothing meaningful
- ❌ No monospace typography as lazy shorthand for "technical/developer" vibes
- ❌ No dark mode with glowing accents as a substitute for actual design decisions
- ❌ No identical card grids (same-sized cards with icon + heading + text repeated)
- ❌ No redundant headers that restate intros

### Specifically Banned Code/CSS Patterns

- Border radii in 20-32px range across sidebar, cards, buttons, panels simultaneously
- Floating detached sidebar with rounded outer shell
- Canvas chart in a glass card with no reason
- Donut chart with hand-wavy percentages
- UI cards using glows instead of hierarchy
- Dramatic box shadows (`0 24px 60px rgba(0,0,0,0.35)`)
- Status indicators with `::before` pseudo-elements creating colored dots
- Muted labels with uppercase + letter-spacing overuse
- Pipeline bars with gradient fills
- KPI cards in a grid as default dashboard layout
- Brand marks with gradient backgrounds
- Nav badges showing "Live" status
- Footer meta lines like "dashboard • dark mode • single-file HTML"
- Transform animations on hover (`translateX(2px)` on nav links)

### Content Anti-Patterns (The "Jane Doe" Effect)

- ❌ No generic names ("John Doe", "Sarah Chan") — use diverse, realistic names
- ❌ No generic avatars (SVG "egg" icons) — use creative placeholders
- ❌ No fake round numbers (`99.99%`, `50%`) — use organic data (`47.2%`, `$1,287.34`)
- ❌ No startup slop names ("Acme", "Nexus", "SmartFlow") — invent premium, contextual brands
- ❌ No AI copy clichés ("Elevate", "Seamless", "Unleash", "Next-Gen", "Delve", "Tapestry") — use concrete verbs
- ❌ No exclamation marks in success messages
- ❌ No "Oops!" error messages — be direct
- ❌ No Lorem Ipsum — write real draft copy
- ❌ No broken Unsplash links — use `https://picsum.photos/seed/{name}/800/600`

---

## 5. CREATIVE ARSENAL (High-End Inspiration)

→ *Full reference at [creative-arsenal reference](reference/creative-arsenal.md).*

Do not default to generic UI. Pull from this library when building visually striking interfaces:

### Navigation
- Mac OS Dock Magnification — icons scale on hover
- Magnetic Buttons — buttons pull toward the cursor
- Gooey Menu — sub-items detach like viscous liquid
- Dynamic Island — pill-shaped morphing status component
- Contextual Radial Menu — circular menu at click coordinates
- Floating Speed Dial — FAB that springs into secondary actions
- Mega Menu Reveal — full-screen stagger-fade dropdowns

### Layout
- Bento Grid — asymmetric, tile-based grouping
- Masonry Layout — staggered grid without fixed row heights
- Split Screen Scroll — two halves sliding in opposite directions
- Curtain Reveal — hero parting like a curtain on scroll
- Chroma Grid — continuously animating color gradients on borders/tiles

### Cards & Containers
- Parallax Tilt Card — 3D-tilting tracking mouse coordinates
- Spotlight Border Card — borders illuminate under cursor
- Holographic Foil Card — iridescent reflections on hover
- Morphing Modal — button expands into its own full-screen dialog

### Scroll & Motion
- Sticky Scroll Stack — cards stick and physically stack
- Horizontal Scroll Hijack — vertical scroll → horizontal gallery pan
- Zoom Parallax — background image zooming on scroll
- Scroll Progress Path — SVG lines drawing as user scrolls
- Liquid Swipe Transition — viscous liquid page wipes

### Typography & Text
- Kinetic Marquee — text bands reversing direction on scroll
- Text Mask Reveal — large text as transparent window to video
- Text Scramble Effect — Matrix-style character decoding
- Gradient Stroke Animation — outlined text with running gradient

### Micro-Interactions
- Particle Explosion Button — CTAs shattering on success
- Skeleton Shimmer — shifting light across placeholder boxes
- Directional Hover Aware Button — fill entering from mouse direction
- Mesh Gradient Background — organic animated color blobs

---

## 6. MOTION ENGINE

→ *Consult [motion reference](reference/motion-design.md) for timing, easing, and reduced motion.*

### Principles
- Focus on high-impact moments: one well-orchestrated page load beats scattered micro-interactions
- Use exponential easing (ease-out-quart/quint/expo) for natural deceleration
- NEVER use bounce or elastic easing — they feel dated
- Animate only `transform` and `opacity` — never layout properties
- Respect `prefers-reduced-motion` always

### Timing
- **100-150ms**: Button press, toggle feedback
- **200-300ms**: Hover states, menu open, state changes
- **300-500ms**: Accordion, modal, layout changes
- **500-800ms**: Page load entrance animations

### Easing Curves
```css
--ease-out-quart: cubic-bezier(0.25, 1, 0.5, 1);
--ease-out-quint: cubic-bezier(0.22, 1, 0.36, 1);
--ease-out-expo: cubic-bezier(0.16, 1, 0.3, 1);
```

### When MOTION_INTENSITY > 5
- Embed continuous micro-animations (pulse, float, shimmer, carousel) in standard components
- Apply spring physics (`type: "spring", stiffness: 100, damping: 20`) to interactive elements
- Stagger children entries — never mount lists instantly
- Use `layout` and `layoutId` for smooth shared-element transitions
- **PERFORMANCE CRITICAL:** Any perpetual motion must be isolated in its own component. Never trigger parent re-renders.

### When MOTION_INTENSITY > 7
- Magnetic micro-physics: buttons pull toward cursor. **Never** use `useState` for continuous animations — use `useMotionValue` and `useTransform` outside React render cycle.
- Scroll-triggered reveals and parallax via Intersection Observer or Framer Motion hooks
- For GSAP/ThreeJS: wrap in strict `useEffect` cleanup blocks; never mix with Framer Motion

---

## 7. REDESIGN PROTOCOL (Existing Projects)

When upgrading an existing project, run `uidetox loop` to automatically orchestrate this sequence, or follow it manually:

### Step 1: Scan (`uidetox scan`)
Read the codebase. Identify framework, styling method, and current design patterns. Run the static analyzer.

### Step 2: Diagnose
Run through the full audit checklist:
- Typography (fonts, hierarchy, weights, spacing)
- Color (palette, contrast, consistency, AI fingerprints)
- Layout (symmetry issues, card overuse, container widths)
- Interactivity (missing hover/focus/active/loading/error/empty states)
- Content (generic names, fake data, AI copy)
- Component patterns (card abuse, modal overuse, accordion FAQs)
- Iconography (Lucide/Feather defaults, inconsistent stroke widths)
- Code quality (div soup, inline styles, broken imports)
- Strategic omissions (legal links, 404, form validation, skip-to-content)

### Step 3: Fix (`uidetox next` → `uidetox batch-resolve`)
1. Font swap — biggest instant improvement, lowest risk
2. Color palette cleanup — remove AI-purple, oversaturation
3. Hover and active states — makes interface feel alive
4. Layout and spacing — proper grid, max-width, consistent padding
5. Replace generic components — swap cliché patterns for modern alternatives
6. Add loading, empty, and error states — makes it feel finished
7. Polish typography scale and spacing — the premium final touch

### Rules
- Work with the existing tech stack. Do not migrate frameworks.
- Do not break existing functionality. Test after every change.
- Check dependencies before importing new libraries.
- Execute fixes at the component-level using `uidetox batch-resolve` to maintain atomic history.

---

## 8. OUTPUT ENFORCEMENT (Anti-Laziness)

### Banned Output Patterns

**In code:** `// ...`, `// rest of code`, `// implement here`, `// TODO`, `/* ... */`, `// similar to above`, `// continue pattern`, bare `...` standing in for omitted code

**In prose:** "Let me know if you want me to continue", "for brevity", "the rest follows the same pattern", "and so on" (when replacing actual content)

**Structural shortcuts:** Outputting a skeleton when a full implementation was requested. Showing first and last section while skipping middle. Replacing repeated logic with one example and a description.

### Execution Process
1. **Scope** — Count how many distinct deliverables are expected. Lock that number.
2. **Build** — Generate every deliverable completely. No partial drafts.
3. **Cross-check** — Before output, re-read the request. Compare deliverable count against scope count. If anything is missing, add it.

### Handling Long Outputs
When approaching token limit:
- Write at full quality up to a clean breakpoint
- End with: `[PAUSED — X of Y complete. Send "continue" to resume from: next section]`
- On "continue", pick up exactly where you stopped. No recap.

---

## 9. ARCHITECTURE & CONVENTIONS

### Dependency Verification [MANDATORY]
Before importing ANY 3rd party library, check `package.json`. If missing, output the install command before providing code.

### Framework Agnostic
Work with whatever the project uses. If no framework specified:
- Default to clean semantic HTML + CSS
- Use CSS Grid for layout, CSS custom properties for theming
- Vanilla JS for interactivity

### When Using React/Next.js
- Default to Server Components (RSC). Wrap providers in `"use client"` components.
- Extract interactive components as isolated `'use client'` leaf components.
- Use local `useState`/`useReducer` for isolated UI. Global state for deep prop-drilling avoidance only.

### Styling
- Work with whatever styling the project uses (Tailwind, CSS Modules, styled-components, vanilla CSS)
- If Tailwind, check version in `package.json` first — don't use v4 syntax in v3 projects

### Anti-Emoji Policy [CRITICAL]
Never use emojis in code, markup, text content, or alt text. Replace with high-quality icons (Phosphor, Radix) or clean SVG primitives.

### Performance Guardrails
- Apply grain/noise filters exclusively to fixed, pointer-events-none pseudo-elements
- Never animate `top`, `left`, `width`, or `height` — use `transform` and `opacity`
- Use z-indexes strictly for systemic layer contexts (navbars, modals, overlays)

---

## 10. COLOR PALETTES

→ *Full reference at [color-palettes reference](reference/color-palettes.md).*

Select randomly when drawing inspiration. Do not always pick the first palette.

### Dark Schemes

| Palette | Background | Surface | Primary | Secondary | Accent | Text |
|--------|-----------|--------|--------|----------|--------|------|
| Midnight Canvas | `#0a0e27` | `#151b3d` | `#6c8eff` | `#a78bfa` | `#f472b6` | `#e2e8f0` |
| Obsidian Depth | `#0f0f0f` | `#1a1a1a` | `#00d4aa` | `#00a3cc` | `#ff6b9d` | `#f5f5f5` |
| Slate Noir | `#0f172a` | `#1e293b` | `#38bdf8` | `#818cf8` | `#fb923c` | `#f1f5f9` |
| Carbon Elegance | `#121212` | `#1e1e1e` | `#bb86fc` | `#03dac6` | `#cf6679` | `#e1e1e1` |
| Charcoal Studio | `#1c1c1e` | `#2c2c2e` | `#0a84ff` | `#5e5ce6` | `#ff375f` | `#f2f2f7` |
| Void Space | `#0d1117` | `#161b22` | `#58a6ff` | `#79c0ff` | `#f78166` | `#c9d1d9` |
| Twilight Mist | `#1a1625` | `#2d2438` | `#9d7cd8` | `#7aa2f7` | `#ff9e64` | `#dcd7e8` |
| Onyx Matrix | `#0e0e10` | `#1c1c21` | `#00ff9f` | `#00e0ff` | `#ff0080` | `#f0f0f0` |

### Light Schemes

| Palette | Background | Surface | Primary | Secondary | Accent | Text |
|--------|-----------|--------|--------|----------|--------|------|
| Cloud Canvas | `#fafafa` | `#ffffff` | `#2563eb` | `#7c3aed` | `#dc2626` | `#0f172a` |
| Pearl Minimal | `#f8f9fa` | `#ffffff` | `#0066cc` | `#6610f2` | `#ff6b35` | `#212529` |
| Ivory Studio | `#f5f5f4` | `#fafaf9` | `#0891b2` | `#06b6d4` | `#f59e0b` | `#1c1917` |
| Linen Soft | `#fef7f0` | `#fffbf5` | `#d97706` | `#ea580c` | `#0284c7` | `#292524` |
| Arctic Breeze | `#f0f9ff` | `#f8fafc` | `#0284c7` | `#0ea5e9` | `#f43f5e` | `#0c4a6e` |
| Sand Warm | `#faf8f5` | `#ffffff` | `#b45309` | `#d97706` | `#059669` | `#451a03` |

---

## 11. REFERENCE FILES

For deep-dive guidance, consult these reference files:

| File | Covers |
|------|--------|
| [typography](reference/typography.md) | Type systems, font pairing, modular scales, OpenType |
| [color-and-contrast](reference/color-and-contrast.md) | OKLCH, tinted neutrals, dark mode, accessibility |
| [spatial-design](reference/spatial-design.md) | Spacing systems, grids, visual hierarchy |
| [motion-design](reference/motion-design.md) | Easing curves, staggering, reduced motion |
| [interaction-design](reference/interaction-design.md) | Forms, focus states, loading patterns |
| [responsive-design](reference/responsive-design.md) | Mobile-first, fluid design, container queries |
| [ux-writing](reference/ux-writing.md) | Button labels, error messages, empty states |
| [anti-patterns](reference/anti-patterns.md) | Consolidated AI slop ban list |
| [color-palettes](reference/color-palettes.md) | Curated dark/light color schemes |
| [creative-arsenal](reference/creative-arsenal.md) | Advanced design concepts |

---

## 12. LAYOUT GENERATION PATTERNS

Positive guidance: what TO generate (not just what to avoid). Choose based on DESIGN_VARIANCE.

### Layout Archetypes by Variance

**VARIANCE 1-3 (Clean):** Centered container, standard stacking, simple grids
```css
.page { max-width: 1200px; margin: 0 auto; padding: 0 clamp(1rem, 5vw, 3rem); }
.grid { display: grid; grid-template-columns: repeat(auto-fit, minmax(280px, 1fr)); gap: 1.5rem; }
```

**VARIANCE 4-7 (Dynamic):** Split-screen hero, asymmetric grids, varied column widths
```css
/* Split-screen hero */
.hero { display: grid; grid-template-columns: 1.2fr 0.8fr; min-height: 80dvh; align-items: center; }
/* Asymmetric feature grid */
.features { display: grid; grid-template-columns: 2fr 1fr 1fr; gap: 1.5rem; }
.features > :first-child { grid-row: span 2; }
```

**VARIANCE 8-10 (Experimental):** Bento grid, masonry, overlapping, offset margins
```css
/* Bento grid */
.bento { display: grid; grid-template-columns: repeat(4, 1fr); grid-auto-rows: 180px; gap: 1rem; }
.bento .wide { grid-column: span 2; }
.bento .tall { grid-row: span 2; }
.bento .featured { grid-column: span 2; grid-row: span 2; }
/* Offset margin editorial */
.editorial-block { margin-left: 15%; max-width: 70%; }
.editorial-block:nth-child(even) { margin-left: 5%; margin-right: 15%; }
```

### Responsive Collapse Strategy

All high-variance layouts MUST collapse gracefully:
```css
@media (max-width: 768px) {
  .hero { grid-template-columns: 1fr; }           /* Stack vertically */
  .features { grid-template-columns: 1fr; }        /* Single column */
  .bento { grid-template-columns: repeat(2, 1fr); } /* 2-col mobile */
  .editorial-block { margin-left: 0; max-width: 100%; }
}
```

### Container Query Patterns

Use container queries for component-level responsiveness:
```css
.card-container { container-type: inline-size; }
@container (min-width: 400px) { .card { flex-direction: row; } }
@container (min-width: 600px) { .card { grid-template-columns: 1fr 2fr; } }
```

### Common Page Structures

| Page Type | Recommended Layout | Grid |
|-----------|-------------------|------|
| Landing | Split-screen hero → bento features → testimonial slider | `1.2fr 0.8fr` → `2fr 1fr 1fr` |
| Dashboard | Sidebar (220px) + main content with card grid | `220px 1fr` → `auto-fit minmax(300px, 1fr)` |
| Blog/Content | Left-aligned prose (65ch) + sticky sidebar | `minmax(0, 65ch) 280px` |
| Settings | Tab nav + stacked form sections | Single column, max-width 640px |
| Pricing | Asymmetric card row (small, featured, small) | `1fr 1.3fr 1fr` |

---

## 13. FULL-STACK INTEGRATION

When backend, API, or database layers are detected, check these integration points:

### DTO & Type Safety
- Frontend types/interfaces must match backend DTOs exactly
- Shared types in a `shared/` or `types/` package are preferred
- Zod schemas, tRPC routers, or OpenAPI specs should be the single source of truth
- Never duplicate type definitions across frontend and backend — use code generation or shared packages

### API Contract Consistency
- Every API call must handle: loading state, success, error, empty data
- Error responses must be typed — never `catch(e: any)`
- Pagination, sorting, and filtering params must match between frontend query and backend handler
- API URLs should come from environment config, never hardcoded

### Database Schema Alignment
- Frontend form validation must reflect database constraints (required fields, max lengths, enums)
- Nullable database columns must be handled as `| null` in frontend types
- Enum values in the database must match dropdown/select options in the UI
- Date formats must be consistent across the stack (ISO 8601 preferred)

### Error Handling Chain
- Backend validation errors must surface as inline field errors, not generic toasts
- Network errors (timeout, 5xx) must show retry-capable error states
- Optimistic updates must have rollback logic
- Auth errors (401/403) must redirect to login, not show a broken page

### Environment & Config
- All API URLs, feature flags, and secrets must come from environment variables
- Frontend must never expose backend secrets or internal API keys
- Feature flags should be consistent — don't show UI for disabled backend features

---

## 14. THE AI SLOP TEST

**Final quality check.** If you showed this interface to someone and said "AI made this," would they believe you immediately? If yes, that's the problem.

A distinctive interface should make someone ask "how was this made?" — not "which AI made this?"

Review the anti-pattern catalog. Those are the fingerprints of AI-generated work from 2024-2025.

---

## 15. PRE-FLIGHT CHECKLIST

Before outputting code, evaluate against this matrix:
- [ ] Does the output pass the AI Slop Test?
- [ ] Is typography intentional (not Inter/system defaults)?
- [ ] Is the color palette cohesive and non-generic?
- [ ] Are hover, focus, active, loading, empty, and error states provided?
- [ ] Is layout asymmetric where appropriate?
- [ ] Is mobile layout collapse guaranteed for high-variance designs?
- [ ] Do full-height sections safely use `min-h-[100dvh]`?
- [ ] Do animations use `transform`/`opacity` only?
- [ ] Are cards used only when elevation communicates hierarchy?
- [ ] Is global state used appropriately?
- [ ] Are `useEffect` animations cleaned up properly?
- [ ] Is all code complete with no banned placeholder patterns?
- [ ] Did you check `package.json` before importing new libraries?
- [ ] Do frontend types match backend DTOs?
- [ ] Are API errors handled with proper UI states?
- [ ] Do form validations reflect database constraints?
- [ ] Are environment variables used for all config?

---
> Source: [OJamals/UIdetox](https://github.com/OJamals/UIdetox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
