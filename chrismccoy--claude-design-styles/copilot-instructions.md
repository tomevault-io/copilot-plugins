## claude-design-styles

> When building or styling a frontend, I'll specify a style by name. Apply the corresponding spec below exactly. If no style is specified, ask which one to use.

# Design Styles

When building or styling a frontend, I'll specify a style by name. Apply the corresponding spec below exactly. If no style is specified, ask which one to use.

**Available styles:**
- `bento style` — Apple/macOS-inspired bento grid, clean and minimal
- `clay style` — Clay morphism, chunky rounded cards with physical depth
- `pure brutalist style` — Monochrome black/white, hard shadows, monospace, no color
- `neobrutalist style` — Hard black shadows with vivid neon color accents
- `pop art style` — Cyan/pink/yellow on loud background, floating bordered container
- `soft modern style` — White bg, blurred orb accents, rounded, friendly and accessible
- `dark cosmic style` — Dark slate, glowing indigo/cyan, radial dot grid, glassmorphism
- `dark action style` — Dark gradient bg, yellow/gold accents, Oswald font, cinematic energy
- `dark saas style` — Slate-950, sky blue accent, stagger animations, clean SaaS
- `acid brutalist style` — Pure black, acid yellow + red, Anton/Bebas fonts, noise grain
- `enterprise editorial style` — White/dark alternating sections, indigo, large rounded app cards
- `utility terminal style` — White bg, strict 1px borders, monospace, no rounding, grid texture
- `dark cinema style` — Near-black, red glow, Bebas Neue, noise overlay, floating labels
- `dark mono style` — Dark zinc surfaces, cyan + pink accents, monospace, scanline texture
- `blueprint style` — Deep blueprint blue, white grid lines, Courier Prime, technical drawing aesthetic
- `monolith style` — White bg, dark navy shadows, thick top border accent, monospace brutalism
- `dot grid style` — Gray dotted background, Archivo Black + Space Mono, hot pink accent, hard shadows
- `pink neo style` — Hot pink dotted background, Archivo Black + Space Mono, pink/yellow/blue palette
- `glassmorphism style` — Frosted glass cards on gradient mesh backgrounds, soft blurs and translucency
- `newspaper style` — Black ink on newsprint, serif fonts, editorial column layouts
- `retro terminal style` — Green-on-black CRT monitor aesthetic with phosphor glow effects
- `memphis style` — 80s/90s geometric shapes, bright pastels, squiggles and confetti
- `luxury style` — Cream/off-white, serif display font, gold accents, generous whitespace
- `skeuomorphic style` — Realistic material textures, depth and shadows mimicking physical objects
- `vaporwave style` — Purple/teal gradients, retro grid floors, synthwave glow and glitch effects
- `swiss style` — Helvetica-inspired, rigid typographic grid, black/red only, zero decoration
- `dark neon style` — Black background, multiple vivid neon glow colors, bleed and bloom effects
- `organic style` — Earthy tones, rounded organic shapes, warm and natural hand-crafted feel
- `neumorphism style` — Soft same-color shadows creating pushed/extruded soft UI on light gray
- `cyberpunk style` — Yellow/black warning stripes, HUD overlays, neon on dark, danger aesthetics
- `art deco style` — Geometric gold ornaments, symmetry, 1920s glamour and opulence
- `isometric style` — 3D isometric grid illustrations, flat-color depth and layered objects
- `groovy style` — Warm oranges/browns, 70s swirls, rounded retro lettering, psychedelic curves
- `zine style` — Photocopied DIY aesthetic, cut-and-paste collage, raw indie energy
- `sci-fi hud style` — Heads-up display, corner brackets, data readouts, radar and targeting UI
- `pixel style` — 8-bit pixelated fonts, game UI, sprite aesthetic, retro game feel
- `scandinavian style` — Cold whites, extreme negative space, hygge minimalism, quiet luxury
- `gothic style` — Dark greens/blacks, ornate serif, candle-wax drips, moody atmosphere
- `handwritten style` — Hand-drawn borders, pencil textures, imperfect sketch-like lines
- `aurora style` — Flowing multi-color gradient backgrounds, silk light effect, soft and dreamy
- `tropical style` — Coral, turquoise, warm vacation energy, Miami/resort vibes
- `grunge style` — Worn textures, splatter marks, distressed rough torn edges
- `y2k style` — Windows 95 beveled gray UI, system fonts, chunky pixel buttons, early internet
- `kawaii style` — Super cute pastel, bubble rounded, character illustration accents
- `manga style` — Speed lines, bold ink outlines, dramatic panel layouts, high contrast
- `dashboard style` — Chart-forward, dense metrics, sidebar navigation, admin/analytics feel
- `maximalist style` — Everything layered, dense pattern-on-pattern, opulent visual chaos
- `corporate style` — Conservative trust blues, structured grid, buttoned-up B2B professionalism
- `psychedelic style` — Acid swirls, melting text, rainbow overflow, mind-bending distortion
- `athletic style` — Diagonal cuts, bold color blocks, high-impact sport energy
- `cottagecore style` — Floral patterns, watercolor washes, storybook softness and whimsy
- `japanese style` — Wabi-sabi imperfection, ink brush strokes, kanji-inspired negative space
- `longform style` — Full-bleed hero images, pull quotes, drop caps, rich magazine editorial flow

---

## Bento Style

Apple/macOS-inspired design system — clean, minimal, trust-building. Use when asked for "bento style".

### Typography
Plus Jakarta Sans (Google Fonts, weights 400–800). Headlines 800 weight, -0.02em letter-spacing, `clamp(28px, 4vw, 44px)`. Section labels 13px, 700 weight, 0.08em letter-spacing, uppercase. Body 16px/1.6. Apply `-webkit-font-smoothing: antialiased`.

### Colors
```css
--bg:        #f5f5f7;
--card:      #ffffff;
--text:      #1d1d1f;
--accent:    #0071e3;
--accent-dk: #0058c5;
--green:     #34c759;
--red:       #ff375f;
--muted:     #6e6e73;
--border:    rgba(0,0,0,0.07);
--shadow-sm: 0 2px 8px rgba(0,0,0,0.06), 0 8px 24px rgba(0,0,0,0.04);
--shadow-md: 0 4px 16px rgba(0,0,0,0.08), 0 16px 40px rgba(0,0,0,0.07);
--radius:    20px;
--radius-lg: 24px;
```

### Components

**.card** — white, 20px radius, `--shadow-sm`, hover `translateY(-2px)` + `--shadow-md`

**.pill variants:**
- `.pill-blue` → `rgba(0,113,227,0.1)` bg, blue text
- `.pill-green` → `rgba(52,199,89,0.12)` bg, dark green text
- `.pill-red` → `rgba(255,55,95,0.1)` bg, red text
- `.pill-gray` → `rgba(0,0,0,0.06)` bg, muted text

**.btn-blue** — `#0071e3` bg, white text, 100px radius, `0 4px 14px rgba(0,113,227,0.35)` shadow, hover darkens + lifts

**.btn-white** — white bg, dark text, `0 4px 14px rgba(0,0,0,0.15)` shadow

**.section-label** — blue uppercase micro-label above headings

### Layout Patterns

**Stats bento** — 5-column responsive grid of stat cards (big number + label)

**Features bento** — 3-column grid; cards can span 2 columns (`.wide`) for emphasis. Each card: icon box + headline + body.

**Widgets bento** — 4-column grid with `.tall` (spans 2 rows) and `.wide` (spans 2 cols) variants

**Design/archive bento** — asymmetric 2-column: large showcase card left, smaller detail cards stacked right

**Customizer bento** — featured card (`.featured`, blue gradient bg) alongside smaller option cards

**Two-column grid** — 50/50 split for comparison or two-panel sections

**Steps grid** — 3-column equal grid for numbered steps

### Header
White/translucent (`rgba(255,255,255,0.85)`), backdrop-filter blur 20px, 1px bottom border, sticky. Product name left + nav pills + CTA right. Nav pills: `rgba(0,0,0,0.06)` bg, hover `rgba(0,113,227,0.1)`.

### Purchase CTA
Full-width blue gradient: `linear-gradient(135deg, #0071e3 0%, #0058c5 100%)`. White text, `.btn-white` + outlined white button. Decorative circles in corners.

### Pricing Card
Centered, max 440px. White card, blue top accent bar. Feature list with blue checkmark bullets. Blue CTA button. One-time price badge.

### Marquee Strip
`overflow: hidden`, scrolling track with `animation: marquee 30s linear infinite`. White bg, blue icon accents, subtle top/bottom borders.

### Footer
Dark `#1d1d1f` bg, white text, 3-column layout (brand | links | social). `#3a3a3c` border on bottom bar with copyright.

### Implementation
- Single HTML file, inline CSS, CDN Tailwind, Font Awesome 6, Google Fonts
- `.container` max-width 1120px, margin auto, 24px padding
- Sections: `padding: 80px 0`
- Smooth scroll on `<html>`
- No section color alternation — entire page stays `#f5f5f7`, cards provide contrast
- Grid gaps 16px–24px consistently
- Headings use `clamp()` for responsive scaling

---

## Clay Style

Playful, chunky, toy-like design system — bold shadows, rounded forms, candy colors, physical depth. Use when asked for "clay style".

### Typography
Nunito (Google Fonts, weights 400–900). Headlines 900 weight. Buttons 800 weight. Body 700 weight. Apply `-webkit-font-smoothing: antialiased`.

### Colors
```
Page background:  #f0eeff   (soft lavender)
Primary text:     #1e1b4b   (deep indigo)
Primary brand:    #7c3aed   (vivid purple)
Accent:           #f59e0b   (amber/gold)
Dark purple:      #5b21b6   (shadow color)
Footer bg:        #1e1b4b
```

### The Clay Card — Core Mechanic
Every card uses a 3-layer box-shadow for physical depth:
```css
.clay-card {
    background: #ffffff;
    border-radius: 28px;
    box-shadow:
        8px 8px 0px rgba(124,58,237,0.15),
        inset 0 -4px 0 rgba(0,0,0,0.08),
        0 20px 40px rgba(124,58,237,0.10);
    transition: transform 0.2s ease, box-shadow 0.2s ease;
}
.clay-card:hover {
    transform: translateY(-3px);
    box-shadow:
        10px 12px 0px rgba(124,58,237,0.18),
        inset 0 -4px 0 rgba(0,0,0,0.08),
        0 28px 48px rgba(124,58,237,0.14);
}
```

**.clay-card-purple** — `#7c3aed` bg, white text:
```css
box-shadow:
    8px 8px 0px rgba(91,33,182,0.4),
    inset 0 -4px 0 rgba(0,0,0,0.18),
    0 20px 40px rgba(124,58,237,0.25);
```

### Clay Buttons — Pressable Feel
Bottom-offset shadow simulates a physical press. Lift on hover, press down on `:active`.
```css
.clay-btn-purple {
    background: #7c3aed; color: #fff;
    border-radius: 100px; font-weight: 800;
    box-shadow:
        0 6px 0px #5b21b6,
        inset 0 -6px 0 rgba(0,0,0,0.15),
        0 12px 32px rgba(124,58,237,0.35);
}
/* hover: translateY(-2px), shadows grow */
/* active: translateY(2px), shadows shrink */

.clay-btn-amber { background: #f59e0b; color: #1e1b4b; /* 0 6px 0px #d97706 */ }
.clay-btn-white { background: #fff; color: #1e1b4b; /* 0 6px 0px rgba(0,0,0,0.15) */ }
```

### Section Color Rotation
Alternate section backgrounds in this order — it's the primary visual rhythm of the page:
```
bg-section-mint:   #ecfdf5
bg-section-peach:  #fff7ed
bg-section-sky:    #eff6ff
bg-section-yellow: #fefce8
bg-section-green:  #f0fdf4
bg-section-pink:   #fdf2f8
bg-section-violet: #f5f3ff
bg-section-rose:   #fff1f2
```

### Components

**.nav-pill** — pastel-colored bg pills, 100px radius, 700 weight

**.icon-box** — 48×48px, 14px radius, brand-colored bg, centered icon

**.clay-pill** — inline label/tag, 100px radius, 800 weight, pastel bg

**.check-badge** — feature list item with `#7c3aed` circle + white checkmark

**.stat-card** — clay-card with large number (`clamp(36px,5vw,56px)`, 900 weight) + label

**.num-badge** — 28×28px circle, `#7c3aed` bg, white text, 900 weight

**.feature-item** — feature row; label text has `border-bottom: 3px solid #7c3aed`, `display: inline`

**.option-card** — clay-card with 4px colored top accent bar (`border-radius: 4px 4px 0 0`)

**.archive-card** — clay-card with colored header bar, title, description

**.swatch-chip** — 28×28px circle, `border-radius: 50%`, `box-shadow: 3px 3px 0px rgba(0,0,0,0.15)`

**.clay-code** — `background: #1e1b4b`, `color: #c4b5fd`, monospace, 16px radius

### Header
White bg, `box-shadow: 0 2px 20px rgba(124,58,237,0.08)`, sticky. Logo left, colored nav pills right, amber "Buy Now" CTA.

### Hero
`background: #f0eeff`. Giant headline `clamp(56px,10vw,96px)`, 900 weight. Two CTA buttons. Stat cards row below. Optional purple marquee bar (`background: #7c3aed`, white scrolling text).

### Pricing Card
Centered, max 480px. Clay-card with gradient top stripe `linear-gradient(135deg, #7c3aed, #a855f7)`. Price in amber. `.check-badge` feature list. `.clay-btn-purple` CTA.

### Purchase CTA
Dark purple gradient: `linear-gradient(135deg, #4c1d95 0%, #7c3aed 50%, #6d28d9 100%)`. White headline, amber price, `.clay-btn-amber` primary + `.clay-btn-white` secondary. Blurred decorative circles as bg accents.

### Marquee Strip
`background: #7c3aed`, white text, `overflow: hidden`. Track: `display: flex; gap: 40px; animation: marquee 36s linear infinite`. Items separated by `•`.

### Footer
`background: #1e1b4b`. White text, `#c4b5fd` accent links. 3-column: brand | links | social. Purple border on bottom bar.

### Implementation
- Single HTML file, inline CSS, CDN Tailwind, Font Awesome 6, Google Fonts (Nunito)
- `.container` max-width 1120px, margin auto, 24px padding
- Sections: `padding: 80px 0`
- `border-radius: 28px` on all major cards
- Card grids: 2–4 columns, `gap: 24px–28px`
- ALL shadows use purple hue family `rgba(124,58,237,...)` for cohesion
- Buttons MUST have the `6px` bottom-offset shadow for the pressed-clay feel
- Font weights: 900 for stats/numbers, 800 for buttons/badges, 700 for nav/labels
- Use at least 4–5 different section background colors per page
- Smooth scroll on `<html>`

---

## Pure Brutalist Style

Strict black-and-white brutalism — no color, no rounding, no softness. Raw, confrontational, structural. Use when asked for "pure brutalist style".

### Typography
System monospace stack: `ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, "Courier New", monospace`. All text. Headlines uppercase, tracking-widest. Labels in `text-xs font-bold uppercase tracking-widest`.

### Colors
```
Background:  #ffffff
Text:        #000000
Border:      #000000
Shadow:      #000000
Muted text:  rgba(0,0,0,0.72)
```
No color accents. Pure black and white only.

### Shadow System
Three sizes, all hard black offset shadows — no blur:
```css
--shadow-lg: 8px 8px 0 0 #000;
--shadow-sm: 5px 5px 0 0 #000;
--shadow-xs: 3px 3px 0 0 #000;
```
Cards: `border: 2px solid #000` + `--shadow-lg`
Buttons: `border: 2px solid #000` + `--shadow-xs`, hover `translate(0.5px, 0.5px)` + shadow shrinks

### Cards
```css
.card { border: 2px solid #000; box-shadow: 8px 8px 0 0 #000; background: #fff; }
```
No border-radius (or minimal: 0px). Section panels use `border-4 border-black`. Headers use `border-b-4 border-black`.

### Buttons
```css
.btn { border: 2px solid #000; background: #fff; font-weight: 900; uppercase; box-shadow: 3px 3px 0 0 #000; }
.btn-primary { background: #000; color: #fff; }
/* hover: translate(0.5px, 0.5px), shadow none */
/* active: translate(2px, 2px), shadow none */
```

### Layout & Details
- Sticky header: `border-b-4 border-black bg-white`
- Small utility bar above nav: `border-b-4 border-black bg-white text-xs uppercase tracking-widest` showing version/status
- Grids use `gap-px bg-black` to create 1px black grid lines between cells
- Section dividers: `border-b-4 border-black`
- Hero: 2-column grid (8-col left content, 4-col right terminal/visual), min-h-600px
- Feature grids: items divided by `border-r border-black`, hover `bg-neutral-50`
- Stats: `text-4xl font-light` numbers + `text-sm font-bold uppercase tracking-widest` labels
- Toasts/modals: `border-4 border-black shadow-brutal bg-white`, icon areas `bg-black text-white`
- Background: plain `#ffffff` — no textures, no gradients

### Footer
`bg-neutral-100`, `border-t border-black`. 4-column grid divided by `divide-x divide-black border border-black`. Links prefixed with `>`. Bottom bar `bg-white border-t border-black`, 10px copyright text.

### Implementation
- Tailwind CDN only — no custom fonts needed (use system monospace)
- No `border-radius` on structural elements
- Everything uppercase for labels and nav
- `selection:bg-black selection:text-white`
- `antialiased` on body

---

## Neobrutalist Style

Hard-edged brutalism with vivid neon color accents. CSS variable-driven with theming support. Use when asked for "neobrutalist style".

### Typography
Space Grotesk (display/body) + Space Mono (code/mono elements). Or substitute any bold sans + monospace pairing.

### Colors
```css
--neo-bg:     #f3f4f6;   /* light gray page background */
--neo-black:  #0a0a0a;   /* borders, shadows, text */
/* Accent palette — all used simultaneously: */
--neo-purple: #a855f7;
--neo-green:  #a3e635;
--neo-pink:   #fb7185;
--neo-yellow: #facc15;
--neo-blue:   #60a5fa;
```
Assign different accents to different sections/features. Use all of them.

### Shadow System
Pure black offset shadows — no blur:
```css
--shadow-neo:    5px 5px 0px 0px rgba(0,0,0,1);
--shadow-neo-lg: 8px 8px 0px 0px rgba(0,0,0,1);
--shadow-neo-sm: 3px 3px 0px 0px rgba(0,0,0,1);
```

### Cards
```css
.neo-card {
  border: 2px solid #0a0a0a;
  box-shadow: 5px 5px 0px 0px #0a0a0a;
  background: #fff;
}
/* Hover: decorative backing div (absolute, bg-black, translate-x-2 translate-y-2) grows */
/* The card itself hovers above via -translate-x-1 -translate-y-1 */
```
Use the "backing shape" pattern: sibling `div.absolute.inset-0.bg-black.translate-x-2.translate-y-2`, hovered to `translate-x-4 translate-y-4`.

### Buttons
```css
.neo-btn {
  border: 2px solid #0a0a0a;
  box-shadow: 5px 5px 0px 0px #0a0a0a;
  font-weight: 700; uppercase;
}
/* hover: translate-x-[2px] translate-y-[2px] + shadow shrinks to 3px */
/* active: translate-x-[5px] translate-y-[5px] + shadow: none */
```
Primary buttons use a neon accent bg (e.g. `--neo-green`). Secondary use `--neo-pink` or white.

### Background
Dot grid: `bg-[radial-gradient(#0000001a_1px,transparent_1px)] [background-size:16px_16px]` or line grid: `bg-[linear-gradient(to_right,rgba(0,0,0,.10)_1px,transparent_1px),linear-gradient(to_bottom,rgba(0,0,0,.10)_1px,transparent_1px)] [background-size:24px_24px]`. Decorative oversized `nb-card` shapes rotated in corners as background accents.

### Marquee
Yellow or accent bg strip, `border-b-4 border-black`, monospace font, text like `NO LOGS • NO TRACKING • SELF-DESTRUCT •`

### Header
`border-b-4 border-black bg-white`. Logo: product name in bold uppercase. Nav links: bold uppercase, `hover:underline decoration-4 decoration-[accent-color] underline-offset-4`. CTA: accent-bg button with neo shadow.

### Version/Tag Badge
Small angled badge: `inline-block bg-black text-white px-4 py-1 font-mono text-sm font-bold -rotate-1` near headline.

### Implementation
- `selection:bg-neo-black selection:text-neo-green`
- All buttons must have the `hover:translate + shadow-shrink` mechanic
- Big step numbers: `text-6xl font-bold opacity-50 absolute font-mono` watermark in card corner
- Headings: all-caps, massive, tight tracking (`tracking-tight leading-[0.9]`)

---

## Pop Art Style

Vivid cyan/pink/yellow on a loud background, housed in a floating bordered container. Energetic and physical. Use when asked for "pop art style".

### Typography
IBM Plex Mono (headlines/labels) + Inter (body). Bold everywhere. Headlines `font-black uppercase tracking-tighter italic`.

### Colors
```
Page bg:      #5CE1E6   (cyan — the whole page is this color)
Container bg: #ffffff
pop-cyan:     #5CE1E6
pop-yellow:   #FFDE59
pop-pink:     #FF90E8
pop-black:    #000000
```

### Container Layout
The entire design lives inside a max-width floating box with thick borders:
```css
.container-main {
  max-width: 1280px; margin: 32px auto;
  background: white;
  border: 4px solid #000;            /* or border-x-4 */
  box-shadow: 10px 10px 0 0 #000;
  min-height: 100vh;
}
```

### Cards & Buttons
**Cards** use the "backing shadow" pattern: `relative group` wrapper with `absolute inset-0 bg-black translate-x-2 translate-y-2 group-hover:translate-x-3 group-hover:translate-y-3` behind `relative bg-white border-4 border-black`.

**Buttons**:
```css
.btn-primary { background: #FF90E8; border: 4px solid #000; font-weight: 900; uppercase; }
.btn-secondary { background: #fff; border: 4px solid #000; }
/* Both use backing shadow pattern */
/* hover: -translate-x-1 -translate-y-1 */
/* active: translate-x-1 translate-y-1 */
```

### Decorative Details
- Angled "sticker" labels: `border-4 border-black px-4 py-2 font-black uppercase shadow-hard-xl transform rotate-6` (absolute positioned near hero visual)
- Inline tags: `bg-gray-100 border-2 border-black px-2 py-1 text-xs font-bold uppercase` with a small colored square indicator
- Section headers: `inline-block` badge with accent bg + border + hard shadow

### Header
`border-b-4 border-black bg-white sticky top-0`. Logo uses backing shadow pattern. Nav links: `hover:bg-[accent] px-2 py-1 border-2 border-transparent hover:border-black`. CTA: backing shadow button.

### Marquee
`bg-black text-white border-b-4 border-black`, monospace font, items with `✦` separators or brackets `[ITEM]`.

### Feature Cards
3-column grid of `border-2 border-black bg-white` cards. Hover: `-translate-x-1 -translate-y-1 shadow-hard`. Icon emoji top-left, yellow highlight on h3 hover.

---

## Soft Modern Style

Clean white design with soft blurred orb backgrounds, generous rounding, and accessible color palette. Trustworthy and friendly. Use when asked for "soft modern style".

### Typography
Default Tailwind font stack (system-ui). Headlines `font-extrabold tracking-tight`. Labels `font-semibold`. Body `text-slate-600 leading-relaxed`.

### Colors
```
Background:   #ffffff
Text:         #0f172a (slate-900)
Muted:        #64748b (slate-500)
Primary:      #2563EB (blue-600)
Accent:       #EC4899 (pink-500)
Border:       #e2e8f0 (slate-200)
Shadow:       0 10px 30px rgba(2,6,23,0.10)
```

### Background Orbs
Place large blurred circles for depth. No hard shapes:
```html
<div class="absolute -left-24 -top-24 size-80 rounded-full bg-blue-500/15 blur-3xl"></div>
<div class="absolute -right-24 top-24 size-80 rounded-full bg-pink-500/15 blur-3xl"></div>
```

### Cards
`rounded-2xl border border-slate-200 bg-white shadow-sm hover:shadow-md transition-shadow`. No hard borders or offset shadows. Subtle hover elevation only.

### Buttons
```css
.btn-primary { background: #EC4899; color: #fff; rounded-xl; padding: 12px 24px; font-weight: 600; shadow-soft; }
.btn-primary:hover { filter: brightness(1.1); }
.btn-secondary { border: 1px solid #e2e8f0; background: #fff; color: #0f172a; rounded-2xl; }
/* Always include focus-visible ring: focus-visible:ring-2 focus-visible:ring-[brand] focus-visible:ring-offset-2 */
```

### Header
`sticky top-0 border-b border-slate-200/70 bg-white/80 backdrop-blur`. Logo with rounded-xl icon. Plain text nav links. Pink rounded-xl CTA. Mobile: `<details>/<summary>` for menu, `rounded-2xl border border-slate-200 shadow-soft` dropdown.

### Small Trust Chips (below hero)
`rounded-2xl border border-slate-200 bg-white p-4 shadow-sm` — 3-column grid with bold heading + muted subtext.

### Feature List Items
`flex gap-3` with small `rounded-full` check circle in blue or pink, semibold text.

### Accessibility
- Skip link: `sr-only focus:not-sr-only focus:fixed focus:top-4 focus:left-4`
- All interactive elements: `focus-visible:outline-none focus-visible:ring-2`
- `aria-label` on icon buttons and logo links
- `role="menu"` on mobile dropdown

### Implementation
- No custom fonts needed — system font stack reads as clean and trusted
- Use `prefers-reduced-motion` media query: `* { scroll-behavior: auto !important; transition: none !important; animation: none !important; }`
- Max-width: `max-w-6xl` (1152px)
- Section padding: `py-14 md:py-20`

---

## Dark Cosmic Style

Deep dark background with glowing indigo/cyan accents, radial dot grid texture, and glassmorphism cards. Feels like SaaS infrastructure. Use when asked for "dark cosmic style".

### Typography
Default system-ui (Tailwind default). Headlines `font-bold tracking-tight`. Body `text-slate-200/300`. Very clean and readable against dark.

### Colors
```
Background:  #020617  (slate-950)
Surface:     #0f172a  (slate-900)
Border:      rgba(255,255,255,0.1)
Text:        #e2e8f0  (slate-200)
Muted:       #94a3b8  (slate-400)
Brand 500:   #6366f1  (indigo)
Brand 400:   #818cf8
Cyan accent: #06b6d4
Glow shadow: 0 0 0 1px rgba(99,102,241,.25), 0 20px 60px -20px rgba(99,102,241,.45)
```

### Background Layers
Three layers stacked `fixed inset-0 pointer-events-none`:
1. Large top blurred blob: `absolute -top-24 left-1/2 h-[32rem] w-[60rem] -translate-x-1/2 rounded-full bg-brand-600/25 blur-3xl`
2. Bottom-right cyan blob: `absolute -bottom-24 right-[-10rem] h-[28rem] w-[48rem] rounded-full bg-cyan-500/10 blur-3xl`
3. Dot grid: `absolute inset-0 bg-[radial-gradient(circle_at_1px_1px,rgba(148,163,184,.18)_1px,transparent_0)] [background-size:22px_22px] opacity-40`

### Cards
```css
.glass-card {
  background: rgba(15,23,42,0.6);   /* slate-900/60 */
  border-radius: 16px;
  border: 1px solid rgba(255,255,255,0.1);
  backdrop-filter: blur(12px);
  box-shadow: 0 0 0 1px rgba(99,102,241,.25), 0 20px 60px -20px rgba(99,102,241,.45);
}
```
Feature icon containers: `rounded-xl bg-brand-600/20 ring-1 ring-brand-500/30` with glow shadow.

### Buttons
```css
.btn-primary { background: #fff; color: #0f172a; rounded-xl; font-semibold; }
.btn-primary:hover { background: #f1f5f9; }
/* White on dark bg reads as high-contrast and premium */
.btn-ghost { rounded-xl; px-3 py-2; text-slate-200; hover:text-white; }
```

### Header
`mx-auto flex max-w-7xl items-center justify-between px-6 py-5`. No sticky panel — floats on the dark bg. Logo: icon in `rounded-xl bg-brand-600/20 ring-1 ring-brand-500/30`. Right: ghost "Sign in" + white "Start free" button.

Mobile menu: `rounded-2xl bg-slate-900/60 ring-1 ring-white/10 backdrop-blur p-4` dropdown.

### Glow Chip (hero badge)
```html
<span class="grid h-10 w-10 place-items-center rounded-xl bg-brand-600/20 ring-1 ring-brand-500/30 shadow-glow">
  [icon]
</span>
```

### Feature Icons
Always use the glowing icon box pattern. Never plain colored circles.

### Implementation
- No Google Fonts needed — system font works perfectly on dark
- All text in `slate-200` or `slate-300` for body, `white` for headings
- Hover states: `hover:text-white` on muted links
- Focus rings: `focus:ring-2 focus:ring-brand-400 focus:ring-offset-2 focus:ring-offset-slate-950`
- Section max-width: `max-w-7xl`

---

## Dark Action Style

Cinematic dark aesthetic with high-contrast yellow/gold accents, Oswald display font, and a tactile martial energy. Confident and aggressive. Use when asked for "dark action style".

### Typography
Oswald (display/headlines, weight 500–700) + Inter (body, weight 400–600). Headlines: `font-display font-bold uppercase tracking-tighter`. Nav labels: `font-sans font-semibold uppercase text-xs tracking-widest`.

### Colors
```
Background:  #111111  (cobra-black)
Dark surface: #1a1a1a (cobra-dark)
Light surface: #f3f4f6
Text on dark: #ffffff
Text on light: #111111
Yellow accent: #FCD34D  (cobra-yellow / amber-300)
Gold:          #D97706
Hero gradient: linear-gradient(135deg, #111111 0%, #2a2a2a 100%)
```

### Hero Section
Dark gradient full-width, `min-h-[85vh] flex items-center justify-center`. Background: large blurred circles at 10% opacity (`bg-cobra-yellow/10 rounded-full blur-[128px]`). Centered content with `font-display text-6xl md:text-8xl uppercase tracking-tighter leading-none`. Gradient text: `bg-clip-text text-transparent bg-gradient-to-b from-white to-gray-400`.

Small top badge: `rounded-full bg-white/10 backdrop-blur-sm border border-white/20 text-xs tracking-widest uppercase text-cobra-yellow`.

### Marquee Strip
`bg-cobra-yellow`, full width, `font-display font-bold text-lg text-cobra-black uppercase whitespace-nowrap tracking-widest`. Use icon separators (Font Awesome bolt). Below the hero, acts as visual separator.

### Cards (light sections)
`rounded-2xl border border-gray-100 bg-white p-8 shadow-sm hover:shadow-2xl hover:-translate-y-2 transition-all duration-300`. Icon areas: `rounded-2xl bg-gray-50 group-hover:bg-cobra-black group-hover:text-cobra-yellow transition-colors`.

### Buttons
```css
.btn-primary { background: #FCD34D; color: #111; rounded-lg; font-display font-bold uppercase tracking-wider; }
.btn-primary:hover { box-shadow: 0 0 30px rgba(252,211,77,0.4); transform: scale(1.05); }
/* Gold pulse-glow: @keyframes pulse-glow { 0%,100% { box-shadow: 0 0 15px rgba(252,211,77,0.2) } 50% { box-shadow: 0 0 25px rgba(252,211,77,0.5) } } */
```

### Navigation
`fixed w-full bg-white/90 backdrop-blur-md border-b border-gray-200 shadow-sm`. Logo: black icon `rounded-lg text-cobra-yellow`, hover turns yellow `bg-cobra-yellow text-black`. Nav buttons: yellow or black rounded-full pills (`px-6 py-2.5 rounded-full font-semibold uppercase text-xs tracking-widest`).

### Section Structure
Alternating light/dark: hero dark → content white → dark feature → white pricing. Each `py-24 px-4`. Yellow underline accent: `w-24 h-1 bg-cobra-yellow mx-auto rounded-full` after section titles.

### Decorative Ghost Text
`text-9xl font-display font-bold text-white outline-text opacity-20 rotate-12 pointer-events-none select-none` — watermark word in hero.

### Implementation
- Google Fonts: Oswald + Inter
- Font Awesome for icons (fa-hand-fist, fa-bolt, etc.)
- `antialiased` + `transition-colors duration-300` on body
- Custom scrollbar: `bg: #f1f1f1, thumb: #111, thumb:hover: #333, border-radius: 4px`

---

## Dark SaaS Style

Polished dark SaaS landing page. Slate-950 base, sky blue accent, staggered reveal animations, clean minimal layout. Use when asked for "dark saas style".

### Typography
System-ui / Tailwind default. Headlines `font-black tracking-tighter leading-none`. Subtext `text-slate-400 leading-relaxed`. Accent label `text-sky-400 font-semibold text-sm uppercase tracking-widest`.

### Colors
```
Background:  #020617  (slate-950)
Nav bg:      slate-950/80  (translucent)
Surface:     #0f172a  (slate-900)
Border:      #1e293b  (slate-800)
Text:        #f1f5f9  (slate-100)
Muted:       #94a3b8  (slate-400)
Brand:       #0ea5e9  (sky-500)
Brand hover: #38bdf8  (sky-400)
Green check: #4ade80  (green-400)
```

### Stagger Animations
```css
@keyframes fadeUp { "0%": { opacity: 0, transform: "translateY(24px)" }, "100%": { opacity: 1, transform: "translateY(0)" } }
.animate-fade-up { animation: fadeUp 0.6s ease-out forwards; }
.stagger-1 { animation-delay: 0.1s; opacity: 0; }
.stagger-2 { animation-delay: 0.2s; opacity: 0; }
/* ... up to stagger-6 */
```
Apply to hero headline, subtext, CTAs, trust chips in sequence.

### Background Glows
```html
<div class="absolute top-0 left-1/2 -translate-x-1/2 w-[800px] h-[500px] bg-sky-500/10 rounded-full blur-3xl"></div>
<div class="absolute top-20 left-1/2 -translate-x-1/2 w-[400px] h-[300px] bg-blue-600/10 rounded-full blur-2xl"></div>
```

### Cards
`bg-slate-900 border border-slate-800 rounded-2xl p-6`. Hover: `translateY(-4px)` + `box-shadow: 0 20px 40px -12px rgba(14,165,233,0.15)`. Feature icon: `w-10 h-10 rounded-lg bg-gradient-to-br from-sky-400 to-blue-600 flex items-center justify-center`.

### Trust Chips (below hero CTA)
`flex items-center gap-2 bg-slate-900 border border-slate-800 px-4 py-2 rounded-full text-sm`. Green check + `text-slate-300` description. Displayed in flex-wrap row.

### Buttons
```css
.btn-primary { background: #0ea5e9; color: #fff; rounded-xl; font-bold; shadow-lg shadow-sky-500/20; }
.btn-primary:hover { background: #38bdf8; }
.btn-secondary { border: 1px solid #334155; color: #cbd5e1; rounded-xl; }
.btn-secondary:hover { border-color: #475569; }
```

### Nav
`fixed bg-slate-950/80 backdrop-blur-md border-b border-slate-800`. Logo left, text links center (`text-sm text-slate-400 hover:text-white`), sky CTA right.

### Inline Code
`bg-slate-800 text-sky-400 px-1.5 py-0.5 rounded text-base font-mono` — for config filenames, commands.

### Accent Label Pattern
`text-sky-400 font-semibold text-sm uppercase tracking-widest mb-3` above every section heading. Consistent rhythm.

### Implementation
- No custom fonts needed
- `antialiased` on body
- Max-width: `max-w-6xl`
- Sections alternate: dark hero → feature grid → dark CTA → pricing
- `animate-pulse` on status dot in hero badge

---

## Acid Brutalist Style

Pure black background, acid yellow + brutal red accents, Anton/Bebas Neue display fonts, noise grain overlay. Loud, chaotic, punk energy. Use when asked for "acid brutalist style".

### Typography
Anton (huge display numbers/headlines) + Bebas Neue (section titles) + Space Mono (body/labels/code). All caps for display. `letter-spacing: 0.05em` on Space Mono.

### Colors
```
Background:   #0A0A0A  (near-black)
Text:         #F5F5F0  (off-white chalk)
Acid yellow:  #FFFF00
Brutal red:   #FF2D00
```

### Noise Grain Overlay
Fixed `::before` pseudo-element, full screen, `pointer-events: none; z-index: 9999`:
```css
body::before {
  content: '';
  position: fixed; inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='0.03'/%3E%3C/svg%3E");
  pointer-events: none; z-index: 9999; opacity: 0.4;
}
```

### Shadow System
Colored offset shadows (no blur):
```css
.brut-box     { box-shadow: 6px 6px 0px #FFFF00; }
.brut-box-red { box-shadow: 6px 6px 0px #FF2D00; }
.brut-box-white { box-shadow: 6px 6px 0px #F5F5F0; }
.brut-box-lg  { box-shadow: 10px 10px 0px #FFFF00; }
```

### Cards
`border: 3px solid #F5F5F0; background: #0A0A0A`. Hover: `border-color: #FFFF00; transform: translate(-4px, -4px); box-shadow: 4px 4px 0px #FFFF00`.

### Buttons
```css
.brut-btn {
  background: #FFFF00; color: #0A0A0A;
  font-family: 'Space Mono'; font-weight: 700; uppercase; letter-spacing: 0.05em;
  border: 3px solid #0A0A0A;
  box-shadow: 5px 5px 0px #0A0A0A;
}
.brut-btn:hover { transform: translate(3px, 3px); box-shadow: 2px 2px 0px #0A0A0A; }

.brut-btn-outline {
  background: transparent; color: #FFFF00;
  border: 3px solid #FFFF00; box-shadow: 5px 5px 0px #FFFF00;
}
.brut-btn-outline:hover { transform: translate(3px, 3px); box-shadow: 2px 2px 0px #FFFF00; }
```

### Decorative Patterns
```css
/* Diagonal stripe background */
.stripe-bg {
  background-image: repeating-linear-gradient(
    -45deg, #FFFF00 0px, #FFFF00 4px, transparent 4px, transparent 24px
  );
}
```
Use sparingly as section backgrounds or dividers.

### Code Block
```css
.code-block {
  background: #111; border: 2px solid #333; border-left: 4px solid #FFFF00;
  font-family: 'Space Mono'; color: #FFFF00; padding: 20px 24px; line-height: 1.8;
}
.code-block .comment { color: #555; }
.code-block .cmd { color: #F5F5F0; }
```

### Section Labels
`font-family: 'Space Mono'; font-size: 0.7rem; letter-spacing: 0.25em; uppercase; color: #FFFF00; border-left: 3px solid #FFFF00; padding-left: 10px;`

### Stack Pills
`border: 2px solid #FFFF00; color: #FFFF00; font-family: 'Space Mono'; text-transform: uppercase; letter-spacing: 0.08em`. Hover: `background: #FFFF00; color: #0A0A0A`.

### Watermark Numbers
`font-family: 'Anton'; font-size: clamp(5rem, 15vw, 12rem); color: #FFFF00; opacity: 0.12; position: absolute; top: -0.1em; left: -0.05em; pointer-events: none; user-select: none;` — behind section content.

### Scroll Reveal Animation
```css
.reveal { opacity: 0; transform: translateY(30px); transition: opacity 0.5s ease, transform 0.5s ease; }
.reveal.visible { opacity: 1; transform: none; }
```
Triggered via IntersectionObserver JS.

### Blink Cursor
`@keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }` `.blink { animation: blink 1s step-end infinite; }` — use on terminal prompts.

### Marquee
`background: #111; border-top: 2px solid #FFFF00; border-bottom: 2px solid #FFFF00`. Inner track: `animation: marquee 28s linear infinite`.

### Implementation
- Google Fonts: Anton + Bebas Neue + Space Mono
- `overflow-x: hidden` on body
- Custom scrollbar: `6px, track #0a0a0a, thumb #FFFF00`
- Red tags for warnings: `background: #FF2D00; color: #F5F5F0; font-family: Space Mono; font-size: 0.7rem; uppercase; padding: 3px 8px`

---

## Enterprise Editorial Style

White base with alternating dark sections, indigo accent, large rounded "app" cards, and a dense editorial layout with subtle grid texture. Feels like premium SaaS documentation. Use when asked for "enterprise editorial style".

### Typography
Inter (all weights 400–900). Headlines `font-black tracking-tighter leading-[0.85]`. Section labels `text-[10px] font-black uppercase tracking-[0.3em]`. Body `text-gray-500 leading-relaxed`. Very tight tracking on all headings.

### Colors
```
Background (light):  #ffffff
Background (dark):   #111827  (gray-900)
Background (darker): #030712  (gray-950)
Text:                #111827
Muted:               #6b7280  (gray-500)
Brand:               #4f46e5  (indigo-600)
Brand hover:         #4338ca  (indigo-700)
Brand light:         #e0e7ff  (indigo-100)
Shadow:              shadow-indigo-200 (colored shadows on light bg)
```

### Background Grid Texture
```css
.bg-grid {
  background-image: url("data:image/svg+xml,%3Csvg xmlns='http://www.w3.org/2000/svg' width='40' height='40' viewBox='0 0 40 40'%3E%3Cpath d='M40 0H0V40H40V0ZM39 1H1V39H39V1Z' fill='%236366f1' fill-opacity='0.05'/%3E%3C/svg%3E");
}
```
Use on the light hero section only.

### Section Alternation
Light → Dark → Light → Dark. Each section `py-32`. Dark sections: `bg-gray-900 text-white`.

### Cards
Light sections: `p-10 rounded-[3rem] bg-gray-50 border border-gray-100 hover:bg-white hover:shadow-2xl transition-all duration-500`. Icon: `w-14 h-14 bg-white rounded-2xl shadow-sm`, hover `scale-110 transition-transform`.

Dark sections (app mockup): `bg-gray-800 rounded-[3rem] p-4 shadow-2xl ring-1 ring-white/10` outer frame, `bg-gray-950 rounded-[2.5rem] p-8 md:p-12` inner screen. Mac traffic lights: small circles `bg-red-500/20 border border-red-500/40`.

### Feature Rows (dark sections)
`flex gap-6` with `shrink-0 w-14 h-14 bg-indigo-500/10 rounded-2xl flex items-center justify-center text-indigo-400 ring-1 ring-indigo-500/20 group-hover:bg-indigo-500 group-hover:text-white transition-all`.

### Buttons
```css
.btn-primary { background: #111827; color: #fff; rounded-2xl; px-12 py-6; font-black uppercase tracking-widest; shadow-2xl; }
.btn-primary:hover { background: #000; }
.btn-secondary { background: #fff; border: 2px solid #f3f4f6; rounded-2xl; px-12 py-6; font-black uppercase; }
.btn-secondary:hover { border-color: #4f46e5; }
/* Active: scale-95 */
```
Size: very large (`text-lg`, `py-6`) — editorial CTAs are prominent.

### Hero Badge
```html
<div class="inline-flex items-center gap-2 px-4 py-1.5 rounded-full bg-indigo-50 border border-indigo-100 mb-10 shadow-sm">
  <span class="relative flex h-2 w-2">
    <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-indigo-400 opacity-75"></span>
    <span class="relative inline-flex rounded-full h-2 w-2 bg-indigo-600"></span>
  </span>
  <span class="text-[10px] font-black uppercase tracking-widest text-indigo-600">Live status message</span>
</div>
```

### Social Proof Strip
Greyscale logos: `opacity-30 grayscale contrast-125`. Text labels in `text-2xl font-black italic`.

### Navigation
`sticky top-0 bg-white/90 backdrop-blur-md border-b border-gray-100`. Labels in `text-[10px] font-black uppercase tracking-[0.2em] text-gray-400`. CTA: `bg-indigo-600 text-white rounded-xl shadow-xl shadow-indigo-200`.

### Implementation
- Google Fonts: Inter only
- `selection:bg-indigo-100 selection:text-indigo-700`
- `antialiased`
- Alpine.js optional for interactive mockup elements
- Headings lead with `[0.85]` tight leading, increase to normal for body

---

## Utility Terminal Style

Pure white, strict 1px black borders everywhere, no border-radius, monospace font, grid background. Feels like a utility dashboard or command interface. Use when asked for "utility terminal style".

### Typography
System monospace: `font-mono` (Tailwind default). Everything is monospace. No other fonts. Sizes small: `text-xs` for labels, `text-sm` for body, `text-4xl font-light` for stat numbers. Labels: uppercase with `tracking-widest`.

### Colors
```
Background:    #ffffff
Surface alt:   #f5f5f5  (neutral-100)
Surface dark:  #171717  (neutral-900 — for nav logo)
Text:          #000000
Muted:         #737373  (neutral-500)
Border:        #000000  (1px solid)
Accent:        none (black only, occasionally neutral-100 hover bg)
```
No color accents. Strictly utilitarian.

### Border System
Everything uses `border border-black` (1px). No `border-2` or `border-4` — thin lines only:
- Outer container: `border-x border-black max-w-[1600px] mx-auto shadow-2xl`
- Header: `border-b border-black`
- Sections: `border-b border-black`
- Cards: separated by `gap-px bg-black` grid (1px black gaps between cells)
- Module tags: `text-[10px] border border-black px-1`

### No Border-Radius
All elements are strictly rectangular. `border-radius: 0` everywhere. No rounding.

### Nav Header
Two-level header:
1. Utility bar: `border-b border-black bg-neutral-100 px-2 py-1 text-[10px] uppercase tracking-widest flex justify-between` — shows `sys_status: online` and `v.2.0.4 build_9912`
2. Main nav: `sticky top-0 border-b border-black bg-white/95 backdrop-blur-sm h-16`. Logo block: `w-48 border-r border-black bg-black text-white` (separate column). Right action: `w-32 border-l border-black` with `[ ACCESS ]` link.

### Hero Layout
2-column: `grid-cols-12`. Left (8 cols): tag + headline + description + 2 buttons. Right (4 cols): `border-l border-black bg-scanlines` terminal output panel.
```css
.bg-scanlines {
  background: linear-gradient(to bottom, rgba(0,0,0,0) 50%, rgba(0,0,0,0.05) 50%);
  background-size: 100% 4px;
}
```

### Background Grid
```css
.bg-grid-sm {
  background-image: linear-gradient(#e5e5e5 1px, transparent 1px),
    linear-gradient(90deg, #e5e5e5 1px, transparent 1px);
  background-size: 20px 20px;
}
```
Used on the main content area.

### Terminal Panel
Right side of hero: `p-4 border-b border-black text-xs font-bold bg-white` header + `p-6 font-mono text-xs text-neutral-500` content with `>` prefixed lines. Bottom: progress bar `h-1 w-full bg-neutral-200` with `bg-black` fill.

### Cards (feature grid)
`gap-px bg-black` parent creates 1px black grid lines. Children: `bg-white p-6 hover:bg-neutral-50 transition-colors`. Each card: bold uppercase title + `text-[10px] border border-black px-1` module tag + `text-xs text-neutral-500` description. Bullet items: `<span class="w-2 h-2 bg-black mr-2"></span>` black squares.

### Buttons
```css
.btn-utility { border: 1px solid #000; background: #fff; px-8 py-4; font-bold uppercase tracking-widest; }
.btn-utility:hover { background: #000; color: #fff; }
.btn-utility-alt { border: 1px solid #000; background: neutral-100; }
.btn-utility-alt:hover { background: neutral-200; }
```
No shadow, no transform.

### Marquee
`bg-black text-white py-3 border-b border-black`. Content: monospace uppercase tracking-widest, `///` separators.

### Module Section
`bg-neutral-100 py-16`. Header: `flex items-end justify-between border-b border-black pb-4` — title left, `text-xs font-mono` index right (e.g. `INDEX: 004-A`). Grid: `gap-px bg-black border border-black`. Each cell `bg-white hover:bg-neutral-50`.

### Footer
`bg-neutral-100 border-t border-black`. 4-column `divide-x divide-black border-b border-black`. Link lists prefixed with `>`. Bottom bar `bg-white` with `©` and status message.

### Implementation
- No Google Fonts — use Tailwind `font-mono`
- `selection:bg-black selection:text-white`
- `antialiased`
- `min-h-screen flex flex-col` on outer container

---

## Dark Cinema Style

Near-black background, red accent with glow effect, Bebas Neue display font, noise texture, diagonal stripe patterns, and floating animated labels. Cinematic and visceral. Use when asked for "dark cinema style".

### Typography
Bebas Neue (headlines/display, `letter-spacing: 0.02em`) + DM Sans (body, weights 300–700). Headings: `font-display text-[clamp(72px,14vw,180px)] leading-none`. Body: `font-body text-white/60 font-light leading-relaxed`.

### Colors
```
Background:    #0a0a0a
Text:          #f0ede8   (warm off-white)
Red accent:    #dc2626   (red-600)
Red hover:     #ef4444   (red-500)
Red glow:      box-shadow: 0 0 60px rgba(220,38,38,0.35), 0 0 120px rgba(220,38,38,0.15)
Yellow float:  #facc15
Text-outline:  -webkit-text-stroke: 2px #f0ede8; color: transparent;
```

### Noise Overlay
```css
body::before {
  content: ''; position: fixed; inset: 0;
  background-image: url("data:image/svg+xml,%3Csvg viewBox='0 0 256 256' xmlns='http://www.w3.org/2000/svg'%3E%3Cfilter id='noise'%3E%3CfeTurbulence type='fractalNoise' baseFrequency='0.9' numOctaves='4' stitchTiles='stitch'/%3E%3C/filter%3E%3Crect width='100%25' height='100%25' filter='url(%23noise)' opacity='1'/%3E%3C/svg%3E");
  opacity: 0.035; pointer-events: none; z-index: 9999;
}
```

### Red Glow
```css
.red-glow { box-shadow: 0 0 60px rgba(220,38,38,0.35), 0 0 120px rgba(220,38,38,0.15); }
.text-red-glow { text-shadow: 0 0 40px rgba(220,38,38,0.6); }
```
Apply `.red-glow` to CTA buttons. Apply `.text-red-glow` to red accent text in headline.

### Hero Background
```html
<!-- Layered behind hero content -->
<div class="absolute top-1/4 left-1/2 -translate-x-1/2 -translate-y-1/2 w-[700px] h-[700px] rounded-full bg-red-900/20 blur-[140px]"></div>
<div class="absolute bottom-0 left-0 w-[400px] h-[400px] rounded-full bg-red-800/10 blur-[100px]"></div>
<!-- Subtle white grid -->
<div class="absolute inset-0 opacity-5" style="background-image: linear-gradient(to right, #fff 1px, transparent 1px), linear-gradient(to bottom, #fff 1px, transparent 1px); background-size: 80px 80px;"></div>
```

### Floating Animated Labels
Decorative elements floating around the hero (hidden on mobile):
```css
@keyframes float { 0%,100% { transform: translateY(0) rotate(-3deg); } 50% { transform: translateY(-14px) rotate(-3deg); } }
@keyframes float2 { 0%,100% { transform: translateY(0) rotate(2deg); } 50% { transform: translateY(-10px) rotate(2deg); } }
.float-1 { animation: float 5s ease-in-out infinite; }
.float-2 { animation: float2 6s ease-in-out infinite 1s; }
```
Labels: `bg-red-600 text-white font-display text-2xl px-5 py-3 rounded-2xl rotate-[-3deg] shadow-2xl`.

### Word Cycle Animation
Cycling word in headline:
```css
.word-cycle span { position: absolute; left: 0; opacity: 0; transform: translateY(20px); transition: none; }
.word-cycle span.active { opacity: 1; transform: translateY(0); transition: opacity 0.4s ease, transform 0.4s ease; }
.word-cycle span.exit { opacity: 0; transform: translateY(-20px); transition: opacity 0.3s ease, transform 0.3s ease; }
```

### Fade-Up Stagger
```css
@keyframes fadeUp { from { opacity: 0; transform: translateY(32px); } to { opacity: 1; transform: translateY(0); } }
.fade-up { opacity: 0; animation: fadeUp 0.7s ease forwards; }
.delay-1 { animation-delay: 0.1s; } /* through .delay-5 */
```

### Stat Chips (below hero CTA)
`bg-white/5 border border-white/10 rounded-lg px-4 py-2 text-sm` — colored bold value + muted label.

### Feature Cards
`feat-card`: `border border-white/10 rounded-2xl bg-white/5 p-6`. Hover: `translateY(-4px); box-shadow: 0 20px 60px rgba(0,0,0,0.5)`.

### Tape/Banner Strip
```css
.tape { background: #dc2626; transform: rotate(-1.5deg); margin: 0 -20px; }
```
Full-width red diagonal tape between sections. `border-y-4 border-red-800 py-3 overflow-hidden`.

### Zigzag Pattern
```css
.zigzag {
  background: linear-gradient(135deg, #0a0a0a 25%, transparent 25%) -20px 0,
              linear-gradient(225deg, #0a0a0a 25%, transparent 25%) -20px 0,
              linear-gradient(315deg, #0a0a0a 25%, transparent 25%),
              linear-gradient(45deg, #0a0a0a 25%, transparent 25%);
  background-size: 40px 40px; background-color: #111;
}
```

### Diagonal Stripe (section divider bg)
```css
.stripe-bg {
  background-image: repeating-linear-gradient(
    -45deg, transparent, transparent 8px, rgba(255,255,255,0.025) 8px, rgba(255,255,255,0.025) 9px
  );
}
```

### Navigation
`fixed nav-blur bg-black/60 border-b border-white/5`. Minimal: logo + text links `text-white/60 hover:text-white` + red CTA `bg-red-600 hover:bg-red-500 rounded-lg`.

### CTA Section
Large red gradient CTA: `bg-gradient-to-r from-red-900 to-red-600`. Or dark section with red glowing button.

### Implementation
- Google Fonts: Bebas Neue + DM Sans
- `overflow-x: hidden` on body
- Custom scrollbar: `6px, track #0a0a0a, thumb #dc2626, border-radius: 3px`
- Pulse dot: `@keyframes pulse-dot { 0%,100% { opacity: 1; transform: scale(1); } 50% { opacity: 0.5; transform: scale(1.5); } }`
- Text outline: `-webkit-text-stroke: 2px #f0ede8; color: transparent` for ghost headline words

---

## Dark Mono Style

Dark zinc blog/dashboard with a monospace terminal feel, cyan + pink color pops, and a subtle scanline texture. Compact, readable, and techy. Use when asked for "dark mono style".

### Typography
System monospace: `ui-monospace, SFMono-Regular, Menlo, Monaco, Consolas, monospace`. Everything monospace. Headlines `font-black tracking-tighter`. Labels `text-[10px] font-bold uppercase tracking-widest`. Small `border-radius: 6px` ("sharp") used everywhere instead of large rounding.

### Colors
```css
surface: #09090b;   /* page background — near-black zinc */
panel:   #18181b;   /* card/component background */
subtle:  #27272a;   /* borders */
muted:   #71717a;   /* muted text */
accent:  #a1a1aa;   /* secondary text */
bright:  #fafafa;   /* primary text */
pop:     #22d3ee;   /* cyan — primary accent */
warm:    #f472b6;   /* pink — secondary accent */
```

### Scanline Texture
Applied to `body` via a class:
```css
.scanline {
  background-image: repeating-linear-gradient(
    0deg, transparent, transparent 1px,
    rgba(255,255,255,0.015) 1px, rgba(255,255,255,0.015) 2px
  );
}
```

### Nav Pills
Small `rounded-sharp` (6px) pill buttons:
- Active/current: `bg-pop text-surface` (cyan bg, dark text)
- Inactive: `border border-subtle bg-panel text-accent hover:text-pop hover:border-pop/30 transition-colors`
- All: `px-3 py-1.5 text-[11px] font-black uppercase tracking-wider inline-flex items-center gap-2`
- Include a tiny Font Awesome icon at `text-[9px]`

### Cards / Panels
`border border-subtle bg-panel rounded-sharp`. Inner content uses `text-bright` for titles, `text-accent` for meta, `text-muted` for body. Breadcrumb tags: `rounded-sharp bg-panel border border-subtle px-2 py-1 text-[10px] font-bold uppercase tracking-widest`.

### Forms / Inputs
`rounded-sharp border border-subtle bg-panel text-bright placeholder:text-muted`. Focus: `box-shadow: 0 0 0 2px #22d3ee` (no outline, custom glow). Search icon inside left padding.

### Section Headings
`border-b border-subtle pb-2 mb-4 text-bright font-black uppercase tracking-tight` — subtle bottom border under each section title.

### Sidebar Layout
2-column: `sidebar-sticky` (`position: sticky; top: 60px`) on the sidebar. Main content `lg:col-span-8` or `3`, sidebar `lg:col-span-4` or `1`.

### Status/Pulse Indicator
Inline `animate-pulse` dot in cyan next to the logo: `h-1.5 w-1.5 rounded-full bg-pop animate-pulse`.

### Selection
`::selection { background: #22d3ee; color: #09090b; }`

### Focus Ring
`a:focus-visible { outline: 2px solid #22d3ee; outline-offset: 2px; border-radius: 4px; }`

### Implementation
- Font Awesome 6 for small nav icons
- `border-radius: sharp` = 6px, used via Tailwind `rounded-sharp` custom config
- `antialiased` on body
- `max-w-7xl` container, `px-4`
- No Google Fonts — system monospace only

---

## Blueprint Style

Deep engineering-blueprint aesthetic — midnight blue background, white grid lines, Courier Prime typeface, crosshair cursor, technical annotation details. Use when asked for "blueprint style".

### Typography
Courier Prime (Google Fonts, weights 400 + 700 + italic). Everything monospace. `font-bold uppercase tracking-tight` for headlines. `tracking-widest uppercase text-sm` for nav. Annotation micro-text at `text-[10px] uppercase`.

### Colors
```css
--bp-blue:  #003366;   /* page background */
--bp-paper: #002b55;   /* slightly darker panels */
--bp-white: #F0F8FF;   /* all text and borders */
--grid-line: rgba(255,255,255,0.15);
```

### Body Background (Grid)
```css
body {
  background-color: #003366;
  cursor: crosshair;
  background-image:
    linear-gradient(var(--grid-line) 1px, transparent 1px),
    linear-gradient(90deg, var(--grid-line) 1px, transparent 1px),
    linear-gradient(rgba(255,255,255,0.05) 2px, transparent 2px),
    linear-gradient(90deg, rgba(255,255,255,0.05) 2px, transparent 2px);
  background-size: 20px 20px, 20px 20px, 100px 100px, 100px 100px;
  background-position: -1px -1px;
}
```

### Container
`max-w-[1600px] mx-auto border-l-8 border-r-8 border-white/10` — flanked by vertical ruler lines:
```css
.ruler-line { position: fixed; top: 0; bottom: 0; width: 1px; background: rgba(255,255,255,0.2); z-index: -1; }
```
Place `.ruler-line.left-4` and `.ruler-line.right-4`.

### Navigation
`sticky top-0 border-b-2 border-white backdrop-blur-sm bg-[rgba(0,51,102,0.9)]`. Logo: `border-2 border-white px-3 py-1 hover:bg-white hover:text-[#003366]` with "Fig. 1.0" annotation + product name. Nav links: underline animation (`w-0 h-px bg-white group-hover:w-full transition-all duration-300`). CTA: `border border-white px-6 py-2 hover:bg-white hover:text-[#003366]` with bracket text `[ Get_Product ]`.

### Cards
`border border-white/60 p-6 relative hover:bg-white/5 transition-colors`. Corner decorators:
- Top-right: `absolute top-2 right-2 text-[10px] text-white/50` — drawing number (DWG-01)
- Bottom-right: `absolute -bottom-1 -right-1 w-2 h-2 bg-white` — corner dot accent

### Section Headers
`text-2xl font-bold uppercase tracking-widest border border-white px-4 py-1` label + `flex-grow border-t border-dashed border-white/50` dashed rule line + `text-xs uppercase` sheet number (Sheet 02). Pattern: `flex items-center gap-4 mb-10`.

### Hero
`border-b-2 border-white`. Corner bracket decorators: `absolute top-10 left-10 w-20 h-20 border-l border-t border-white/50` (top-left corner) and matching bottom-right. Center-aligned content. Badge: `inline-block border border-white px-4 py-1 text-xs uppercase tracking-widest`. Headline: bold, uppercase, `border-b-4 border-double border-white` on key phrase.

### Quote/Description Block
`border-l-2 border-white pl-6 relative`. End-cap marks: `absolute -left-[9px] top-0 w-4 h-px bg-white` (horizontal tick top and bottom).

### Filter Bar
`flex flex-wrap gap-px bg-white/20 p-px border border-white/30`. Each filter link: `flex-grow text-center bg-[#003366] text-white/80 hover:bg-[#0047AB] py-2 px-4 uppercase text-xs tracking-wider`.

### Selection
`::selection { background: white; color: #003366; }`

### Implementation
- Google Fonts: Courier Prime only
- `cursor: crosshair` on body
- `border-l-8 border-r-8 border-white/10` on outer container for binding-edge feel
- No color accents — pure white on blue only
- Mobile menu uses `border border-dashed border-white` buttons

---

## Monolith Style

Clean white brutalism with a dark navy shadow system, thick top border, monospace font, and no color accents — raw and serious. Use when asked for "monolith style".

### Typography
System monospace (`font-mono` Tailwind default). All headings forced `font-weight: 900`. `uppercase tracking-tighter leading-none` for display headings. `uppercase tracking-wide` for card headings. Body text `text-dark` (`#111827`).

### Colors
```css
--color-dark:   #111827;   /* text, borders, shadows */
--color-light:  #ffffff;   /* background */
--color-medium: #4b5563;   /* muted text, secondary */
```
No color accents whatsoever — strictly dark/light/medium.

### Top Border
`border-t-8 border-dark` on `<body>` — thick top edge is the defining visual signature.

### Shadow System
```css
.shadow-monolith    { box-shadow: 4px 4px 0px #111827; }
.shadow-monolith-lg { box-shadow: 8px 8px 0px #111827; }
.shadow-monolith-hover:hover { box-shadow: 6px 6px 0px #111827; transform: translate(-2px, -2px); }
```

### Buttons
```css
.brutal-btn {
  border: 2px solid #111827;
  box-shadow: 3px 3px 0px #111827;
  transition: all 0.1s ease-out;
}
.brutal-btn:hover  { box-shadow: 4px 4px 0px #111827; transform: translate(-1px, -1px); }
.brutal-btn:active { box-shadow: 1px 1px 0px #111827; transform: translate(2px, 2px); }
```
Primary: `bg-dark text-light`. Secondary: `bg-light text-dark`. Both use `.brutal-btn`. Text `font-extrabold uppercase`.

### Cards
`border-4 border-dark shadow-monolith`. Icon area: `bg-dark p-4 inline-block border-2 border-dark shadow-monolith` — dark square with light icon inside. Card headings: `font-extrabold uppercase tracking-wide`.

### Section Headings
`text-4xl md:text-5xl font-extrabold uppercase text-center border-b-4 border-medium pb-4` — bottom border on section titles in medium gray.

### Hero
`border-b-4 border-dark shadow-monolith-lg`. 2-column grid (headline + visual panel). Headline: `border-b-4 border-dark pb-4`. Subtext: `border-l-4 border-medium pl-4`. Visual panel: `border-4 border-dark shadow-monolith-lg` with `absolute inset-0 grid grid-cols-3 grid-rows-3` grid overlay at 20% opacity.

### Pricing/CTA Cards
`border-4 border-dark shadow-monolith-lg` panels. Highlight row: `bg-dark text-light` header bar. Feature lists with `border-b border-medium` row separators.

### Implementation
- No Google Fonts — system monospace
- `antialiased leading-tight`
- `max-w-7xl mx-auto px-4`
- Section padding: `py-16 md:py-24`
- Font Awesome for icons (dark square icon boxes: `bg-dark p-4 inline-block`)

---

## Dot Grid Style

Gray dotted-pattern background, Archivo Black display font, Space Mono body, hot pink accent, and large hard black shadows. Content-dense portal layout. Use when asked for "dot grid style".

### Typography
Archivo Black (display/headlines, `font-display`) + Space Mono (body/labels, `font-mono`). Headlines `font-display text-4xl md:text-6xl leading-none tracking-tight`. Labels: `font-mono font-bold uppercase text-xs tracking-widest`.

### Colors
```
Background:    #e5e7eb  (gray-200, dotted)
Text:          #000000
Accent:        #F5276C  (hot pink)
White surface: #ffffff
Black:         #000000
```

### Dotted Background
```css
body {
  background-color: #e5e7eb;
  background-image: radial-gradient(black 1px, transparent 0);
  background-size: 25px 25px;
}
```

### Shadow System
```css
shadow-hard:    10px 10px 0px 0px #000000;
shadow-hard-sm:  5px 5px 0px 0px #000000;
```

### Cards / Panels
`border-4 border-black bg-white shadow-hard`. Header bars within cards: `bg-black text-white font-bold uppercase text-xs` bar across the top. Logo/header card: `border-4 border-black bg-white shadow-hard p-6 md:p-10` with `overflow-hidden relative`. Ghost watermark text: `absolute -right-10 -top-10 text-[150px] opacity-20 font-display select-none pointer-events-none`.

### Buttons (Nav)
```css
.btn-primary { border: 2px solid black; background: black; color: white; font-bold; }
.btn-primary:hover { background: #F5276C; color: black; }
.btn-secondary { border: 2px solid black; background: white; }
.btn-secondary:hover { background: #F5276C; }
```
No border-radius. All buttons rectangular with `px-3 py-2 text-sm font-bold`.

### Link Items
```css
a.link-item {
  text-decoration: underline;
  text-underline-offset: 4px;
  text-decoration-thickness: 2px;
  text-decoration-color: #d1d5db;
}
a.link-item:hover { background-color: #F5276C; color: black; text-decoration: none; }
```

### Content Grid
12-column grid: main panel `lg:col-span-8`, sidebar `lg:col-span-4`. Main: `border-4 border-black bg-white shadow-hard`. Content rows: `divide-y-2 divide-black`. Each row: date badge + title + source domain.

### Status Badge
`inline-flex items-center gap-2 bg-black px-2 py-1 text-xs text-white` — "STATUS THING / ONLINE" inline chip.

### Section Labels
`font-display text-4xl leading-none tracking-tight` headline + `h-px flex-grow` dashed rule line layout.

### Implementation
- Google Fonts: Archivo Black + Space Mono
- `selection:bg-[#F5276C] selection:text-black` or yellow
- No border-radius — all rectangular
- `focus-visible:ring-4 focus-visible:ring-[accent]` for accessibility
- Font Awesome for any icons needed

---

## Pink Neo Style

Hot pink dotted background with a floating white content container, Archivo Black + Space Mono fonts, and a pink/yellow/blue neon palette. Archive/directory-style layout. Use when asked for "pink neo style".

### Typography
Archivo Black (`font-head`) for all headings and display text. Space Mono (`font-mono`) for body, labels, and nav. Headlines `font-head uppercase tracking-tighter leading-none`. All caps on section titles.

### Colors
```
Page background: #FF90E8  (hot pink — the entire page)
neo-pink:        #FF90E8
neo-yellow:      #FFC900
neo-blue:        #23A6F0
neo-black:       #1a1a1a
neo-white:       #fafafa
```

### Dotted Background
```css
body {
  background-color: #FF90E8;
  background-image: radial-gradient(#000000 1px, transparent 1px);
  background-size: 20px 20px;
}
```

### Border System
```css
.neo-border { border: 3px solid #000000; }
```
Used on all cards, panels, and containers.

### Shadow System
```css
shadow-neo:       5px 5px 0px 0px #000000;
shadow-neo-sm:    3px 3px 0px 0px #000000;
shadow-neo-hover: 2px 2px 0px 0px #000000;
```

### Main Container / Header Card
`bg-neo-white neo-border shadow-neo p-6 md:p-10 flex flex-col md:flex-row justify-between items-center gap-6 relative overflow-hidden`. Ghost letters watermark: `absolute -right-10 -top-10 text-[200px] text-neo-pink opacity-20 font-head select-none pointer-events-none`.

### Cards
`bg-white neo-border shadow-neo`. Card header bar: `border-b-3 border-black p-2 bg-neo-black text-white font-bold text-xs text-center uppercase`. Hover: `group-hover:translate-x-1 transition-transform`. Sticker badge: `absolute -top-3 -right-3 bg-neo-pink border-2 border-black px-2 py-1 text-xs font-bold transform rotate-12 z-20`.

### Link/Tag Buttons
```css
/* Each gets an accent color bg */
.tag-yellow { background: #FFC900; border: 2px solid black; }
.tag-blue   { background: #23A6F0; color: white; border: 2px solid black; }
.tag-pink   { background: white; border: 2px solid black; }
.tag-black  { background: #1a1a1a; color: white; border: 2px solid black; }
/* All: px-3 py-2 font-bold text-xs uppercase */
/* hover: bg-neo-pink text-black -translate-y-1 shadow-neo-sm transition-all */
```

### Content List Rows
`divide-y-2 divide-black`. Each row: date badge `bg-neo-black text-white px-3 py-1 border-2 border-transparent group-hover:border-black group-hover:bg-neo-pink group-hover:text-black` + title (hover: `text-neo-blue translate-x-1`) + source domain `text-[10px] text-gray-500 font-bold uppercase tracking-widest` + arrow `→` fades in on hover.

### Mac Traffic Lights
`h-3 w-3 rounded-full border border-white` — pink/yellow/blue dots in card headers for decoration.

### Layout
Content wrapped in `max-w-7xl mx-auto` with `p-2 md:p-8` body padding. Full-width sections as separate cards. Sidebar+main: `lg:col-span-1` sidebar sticky, `lg:col-span-3` main.

### Partner/Tag Strip
`bg-white neo-border shadow-neo p-4 md:p-6`. Section title: `font-head text-xl uppercase border-b-4 border-black inline-block`. Tags in `flex flex-wrap gap-3`.

### Scrollbar
```css
::-webkit-scrollbar { width: 12px; }
::-webkit-scrollbar-track { background: #FF90E8; border-left: 3px solid black; }
::-webkit-scrollbar-thumb { background: #000; border: 2px solid #FF90E8; }
```

### Implementation
- Google Fonts: Archivo Black + Space Mono
- `selection:bg-neo-yellow selection:text-black`
- Body: `p-2 md:p-8 min-h-screen flex flex-col`
- Font Awesome optional for icons

---

## Glassmorphism Style

Frosted glass cards floating over vivid gradient mesh backgrounds. Translucent surfaces, soft blurs, glowing borders. Feels futuristic and airy. Use when asked for "glassmorphism style".

### Typography
Outfit or DM Sans (Google Fonts, weights 300–800). Headlines `font-extrabold tracking-tight`. Body `font-light leading-relaxed`. Labels `font-semibold text-xs uppercase tracking-widest`.

### Colors
```
No fixed palette — the gradient mesh provides color.
Text on glass:  rgba(255,255,255,0.95)  (primary)
Text muted:     rgba(255,255,255,0.6)
Glass surface:  rgba(255,255,255,0.08) to rgba(255,255,255,0.15)
Glass border:   rgba(255,255,255,0.2)
```
Background is always a vivid gradient mesh — never a flat color.

### Background Gradient Mesh
Layer multiple large blurred blobs over a deep base color:
```css
body { background: #0f0f1a; }  /* deep base */

/* Mesh blobs — position freely */
.blob-1 { position:fixed; width:600px; height:600px; border-radius:50%; background: radial-gradient(circle, #7c3aed, transparent 70%); top:-200px; left:-100px; opacity:0.6; filter:blur(80px); }
.blob-2 { position:fixed; width:500px; height:500px; border-radius:50%; background: radial-gradient(circle, #06b6d4, transparent 70%); bottom:-100px; right:-100px; opacity:0.5; filter:blur(80px); }
.blob-3 { position:fixed; width:400px; height:400px; border-radius:50%; background: radial-gradient(circle, #ec4899, transparent 70%); top:40%; left:40%; opacity:0.3; filter:blur(100px); }
```
Choose a color story: purple+cyan, pink+orange, blue+green, etc.

### Glass Cards
```css
.glass-card {
  background: rgba(255,255,255,0.08);
  backdrop-filter: blur(20px);
  -webkit-backdrop-filter: blur(20px);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.3), inset 0 1px 0 rgba(255,255,255,0.15);
}
.glass-card:hover {
  background: rgba(255,255,255,0.12);
  border-color: rgba(255,255,255,0.25);
  transform: translateY(-4px);
}
```

### Glass Buttons
```css
.glass-btn {
  background: rgba(255,255,255,0.15);
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255,255,255,0.25);
  border-radius: 100px;
  color: white; font-weight: 600;
  box-shadow: 0 4px 15px rgba(0,0,0,0.2), inset 0 1px 0 rgba(255,255,255,0.2);
}
.glass-btn-accent {
  background: rgba(124,58,237,0.7);  /* tinted with brand color */
  backdrop-filter: blur(10px);
  border: 1px solid rgba(255,255,255,0.2);
}
```

### Nav
`backdrop-filter: blur(20px); background: rgba(0,0,0,0.2); border-bottom: 1px solid rgba(255,255,255,0.1)`. Logo icon: small glass card. Links: `text-white/60 hover:text-white`.

### Icon Boxes
`background: rgba(255,255,255,0.1); backdrop-filter: blur(10px); border: 1px solid rgba(255,255,255,0.15); border-radius: 14px` — tinted with the feature's accent color at low opacity.

### Stat Numbers
Large frosted number cards. Number in white, label in `text-white/60`. Each card optionally tinted differently.

### Input Fields
```css
.glass-input {
  background: rgba(255,255,255,0.06);
  border: 1px solid rgba(255,255,255,0.15);
  border-radius: 12px; color: white;
}
.glass-input:focus { border-color: rgba(255,255,255,0.4); background: rgba(255,255,255,0.1); outline: none; }
```

### Implementation
- Google Fonts: Outfit or DM Sans
- `pointer-events-none` and `fixed inset-0` on all blob elements
- All body text `text-white` or `text-white/[opacity]`
- `selection:bg-white/20 selection:text-white`
- `antialiased` on body
- Sections: `py-24 px-6`, max-width `max-w-6xl`
- Avoid sharp edges — everything rounded-2xl or higher

---

## Newspaper Style

Black ink on off-white newsprint. Serif headlines, column grid layouts, editorial cutlines, and print-inspired decoration. Timeless and authoritative. Use when asked for "newspaper style".

### Typography
Playfair Display (headlines, weights 400–900) + Source Serif 4 or Lora (body, weights 400–600). Headlines `font-serif font-black italic tracking-tight leading-none`. Subheads `font-serif font-bold uppercase tracking-widest text-sm`. Body `font-serif text-base leading-relaxed`. Labels/captions `font-sans text-xs uppercase tracking-widest`.

### Colors
```
Paper:      #f5f0e8   (warm newsprint off-white)
Ink:        #1a1a1a   (near-black)
Rule:       #1a1a1a   (border lines)
Muted:      #6b6560   (secondary text, captions)
Red accent: #c0392b   (section flag color — used sparingly)
```
No gradients. Flat colors only, like print.

### Page Background
`background: #f5f0e8` — warm off-white throughout. Optional paper texture via SVG noise filter at very low opacity.

### Masthead / Header
Full-width `border-b-4 border-ink`. Newspaper name in massive Playfair Display italic, centered. Below: thin rule line + date/edition/tagline in `font-sans text-xs uppercase tracking-[0.3em]`. Above: `border-t-2 border-ink` and thin rule.

### Column Grid
Multi-column editorial layout using CSS columns or explicit grid:
```css
.column-grid-3 { display: grid; grid-template-columns: repeat(3, 1fr); gap: 0; }
.column-divider { border-right: 1px solid #1a1a1a; padding-right: 24px; }
/* Each column gets padding-left: 24px except first */
```

### Article Cards
No box-shadow, no radius. `border-top: 3px solid #1a1a1a` accent on top of featured articles. Headline in Playfair italic. Byline: `font-sans text-xs uppercase text-muted`. Cutline (image caption): `font-sans text-xs italic text-muted border-t border-ink pt-1 mt-1`.

### Section Flags
`background: #1a1a1a; color: #f5f0e8; font-sans font-black uppercase tracking-widest text-xs px-3 py-1` — small banner before section headings. Or red variant: `background: #c0392b`.

### Pull Quote
`border-left: 4px solid #1a1a1a; border-right: 4px solid #1a1a1a; padding: 16px 24px; font-serif font-bold text-xl italic text-center`. Top and bottom `border-top: 1px solid #1a1a1a`.

### Rule Lines
Use `<hr>` or `border-t` liberally to divide sections — 3px for major, 1px for minor, double (`border-double`) for special separators.

### Drop Cap
```css
.drop-cap::first-letter {
  float: left; font-family: 'Playfair Display'; font-size: 4.5rem;
  font-weight: 900; line-height: 0.8; margin-right: 8px; margin-top: 4px;
}
```

### Buttons
`border: 2px solid #1a1a1a; background: #1a1a1a; color: #f5f0e8; font-sans font-bold uppercase tracking-widest px-6 py-3`. Hover: `background: transparent; color: #1a1a1a`. No border-radius. No shadows.

### Implementation
- Google Fonts: Playfair Display + Lora (or Source Serif 4)
- No rounded corners anywhere — print is rectangular
- `max-w-6xl mx-auto px-6`
- `antialiased` on body
- Sections separated by rule lines, not whitespace
- Image treatments: `filter: grayscale(20%) contrast(1.1)` for print-like photos

---

## Retro Terminal Style

Green phosphor text on a near-black screen. CRT scanlines, blinking cursors, glowing text, monospace typefaces. Feels like a 1980s computer terminal. Use when asked for "retro terminal style".

### Typography
System monospace or JetBrains Mono / Share Tech Mono (Google Fonts). Everything monospace. `uppercase` for labels and nav. Text at reduced opacity for depth: primary `opacity-90`, secondary `opacity-60`, dim `opacity-40`.

### Colors
```
Screen bg:       #0a0e0a   (near-black with green tint)
Phosphor green:  #00ff41   (bright CRT green)
Dim green:       #00b32d   (medium green)
Dark green:      #003b00   (very dark green, borders)
Amber variant:   #ffb000   (alternative amber terminal)
```
Pick one: green or amber. Don't mix.

### Body Background + Scanlines
```css
body {
  background-color: #0a0e0a;
  color: #00ff41;
  font-family: monospace;
}
/* CRT scanline overlay */
body::after {
  content: '';
  position: fixed; inset: 0;
  background: repeating-linear-gradient(
    0deg, rgba(0,0,0,0.15) 0px, rgba(0,0,0,0.15) 1px, transparent 1px, transparent 2px
  );
  pointer-events: none; z-index: 9999;
}
```

### Phosphor Glow
```css
.glow-text { text-shadow: 0 0 7px #00ff41, 0 0 14px #00ff41, 0 0 21px #00ff41; }
.glow-sm   { text-shadow: 0 0 4px #00ff41; }
.glow-box  { box-shadow: 0 0 8px #00ff41, 0 0 2px #00ff41 inset; }
```
Apply `.glow-text` to headlines. `.glow-sm` to body text. `.glow-box` to bordered elements.

### Screen Vignette
```css
body::before {
  content: ''; position: fixed; inset: 0; pointer-events: none; z-index: 9998;
  background: radial-gradient(ellipse at center, transparent 60%, rgba(0,0,0,0.7) 100%);
}
```

### Cards / Panels
```css
.terminal-panel {
  border: 1px solid #00ff41;
  box-shadow: 0 0 8px rgba(0,255,65,0.3), inset 0 0 8px rgba(0,255,65,0.05);
  background: rgba(0,255,65,0.03);
  border-radius: 0;
}
```
Panel headers: `background: #00ff41; color: #0a0e0a; font-weight: 700; padding: 4px 12px; text-transform: uppercase; letter-spacing: 0.1em`.

### Prompt / Code Lines
```
> SYSTEM ONLINE...
> LOADING MODULES [████████░░] 80%
> PRESS ANY KEY TO CONTINUE_
```
Progress bars: `[████████░░]` or CSS: `background: #00ff41` fill on `background: #003b00` track, `border: 1px solid #00ff41`.

### Blinking Cursor
```css
.cursor::after { content: '█'; animation: blink 1s step-end infinite; }
@keyframes blink { 0%,100%{opacity:1} 50%{opacity:0} }
```

### Buttons
```css
.term-btn {
  border: 1px solid #00ff41; background: transparent; color: #00ff41;
  font-family: monospace; uppercase; letter-spacing: 0.1em;
  box-shadow: 0 0 6px rgba(0,255,65,0.4);
}
.term-btn:hover { background: #00ff41; color: #0a0e0a; box-shadow: 0 0 16px rgba(0,255,65,0.8); }
```

### Nav
No background — floats on the dark screen. Links separated by ` // ` or `|`. Active link: `.glow-text`. Logo: `[PRODUCT_NAME]` in brackets.

### Boot Sequence Hero
Simulate a terminal boot: staggered lines of text appearing with delays, a progress bar, then the main headline "unlocks". Use `@keyframes fadeIn` with increasing `animation-delay` values.

### Implementation
- Google Fonts: Share Tech Mono or JetBrains Mono (optional — system mono works)
- `overflow-x: hidden`
- Custom scrollbar: `scrollbar-color: #00ff41 #0a0e0a; scrollbar-width: thin`
- `selection::bg: #00ff41; color: #0a0e0a`
- No border-radius anywhere
- All section padding via `padding: 80px 0`

---

## Memphis Style

Bold 80s/90s geometric shapes, confetti patterns, bright pastels, and playful squiggles. Chaotic but joyful. Use when asked for "memphis style".

### Typography
Fredoka One or Righteous (Google Fonts) for headlines. Nunito or Quicksand for body. Headlines `font-display font-bold` with no letter-spacing — friendly and rounded. Never rigid or cold.

### Colors
Memphis uses 5–6 simultaneously:
```
Background:   #fef9f0   (warm cream)
Hot pink:     #ff6b9d
Yellow:       #ffd93d
Turquoise:    #4ecdc4
Purple:       #a855f7
Coral:        #ff6b35
Mint:         #95e1d3
Black:        #2d2d2d   (outlines)
```
Rotate colors across cards, shapes, and accents. No single dominant brand color.

### Geometric Background Shapes
Scatter decorative SVG/CSS shapes across the page — none are interactive:
```css
/* Diagonal stripe block */
.stripe-block {
  background: repeating-linear-gradient(
    45deg, #ffd93d, #ffd93d 4px, transparent 4px, transparent 20px
  );
}
/* Dot pattern */
.dot-pattern {
  background-image: radial-gradient(#ff6b9d 2px, transparent 2px);
  background-size: 20px 20px;
}
/* Squiggle: use SVG inline or background-image */
```
Place these as `absolute` decorative elements behind content.

### Cards
```css
.memphis-card {
  background: white;
  border: 3px solid #2d2d2d;
  border-radius: 16px;
  box-shadow: 6px 6px 0px #2d2d2d;
  position: relative; overflow: hidden;
}
/* Top accent bar — different color per card */
.memphis-card::before {
  content: ''; position: absolute; top: 0; left: 0; right: 0;
  height: 6px; background: var(--card-accent);
}
```

### Buttons
```css
.memphis-btn {
  border: 3px solid #2d2d2d;
  border-radius: 100px;
  font-weight: 800; uppercase;
  box-shadow: 4px 4px 0px #2d2d2d;
}
.memphis-btn:hover { transform: translate(-2px, -2px); box-shadow: 6px 6px 0px #2d2d2d; }
.memphis-btn:active { transform: translate(2px, 2px); box-shadow: 2px 2px 0px #2d2d2d; }
```
Each button gets a different pastel background.

### Pills / Tags
`border: 2px solid #2d2d2d; border-radius: 100px; font-weight: 700; padding: 4px 12px` — each a different pastel bg color.

### Section Rotation
Each section gets a different background from the palette:
- Hero: `#fef9f0` (cream)
- Features: `#fff0f5` (pink tint)
- Stats: `#fffde7` (yellow tint)
- CTA: `#f0fffe` (turquoise tint)

### Icon Treatment
Icons inside `rounded-2xl` boxes, each a different pastel bg. Slightly rotated: `transform: rotate(5deg)` or `-rotate(3deg)`.

### Decorative Elements
- Scattered geometric shapes: triangles, circles, stars, diamonds — all `absolute`, `pointer-events-none`, varying opacities
- Zigzag dividers between sections
- Hand-drawn-style underlines: `border-bottom: 3px solid #ff6b9d; border-radius: 0 0 50% 50%`

### Implementation
- Google Fonts: Fredoka One + Nunito
- `antialiased`
- `overflow-x: hidden` on body
- Generous padding: `py-20 px-6`
- Card grids: `gap-6`, `grid-cols-1 md:grid-cols-2 lg:grid-cols-3`
- Never use more than 3 shapes per section or it becomes unreadable

---

## Luxury Style

Refined, editorial, and expensive-feeling. Cream backgrounds, a single serif display font, gold/brass accents, and generous whitespace. Less is more. Use when asked for "luxury style".

### Typography
Cormorant Garamond or Bodoni Moda (Google Fonts, weights 300–700) for all display and headlines. Optionally pair with Jost or Montserrat (300–400 weight only) for body. Headlines `font-serif font-light tracking-[0.15em] uppercase`. Body `font-sans font-light leading-loose tracking-wide`. All caps labels at `text-[11px] tracking-[0.3em]`.

### Colors
```
Background:  #f8f4ef   (warm cream)
Surface:     #ffffff
Text:        #1c1917   (warm near-black)
Muted:       #78716c
Gold:        #b8942a   (brass/gold — used sparingly)
Gold light:  #d4af6a
Border:      #e5ddd3   (warm light gray)
```

### Whitespace
The defining characteristic. Sections use `py-32 md:py-48`. Cards have `p-10 md:p-14`. Hero padding `pt-40 pb-32`. Never crowd elements — breathing room is the luxury.

### Section Dividers
Thin gold lines: `border-top: 1px solid #b8942a` centered, max-width 80px, `mx-auto`. Or: `<div class="w-16 h-px bg-gold mx-auto my-12">`.

### Cards
No box-shadow. `border: 1px solid #e5ddd3; background: white`. On hover: `border-color: #d4af6a` — the gold border appears.

### Headlines
`font-serif font-light text-5xl md:text-7xl tracking-[0.1em] uppercase leading-tight`. Often split across 2 lines with intentional line breaks. Italics used for emphasis: `<em class="font-serif italic not-italic font-light">`.

### Gold Accent Elements
- Thin gold line under logo
- Gold `•` bullet separators in nav
- Gold number labels on steps/features
- Gold top border on featured cards: `border-top: 2px solid #b8942a`
- Never fill a button entirely in gold — it reads cheap

### Buttons
```css
.lux-btn {
  border: 1px solid #1c1917; background: transparent; color: #1c1917;
  font-sans; font-light; uppercase; letter-spacing: 0.2em; font-size: 11px;
  padding: 14px 40px;
}
.lux-btn:hover { background: #1c1917; color: #f8f4ef; }
.lux-btn-gold { border-color: #b8942a; color: #b8942a; }
.lux-btn-gold:hover { background: #b8942a; color: white; }
```
No border-radius or minimal (2px). No shadows.

### Nav
No box-shadow. `border-bottom: 1px solid #e5ddd3`. Logo: brand name in serif, small. Links: `text-[11px] uppercase tracking-[0.2em] font-light`. Gold CTA button.

### Quote / Testimonial
Massive `"` character in gold at 6rem, `font-serif font-light`. Quote text in serif italic. Attribution in small caps.

### Pricing
No card. Just: price in large serif, a thin gold rule, then a feature list with `•` bullets, then a button. Centered. Maximum whitespace.

### Implementation
- Google Fonts: Cormorant Garamond (or Bodoni Moda) + Jost
- `antialiased`
- `max-w-5xl` container (narrower for luxury feel)
- Image treatment: slightly desaturated `filter: saturate(0.85) contrast(1.05)`
- No Font Awesome — use Unicode symbols (→ ← • ") or inline SVGs
- Every element should feel like it has room to breathe

---

## Skeuomorphic Style

Design that mimics real physical materials — brushed metal, leather, stitched fabric, paper, wood grain. Realistic depth, textures, and tactile shadows. Use when asked for "skeuomorphic style".

### Typography
SF Pro / system-ui for controls and labels. Georgia or a humanist serif for document-style content. Headlines `font-medium` (not too heavy — the materials carry the visual weight).

### Colors
No single palette — material determines color. Common materials and their values:
```
Metal (silver): background: linear-gradient(145deg, #e0e0e0, #c8c8c8); border: 1px solid #aaa;
Metal (dark):   background: linear-gradient(145deg, #4a4a4a, #2a2a2a);
Leather (tan):  background: #8B6914; texture via noise overlay
Paper:          background: #f9f6f0; subtle noise texture
Wood:           background: linear-gradient(90deg, #8B5E3C, #6B4423, #8B5E3C);
```

### Realistic Shadow System
```css
/* Raised / embossed element */
.raised {
  box-shadow:
    0 1px 0 rgba(255,255,255,0.6) inset,  /* top highlight */
    0 -1px 0 rgba(0,0,0,0.2) inset,       /* bottom inner shadow */
    0 4px 8px rgba(0,0,0,0.3),            /* drop shadow */
    0 1px 3px rgba(0,0,0,0.4);
}
/* Pressed / inset element */
.pressed {
  box-shadow:
    0 1px 0 rgba(255,255,255,0.3),
    0 2px 4px rgba(0,0,0,0.3) inset,
    0 1px 2px rgba(0,0,0,0.5) inset;
}
```

### Metal Button
```css
.metal-btn {
  background: linear-gradient(145deg, #e8e8e8 0%, #c0c0c0 50%, #d8d8d8 100%);
  border: 1px solid #999;
  border-radius: 8px;
  box-shadow: 0 1px 0 rgba(255,255,255,0.8) inset, 0 -1px 0 rgba(0,0,0,0.15) inset, 0 3px 6px rgba(0,0,0,0.25);
  color: #333; font-weight: 500; text-shadow: 0 1px 0 rgba(255,255,255,0.6);
}
.metal-btn:active { box-shadow: 0 1px 2px rgba(0,0,0,0.4) inset; transform: translateY(1px); }
```

### Stitched Edge
```css
.stitched {
  border: 2px solid rgba(0,0,0,0.2);
  outline: 3px dashed rgba(255,255,255,0.25);
  outline-offset: -8px;
}
```

### Paper Texture
```css
.paper {
  background: #f9f6f0;
  background-image: url("data:image/svg+xml,...");  /* SVG noise filter */
  box-shadow: 0 2px 4px rgba(0,0,0,0.1), 0 0 0 1px rgba(0,0,0,0.05);
  border-radius: 2px;
}
/* Page curl effect on corner: use ::after pseudo with gradient */
```

### Progress Bar
```css
.skeuo-progress-track {
  background: linear-gradient(to bottom, #b0b0b0, #d0d0d0);
  border-radius: 10px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.4) inset;
  border: 1px solid #999;
}
.skeuo-progress-fill {
  background: linear-gradient(to bottom, #4a90d9, #2d6fba);
  border-radius: 10px;
  box-shadow: 0 1px 0 rgba(255,255,255,0.4) inset;
}
```

### Toggle / Switch
Raised oval track, circular thumb with gloss highlight. All via `box-shadow` and `border-radius`.

### Implementation
- No Google Fonts required — system-ui for controls
- Every interactive element needs both a rest state (raised) and active state (pressed)
- Use `text-shadow: 0 1px 0 rgba(255,255,255,0.6)` on light buttons, `text-shadow: 0 -1px 0 rgba(0,0,0,0.3)` on dark
- Avoid flat colors — everything should have a gradient or texture
- Consistent light source: top-left. All highlights top-left, all shadows bottom-right.

---

## Vaporwave Style

Pastel purples and pinks over a retro grid floor, synthwave sun, glitch effects, and nostalgic 80s/early-internet energy. Dreamy, surreal, and deeply aesthetic. Use when asked for "vaporwave style".

### Typography
Audiowide or Orbitron (Google Fonts) for headlines. VT323 (Google Fonts) for terminal/retro text. Or: any bold italic serif for that early-web aesthetic. Headlines often in gradient or with neon glow.

### Colors
```
Background:    #0d0015   (very dark purple-black)
Primary pink:  #ff71ce
Primary blue:  #01cdfe
Purple:        #b967ff
Teal/cyan:     #05ffa1
Yellow:        #fffb96
Grid line:     rgba(179,103,255,0.3)
```

### Background — Retro Grid Floor + Sunset
```css
body { background: #0d0015; }

/* Retro perspective grid */
.retro-grid {
  position: fixed; bottom: 0; left: 0; right: 0; height: 50vh;
  background-image:
    linear-gradient(rgba(179,103,255,0.3) 1px, transparent 1px),
    linear-gradient(90deg, rgba(179,103,255,0.3) 1px, transparent 1px);
  background-size: 40px 40px;
  transform: perspective(300px) rotateX(60deg);
  transform-origin: bottom center;
  pointer-events: none;
}

/* Synthwave sun */
.synth-sun {
  position: fixed; bottom: 45vh; left: 50%; transform: translateX(-50%);
  width: 200px; height: 100px;
  background: linear-gradient(#ff71ce, #fffb96);
  border-radius: 100px 100px 0 0;
  /* Horizontal stripe cutouts — the sun lines */
  -webkit-mask-image: repeating-linear-gradient(
    to top, black 0px, black 8px, transparent 8px, transparent 14px
  );
  pointer-events: none;
}
```

### Gradient Text
```css
.vw-headline {
  background: linear-gradient(135deg, #ff71ce 0%, #b967ff 50%, #01cdfe 100%);
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  background-clip: text;
}
```

### Neon Glow
```css
.neon-pink  { text-shadow: 0 0 7px #ff71ce, 0 0 20px #ff71ce, 0 0 40px #ff71ce; }
.neon-blue  { text-shadow: 0 0 7px #01cdfe, 0 0 20px #01cdfe, 0 0 40px #01cdfe; }
.neon-box   { box-shadow: 0 0 10px #b967ff, 0 0 30px #b967ff, inset 0 0 10px rgba(185,103,255,0.2); }
```

### Cards
```css
.vw-card {
  background: rgba(13,0,21,0.8);
  border: 1px solid rgba(255,113,206,0.4);
  border-radius: 4px;
  box-shadow: 0 0 20px rgba(185,103,255,0.2), inset 0 0 20px rgba(185,103,255,0.05);
}
.vw-card:hover { border-color: #ff71ce; box-shadow: 0 0 30px rgba(255,113,206,0.4); }
```

### Glitch Effect
```css
@keyframes glitch {
  0%,100% { clip-path: inset(0 0 100% 0); transform: translate(0); }
  20% { clip-path: inset(30% 0 50% 0); transform: translate(-4px, 2px); }
  40% { clip-path: inset(60% 0 20% 0); transform: translate(4px, -2px); }
  60% { clip-path: inset(10% 0 70% 0); transform: translate(-2px, 1px); }
}
.glitch { position: relative; }
.glitch::before, .glitch::after {
  content: attr(data-text); position: absolute; top: 0; left: 0;
  color: #ff71ce; /* ::before */ /* #01cdfe for ::after */
  animation: glitch 4s infinite;
}
.glitch::after { color: #01cdfe; animation-delay: 0.1s; }
```

### Buttons
```css
.vw-btn {
  border: 1px solid #ff71ce;
  background: transparent;
  color: #ff71ce;
  border-radius: 2px;
  box-shadow: 0 0 10px rgba(255,113,206,0.4), inset 0 0 10px rgba(255,113,206,0.1);
  font-family: 'Audiowide'; uppercase; letter-spacing: 0.1em;
}
.vw-btn:hover { background: rgba(255,113,206,0.1); box-shadow: 0 0 20px rgba(255,113,206,0.8); }
```

### Scanlines
Same as terminal style — subtle horizontal line repeat at 2px interval, `opacity: 0.3`.

### Implementation
- Google Fonts: Audiowide + VT323
- `overflow-x: hidden`
- All sections layered on top of `fixed` background elements (grid + sun)
- Text colors: white, pink, cyan — never black on dark
- `selection:bg-[#b967ff] selection:text-white`
- Custom scrollbar: purple track, pink thumb

---

## Swiss Style

Pure International Typographic Style. Helvetica/system-sans, strict mathematical grid, black and red only, zero decorative elements. Function is the only form. Use when asked for "swiss style".

### Typography
System-ui / `-apple-system` (mimics Helvetica). Or load Inter at very low weights. Headlines `font-bold` (not black — 700 max). `tracking-tight`. Body: `font-normal text-sm leading-[1.5]`. Labels: `font-bold text-[11px] uppercase tracking-[0.15em]`. Numbers: `font-bold tabular-nums`.

### Colors
```
White:   #ffffff
Black:   #000000
Red:     #e60000   (one accent color only — use sparingly)
Gray:    #f4f4f4   (background sections)
```
Nothing else. Ever. If you feel the urge to add a color, remove it.

### Grid System
Everything aligns to a strict baseline grid. Use `gap-px bg-black` technique to create visible grid lines:
```css
.swiss-grid { display: grid; gap: 1px; background: #000; }
.swiss-grid > * { background: white; }
```
Columns: always mathematical (2, 3, 4, or 6). Never asymmetric.

### Borders and Rules
Only `1px solid #000` or `2px solid #000`. No rounding. No shadows. A thick `4px solid #e60000` rule is the maximum decoration allowed.

### Cards
No card visual treatment. Content sits directly in the grid cell: `padding: 24px; background: white`. The grid gap creates the borders.

### Typography Scale
```
Display:  clamp(48px, 8vw, 96px), font-bold, tracking-tight
H1:       48px, font-bold
H2:       32px, font-bold
H3:       20px, font-bold
Body:     14px, font-normal, leading-[1.6]
Label:    11px, font-bold, uppercase, tracking-[0.15em]
```

### Red Accent — Used For
- One featured stat number
- A single CTA button
- Section flag/label on the most important section
- Never on more than 2 elements per page

### Buttons
```css
.swiss-btn {
  background: #000; color: #fff;
  border: none; border-radius: 0;
  font-bold; uppercase; tracking-[0.1em]; font-size: 12px;
  padding: 12px 32px;
}
.swiss-btn:hover { background: #e60000; }
.swiss-btn-outline { background: transparent; color: #000; border: 2px solid #000; }
.swiss-btn-outline:hover { background: #000; color: #fff; }
```

### Nav
Pure white, `border-bottom: 2px solid #000`. Logo: just the product name in `font-bold uppercase tracking-tight`. Links: `font-bold text-xs uppercase tracking-[0.15em]`. No icons, no pills, no rounding.

### Section Structure
Large, bold section numbers: `text-[120px] font-bold text-black leading-none opacity-10` as background watermark. Section heading left-aligned, body text in a narrower column.

### Pull Quote / Stat
Single very large number: `text-[80px] font-bold leading-none`. A `4px solid #e60000` rule above it. Label below in `text-xs uppercase tracking-[0.2em]`.

### Implementation
- No Google Fonts — use system-ui stack
- `max-w-7xl mx-auto` — full use of available width
- Section padding: `py-24`
- No `box-shadow` anywhere — ever
- No `border-radius` anywhere — ever
- `antialiased` on body

---

## Dark Neon Style

Pure black background with multiple vivid neon colors bleeding and glowing simultaneously. Electric, high-energy, club/gaming aesthetic. Use when asked for "dark neon style".

### Typography
Rajdhani or Exo 2 (Google Fonts, weights 400–700) for body. Orbitron or Russo One for display headlines. Headlines `font-bold uppercase tracking-wide`. Body `font-medium leading-relaxed`.

### Colors
```
Background:  #050505   (pure near-black)
Neon pink:   #ff0090
Neon cyan:   #00f5ff
Neon green:  #39ff14
Neon purple: #bf00ff
Neon orange: #ff6600
White text:  #ffffff
```
Use 3–4 of these simultaneously. Each major feature/section gets its own neon color.

### Neon Glow System
```css
/* Per-color glow classes */
.glow-pink   { box-shadow: 0 0 5px #ff0090, 0 0 20px #ff0090, 0 0 40px #ff0090; }
.glow-cyan   { box-shadow: 0 0 5px #00f5ff, 0 0 20px #00f5ff, 0 0 40px #00f5ff; }
.glow-green  { box-shadow: 0 0 5px #39ff14, 0 0 20px #39ff14, 0 0 40px #39ff14; }
.glow-purple { box-shadow: 0 0 5px #bf00ff, 0 0 20px #bf00ff, 0 0 40px #bf00ff; }

.text-glow-pink   { text-shadow: 0 0 7px #ff0090, 0 0 20px #ff0090; }
.text-glow-cyan   { text-shadow: 0 0 7px #00f5ff, 0 0 20px #00f5ff; }
.text-glow-green  { text-shadow: 0 0 7px #39ff14, 0 0 20px #39ff14; }
```

### Background
Multiple `fixed` radial gradient blobs, very subtle:
```css
.bg-neon-1 { background: radial-gradient(ellipse at 20% 50%, rgba(255,0,144,0.15) 0%, transparent 60%); }
.bg-neon-2 { background: radial-gradient(ellipse at 80% 50%, rgba(0,245,255,0.1) 0%, transparent 60%); }
.bg-neon-3 { background: radial-gradient(ellipse at 50% 80%, rgba(191,0,255,0.1) 0%, transparent 60%); }
```

### Cards
```css
.neon-card {
  background: rgba(255,255,255,0.03);
  border: 1px solid rgba(255,255,255,0.08);
  border-radius: 12px;
}
/* Each card gets a neon color — applied as top border + subtle glow */
.neon-card-pink {
  border-top: 2px solid #ff0090;
  box-shadow: 0 0 20px rgba(255,0,144,0.15), inset 0 1px 0 rgba(255,0,144,0.1);
}
/* hover: glow intensifies, subtle translateY(-4px) */
```

### Buttons
```css
.neon-btn {
  background: transparent;
  border: 1px solid [neon-color];
  color: [neon-color];
  border-radius: 6px;
  font-weight: 600; uppercase; letter-spacing: 0.05em;
  box-shadow: 0 0 10px [neon-color-alpha];
}
.neon-btn:hover {
  background: [neon-color-alpha-10];
  box-shadow: 0 0 20px [neon-color], 0 0 40px [neon-color-alpha];
}
/* Filled variant: */
.neon-btn-filled {
  background: [neon-color];
  color: #050505;
  box-shadow: 0 0 20px [neon-color], 0 0 40px [neon-color-alpha];
}
```

### Neon Divider Lines
```css
.neon-line {
  height: 1px;
  background: linear-gradient(to right, transparent, #ff0090, #00f5ff, transparent);
}
```

### Nav
`background: rgba(5,5,5,0.8); backdrop-filter: blur(20px); border-bottom: 1px solid rgba(255,255,255,0.05)`. Logo with neon glow. Links `text-white/60 hover:text-white` — keep nav subtle.

### Stat Numbers
Very large `text-6xl font-bold` in neon color with `text-glow-[color]`. Unit/label in `text-white/50 text-sm`.

### Implementation
- Google Fonts: Rajdhani + Orbitron (or Russo One)
- `overflow-x: hidden`
- `selection:bg-[#ff0090] selection:text-black`
- Custom scrollbar: dark track, neon pink thumb
- `antialiased` on body
- Assign one neon color per major section/feature and be consistent

---

## Organic Style

Warm earthy tones, rounded organic shapes, natural textures, and a hand-crafted human feel. Cozy, approachable, and grounded. Use when asked for "organic style".

### Typography
Fraunces (Google Fonts, variable, weights 300–700) for headlines — it has beautiful ink-trap details. Plus Jakarta Sans or DM Sans (light weight) for body. Headlines `font-serif font-semibold` with `leading-tight`. Body `font-sans font-light leading-relaxed`.

### Colors
```
Background:   #faf7f2   (warm cream)
Surface:      #f2ede4   (warm tan card bg)
Deep surface: #e8dfd0   (darker tan)
Text:         #2c2416   (warm dark brown)
Muted:        #8a7560
Terracotta:   #c4623a   (primary accent)
Sage:         #6b8f6e   (secondary accent)
Sand:         #d4b896
Bark:         #7a5c44
```

### Background Texture
Subtle SVG noise or grain at very low opacity over the cream background. Optional: `background-image: url()` with a fine linen texture.

### Organic Shapes — Not Rectangles
All cards and containers use heavily irregular border-radius:
```css
.organic-card {
  border-radius: 30% 70% 70% 30% / 30% 30% 70% 70%;  /* blob */
  /* Or: */
  border-radius: 60px 20px 60px 20px;  /* asymmetric */
}
/* Vary each card's border-radius slightly for a handmade feel */
```
Or use SVG clip-paths for truly organic blobs.

### Section Dividers
Wavy SVG dividers between sections instead of straight lines:
```html
<svg viewBox="0 0 1440 60" preserveAspectRatio="none">
  <path d="M0,30 C360,60 1080,0 1440,30 L1440,60 L0,60 Z" fill="#f2ede4"/>
</svg>
```

### Cards
```css
.organic-card {
  background: #f2ede4;
  border-radius: 24px 8px 24px 8px;  /* slightly irregular */
  box-shadow: 4px 6px 20px rgba(44,36,22,0.08);
  border: 1px solid rgba(44,36,22,0.08);
}
.organic-card:hover { transform: rotate(-0.5deg) translateY(-4px); box-shadow: 6px 10px 28px rgba(44,36,22,0.12); }
```
Slight rotation on hover adds organic feel.

### Buttons
```css
.organic-btn {
  background: #c4623a; color: #faf7f2;
  border-radius: 100px;
  font-semibold; letter-spacing: 0.02em;
  box-shadow: 0 4px 14px rgba(196,98,58,0.3);
  border: none;
}
.organic-btn:hover { background: #b05530; transform: translateY(-2px); box-shadow: 0 6px 20px rgba(196,98,58,0.4); }
.organic-btn-outline {
  background: transparent; color: #c4623a;
  border: 2px solid #c4623a; border-radius: 100px;
}
```

### Icon Treatments
Icons inside rounded blobs with warm bg colors:
```css
.icon-blob {
  width: 56px; height: 56px;
  background: #e8dfd0;
  border-radius: 40% 60% 60% 40% / 40% 40% 60% 60%;  /* organic blob */
  display: flex; align-items: center; justify-content: center;
  color: #c4623a;
}
```

### Nav
`background: #faf7f2; border-bottom: 1px solid #e8dfd0`. Logo in serif. Links `font-light tracking-wide`. CTA: terracotta rounded button.

### Decorative Elements
- Leaf/plant SVG icons scattered as decorative accents
- Hand-drawn underlines via `border-bottom: 3px solid #c4623a` with `border-radius: 0 0 50% 50%`
- Small circular dot patterns at section edges

### Implementation
- Google Fonts: Fraunces + DM Sans
- `antialiased`
- `max-w-5xl` container (narrower, more intimate)
- `py-20 px-6` sections
- Avoid any sharp corners — even form inputs get `border-radius: 12px`
- Color accents: terracotta for CTAs/primary, sage for secondary tags/badges

---

## Neumorphism Style

Soft extruded UI — elements appear pushed out of or pressed into the same-color background. No hard borders, only dual light/dark shadows. Tactile and ethereal. Use when asked for "neumorphism style".

### Typography
Poppins or Nunito (Google Fonts, weights 300–600). Never heavy — the softness of the shadows IS the design. `font-medium` for headings, `font-light` for body. Letter spacing normal.

### Colors
```
Background:    #e0e5ec   (the defining color — everything matches it)
Text:          #4a5568
Muted:         #8898aa
Accent:        #6c63ff   (soft purple — or any single accent)
Shadow light:  #ffffff
Shadow dark:   #a3b1c6
```
The background, card backgrounds, and input backgrounds are ALL the same `#e0e5ec`. The shadows create all the depth.

### The Core Shadow Mechanic
```css
/* Raised / protruding element */
.neu-raised {
  background: #e0e5ec;
  border-radius: 16px;
  box-shadow: 6px 6px 12px #a3b1c6, -6px -6px 12px #ffffff;
}
/* Pressed / inset element */
.neu-pressed {
  background: #e0e5ec;
  border-radius: 16px;
  box-shadow: inset 4px 4px 8px #a3b1c6, inset -4px -4px 8px #ffffff;
}
/* Flat (hover state — subtle) */
.neu-flat {
  background: #e0e5ec;
  border-radius: 16px;
  box-shadow: 3px 3px 6px #a3b1c6, -3px -3px 6px #ffffff;
}
```

### Cards
Same as `.neu-raised`. No border. The shadow IS the card edge. Padding `24px–32px`. On hover: shadow grows slightly.

### Buttons
```css
.neu-btn {
  background: #e0e5ec;
  border: none; border-radius: 50px;
  box-shadow: 5px 5px 10px #a3b1c6, -5px -5px 10px #ffffff;
  color: #6c63ff; font-weight: 600;
  padding: 14px 32px;
  transition: all 0.2s ease;
}
.neu-btn:hover { box-shadow: 7px 7px 14px #a3b1c6, -7px -7px 14px #ffffff; }
.neu-btn:active { box-shadow: inset 4px 4px 8px #a3b1c6, inset -4px -4px 8px #ffffff; }
/* Accent variant */
.neu-btn-accent { background: #6c63ff; color: white; box-shadow: 5px 5px 10px rgba(108,99,255,0.4), -2px -2px 8px rgba(255,255,255,0.3); }
```

### Toggle / Switch
Oval track in `.neu-pressed` state. Circular thumb in `.neu-raised`. When active, thumb moves right and accent color appears.

### Input Fields
```css
.neu-input {
  background: #e0e5ec;
  border: none; border-radius: 12px;
  box-shadow: inset 4px 4px 8px #a3b1c6, inset -4px -4px 8px #ffffff;
  padding: 14px 18px; color: #4a5568;
}
.neu-input:focus { outline: none; box-shadow: inset 5px 5px 10px #a3b1c6, inset -5px -5px 10px #ffffff, 0 0 0 2px #6c63ff; }
```

### Progress Bar
Track: `.neu-pressed`. Fill: gradient from accent color, `border-radius: 50px`, no box-shadow on fill.

### Icon Circles
`width: 56px; height: 56px; border-radius: 50%` with `.neu-raised`. Accent-colored icon inside.

### Nav
Same background as page. Logo and links floating on the surface. Active link gets `.neu-pressed` treatment as a pill.

### Implementation
- Google Fonts: Poppins
- `background: #e0e5ec` on `html` AND `body` — every surface must match
- Never use hard borders or background color differences for cards
- Light source always top-left: light shadow top-left (`-x -y`), dark shadow bottom-right (`+x +y`)
- Works poorly on dark backgrounds — keep light

---

## Cyberpunk Style

High-contrast yellow and black warning stripes, HUD-style corner brackets, neon magenta/cyan on near-black, and a sense of industrial danger. Dystopian and electric. Use when asked for "cyberpunk style".

### Typography
Rajdhani or Barlow Condensed (Google Fonts, weights 400–700) for body. Orbitron for display/headlines. Headlines `font-bold uppercase tracking-wider`. Labels `font-mono text-xs uppercase tracking-[0.2em]`.

### Colors
```
Background:    #0a0a0f
Surface:       #12121a
Yellow:        #f5e642   (warning/primary)
Magenta:       #ff003c
Cyan:          #00f0ff
White:         #e8e8ff   (slightly blue-tinted white)
Muted:         #4a4a6a
Danger stripe: repeating yellow/black
```

### Warning Stripe Pattern
```css
.hazard-stripe {
  background: repeating-linear-gradient(
    -45deg,
    #f5e642 0px, #f5e642 10px,
    #0a0a0f 10px, #0a0a0f 20px
  );
  height: 6px;
}
```
Use as section dividers, button borders, header accents.

### Corner Bracket Decoration
```css
.bracket-corner { position: relative; }
.bracket-corner::before, .bracket-corner::after {
  content: ''; position: absolute; width: 16px; height: 16px;
  border-color: #f5e642; border-style: solid;
}
.bracket-corner::before { top: 0; left: 0; border-width: 2px 0 0 2px; }
.bracket-corner::after  { bottom: 0; right: 0; border-width: 0 2px 2px 0; }
```

### Cards
```css
.cyber-card {
  background: #12121a;
  border: 1px solid rgba(245,230,66,0.2);
  border-radius: 4px;
  position: relative;
  clip-path: polygon(0 0, calc(100% - 16px) 0, 100% 16px, 100% 100%, 16px 100%, 0 calc(100% - 16px));
}
.cyber-card::before { /* top-right angled corner accent */ }
```
Clipped corners (diagonal cut) are the signature card shape.

### Buttons
```css
.cyber-btn {
  background: #f5e642; color: #0a0a0f;
  border: none; border-radius: 2px;
  font-family: 'Orbitron'; font-weight: 700; uppercase; letter-spacing: 0.1em;
  clip-path: polygon(0 0, calc(100% - 8px) 0, 100% 8px, 100% 100%, 8px 100%, 0 calc(100% - 8px));
  box-shadow: 0 0 20px rgba(245,230,66,0.4);
}
.cyber-btn:hover { box-shadow: 0 0 40px rgba(245,230,66,0.8); filter: brightness(1.1); }
.cyber-btn-outline {
  background: transparent; color: #00f0ff;
  border: 1px solid #00f0ff;
  box-shadow: 0 0 10px rgba(0,240,255,0.3), inset 0 0 10px rgba(0,240,255,0.05);
}
```

### HUD Elements
- `[SYS::ONLINE]` style labels in monospace with `::before { content: '> '; color: #f5e642; }`
- Scan-line animated div: `animation: scan 3s linear infinite; background: linear-gradient(transparent, rgba(0,240,255,0.05), transparent)`
- Progress meters with `border: 1px solid #f5e642; box-shadow: 0 0 8px rgba(245,230,66,0.4)`

### Nav
`border-bottom: 1px solid rgba(245,230,66,0.3)`. Hazard stripe as 4px top bar. Logo: `[BRAND_ID]` in Orbitron. Links `font-mono text-xs uppercase tracking-[0.2em] text-white/60 hover:text-[#f5e642]`.

### Implementation
- Google Fonts: Orbitron + Rajdhani
- `clip-path` for angled corners on key elements
- Hazard stripes as borders, not fills
- `selection:bg-[#f5e642] selection:text-black`
- Custom scrollbar: dark track, yellow thumb
- `antialiased` on body

---

## Art Deco Style

1920s geometric glamour. Symmetrical layouts, sunburst and fan motifs, gold and black, sharp angular ornaments, and an air of opulent formality. Use when asked for "art deco style".

### Typography
Cormorant Garamond or Playfair Display (headlines, weights 300–700) for elegance. Josefin Sans (body/labels, weights 100–400) — ultra-light with wide tracking. Headlines `font-serif font-light tracking-[0.2em] uppercase`. Labels `font-sans font-light text-[11px] tracking-[0.4em] uppercase`.

### Colors
```
Background:   #0d0d0d   (near-black — or ivory #f5f0e8 for light variant)
Gold:         #c9a84c
Gold light:   #e8c97a
Gold dark:    #8b6914
Cream:        #f5f0e8
White:        #ffffff
```
Two variants: **Dark** (black + gold) or **Light** (ivory + gold). Choose one.

### Geometric Ornaments (CSS only)
```css
/* Sunburst lines radiating from center */
.sunburst {
  background: conic-gradient(
    from 0deg, transparent 0deg, transparent 8deg,
    #c9a84c 8deg, #c9a84c 9deg, transparent 9deg
  );
  border-radius: 50%;
}
/* Diamond shape */
.diamond { transform: rotate(45deg); width: 20px; height: 20px; background: #c9a84c; }
/* Fan/chevron pattern as border */
.fan-border {
  border-image: repeating-linear-gradient(
    90deg, #c9a84c 0, #c9a84c 8px, transparent 8px, transparent 16px
  ) 1;
}
```

### Section Dividers
Symmetrical ornamental dividers:
```html
<div class="flex items-center gap-4 my-8">
  <div class="flex-1 h-px bg-gold"></div>
  <div class="diamond"></div>
  <div class="w-2 h-2 rounded-full bg-gold"></div>
  <div class="diamond"></div>
  <div class="flex-1 h-px bg-gold"></div>
</div>
```

### Cards
```css
.deco-card {
  border: 1px solid #c9a84c;
  background: rgba(201,168,76,0.05);  /* dark variant */
  position: relative;
}
/* Gold corner ornaments via ::before/::after */
.deco-card::before {
  content: ''; position: absolute; top: -1px; left: -1px;
  width: 20px; height: 20px;
  border-top: 2px solid #c9a84c; border-left: 2px solid #c9a84c;
}
```

### Buttons
```css
.deco-btn {
  background: transparent; color: #c9a84c;
  border: 1px solid #c9a84c;
  font-family: 'Josefin Sans'; font-weight: 300;
  letter-spacing: 0.4em; text-transform: uppercase; font-size: 11px;
  padding: 14px 48px;
}
.deco-btn:hover { background: #c9a84c; color: #0d0d0d; }
.deco-btn-filled { background: #c9a84c; color: #0d0d0d; }
```

### Masthead / Logo Area
Large, centered product name with decorative line above and below. Flanked by symmetrical ornamental lines: `<hr class="border-gold w-32">` on each side.

### Typography Details
- All headline text centered
- `letter-spacing: 0.2em–0.4em` everywhere
- Numbers in `font-serif font-light` for elegance
- `<em>` elements in italic serif

### Implementation
- Google Fonts: Cormorant Garamond + Josefin Sans
- Strict symmetry — centered layout throughout
- `max-w-4xl mx-auto` — narrower column for formality
- `antialiased`
- Gold used only on ornaments, dividers, borders, and CTAs — never as fill backgrounds

---

## Isometric Style

3D isometric grid with flat-color objects giving depth without perspective. Layered, structural, and playful. Use when asked for "isometric style".

### Typography
IBM Plex Sans or Inter (clean geometric sans). Headlines `font-bold tracking-tight`. Labels `font-mono text-xs uppercase`. The typography is secondary — the isometric illustrations carry the design.

### Colors
```
Background:   #f0f4ff   (soft blue-white)
Top face:     #7c9dff   (lightest — light source)
Left face:    #4a6fe8   (medium)
Right face:   #2a4db5   (darkest — shadow)
Accent top:   #a78bfa
Accent left:  #7c3aed
Accent right: #5b21b6
Neutral top:  #e2e8f0
Neutral left: #94a3b8
Neutral right:#64748b
```

### The Isometric Transform
```css
.iso-scene {
  transform: rotateX(60deg) rotateZ(-45deg);
  transform-style: preserve-3d;
}
/* Or use pure CSS clip-paths for flat isometric faces */
```

### Flat Isometric Cube (CSS)
```css
.iso-cube { position: relative; width: 60px; height: 60px; }
.iso-top {
  position: absolute;
  width: 60px; height: 60px;
  background: #7c9dff;
  transform: rotate(210deg) skewX(-30deg) scaleY(0.864);
}
.iso-left {
  position: absolute; bottom: 0; left: 0;
  width: 60px; height: 40px;
  background: #4a6fe8;
  transform: rotate(90deg) skewX(-30deg) scaleY(0.864) translateY(-50%);
}
.iso-right {
  position: absolute; bottom: 0; right: 0;
  width: 60px; height: 40px;
  background: #2a4db5;
  transform: skewX(-30deg) scaleY(0.864);
}
```

### Cards
Flat cards with an isometric-style top-left shadow that mimics 3D:
```css
.iso-card {
  background: white; border-radius: 12px;
  box-shadow: -6px 6px 0 #2a4db5, -4px 4px 0 #4a6fe8;
}
.iso-card:hover { transform: translate(3px, -3px); box-shadow: -9px 9px 0 #2a4db5, -6px 6px 0 #4a6fe8; }
```

### Buttons
```css
.iso-btn {
  background: #4a6fe8; color: white;
  border-radius: 8px;
  box-shadow: -4px 4px 0 #2a4db5;
  font-weight: 600;
}
.iso-btn:hover { transform: translate(2px, -2px); box-shadow: -6px 6px 0 #2a4db5; }
.iso-btn:active { transform: translate(-2px, 2px); box-shadow: -2px 2px 0 #2a4db5; }
```

### Grid Pattern Background
```css
body::before {
  content: ''; position: fixed; inset: 0; pointer-events: none; opacity: 0.4;
  background-image: url("isometric-grid.svg");  /* SVG isometric grid */
  /* Or use CSS: rotated grid */
  background-image: linear-gradient(60deg, #c7d2fe 1px, transparent 1px),
                    linear-gradient(120deg, #c7d2fe 1px, transparent 1px),
                    linear-gradient(to right, #c7d2fe 1px, transparent 1px);
  background-size: 40px 40px;
}
```

### Implementation
- Google Fonts: IBM Plex Sans
- Illustrations are the centerpiece — build 2–3 isometric feature illustrations
- `overflow: hidden` on sections containing iso scenes
- Neutral page background so the colored faces pop
- Card hover should slide in the iso shadow direction

---

## Groovy Style

Warm 70s design — muted oranges, mustard yellows, avocado greens, earthy browns, rounded bubble lettering, psychedelic wave patterns, and retro warmth. Use when asked for "groovy style".

### Typography
Righteous or Baloo 2 (Google Fonts) for headlines — round and bubbly. DM Sans for body. Headlines `font-display font-bold` with no tight tracking — let it breathe wide. Very rounded feel.

### Colors
```
Background:   #fdf6e3   (warm cream)
Orange:       #e8621a
Mustard:      #d4a017
Avocado:      #6b7c35
Brown:        #8b4513
Rust:         #b7410e
Cream:        #fdf6e3
Dark text:    #2d1b00
```

### Wavy / Organic Shapes
```css
/* Wavy divider between sections */
.wave-divider {
  background: url("wavy.svg") repeat-x;
  height: 40px; width: 100%;
}
/* Wavy border on cards */
.wavy-card {
  border-radius: 60% 40% 70% 30% / 40% 60% 40% 60%;
}
/* Or use SVG path borders */
```

### Pattern Backgrounds
```css
.groovy-bg {
  background-color: #fdf6e3;
  background-image: url("retro-pattern.svg");  /* circles/swirls repeat */
  /* CSS circles pattern: */
  background-image: radial-gradient(circle at 50% 50%, #e8621a22 0%, transparent 60%),
                    radial-gradient(circle at 20% 80%, #d4a01722 0%, transparent 50%);
}
```

### Cards
```css
.groovy-card {
  background: white;
  border: 3px solid #2d1b00;
  border-radius: 40px 20px 40px 20px;  /* irregular retro rounding */
  box-shadow: 6px 6px 0 #2d1b00;
  padding: 32px;
}
```

### Buttons
```css
.groovy-btn {
  background: #e8621a; color: white;
  border: 3px solid #2d1b00;
  border-radius: 100px;
  font-family: 'Righteous'; font-size: 16px;
  box-shadow: 5px 5px 0 #2d1b00;
  padding: 14px 36px;
}
.groovy-btn:hover { transform: translate(-2px, -2px); box-shadow: 7px 7px 0 #2d1b00; }
.groovy-btn-mustard { background: #d4a017; color: #2d1b00; }
```

### Retro Badge / Sticker
`border: 3px solid #2d1b00; border-radius: 50%; padding: 12px 16px; background: #d4a017; font-family: 'Righteous'; font-size: 13px; uppercase; transform: rotate(-8deg)`.

### Section Rotation
Alternate warm section backgrounds: cream → orange-tint (#fff3e0) → mustard-tint (#fffde7) → green-tint (#f1f8e9).

### Implementation
- Google Fonts: Righteous + DM Sans
- `antialiased`
- Decorative retro sun/star/circle SVGs scattered as accents
- `overflow-x: hidden`
- Card grids: `gap-8`, generous padding
- Avoid anything that reads "modern tech" — no system fonts, no flat blues

---

## Zine Style

DIY photocopied aesthetic — rough edges, cut-and-paste collage energy, misaligned text, hand-stamped labels, and raw underground publishing. Intentionally imperfect. Use when asked for "zine style".

### Typography
VT323 or Special Elite (Google Fonts) for headlines — typewriter/distressed feel. Courier New/monospace for body. Mix font sizes dramatically. Headlines can be oversized, italic, or slightly rotated. `font-bold uppercase` but with intentional misalignment.

### Colors
```
Paper:         #f0ebe0   (aged newsprint)
Ink:           #1a1008   (warm near-black)
Red stamp:     #cc1100   (rubber stamp red)
Blue stamp:    #003388
Yellow hi:     #ffd700   (highlighter yellow)
Photocopy gray:#888880
```
Or go full black/white only for maximum xerox authenticity.

### Paper Texture
```css
body {
  background: #f0ebe0;
  background-image: url("paper-noise.svg");  /* grain texture */
  /* CSS approximation: */
  background-image: repeating-linear-gradient(
    0deg, transparent, transparent 2px, rgba(0,0,0,0.015) 2px, rgba(0,0,0,0.015) 4px
  );
}
```

### Cut-and-Paste Cards
Cards look like cut paper pieces layered at angles:
```css
.zine-card {
  background: white;
  border: 2px solid #1a1008;
  transform: rotate(-1.5deg);  /* each card slightly different angle */
  box-shadow: 3px 3px 0 #1a1008;
  padding: 20px;
  position: relative;
}
.zine-card:nth-child(2) { transform: rotate(1deg); }
.zine-card:nth-child(3) { transform: rotate(-0.5deg); }
```

### Stamp / Sticker Labels
```css
.stamp {
  display: inline-block;
  border: 3px solid #cc1100;
  color: #cc1100;
  font-family: monospace; font-weight: 700; uppercase;
  padding: 4px 10px;
  transform: rotate(-3deg);
  opacity: 0.85;
  /* Rough edge via filter: */
  filter: contrast(1.3);
}
.stamp-filled { background: #cc1100; color: white; }
```

### Highlighter Effect
```css
.highlight {
  background: linear-gradient(transparent 40%, #ffd700 40%, #ffd700 85%, transparent 85%);
  padding: 0 4px;
}
```

### Tape Strips
```css
.tape {
  background: rgba(255,230,150,0.6);
  border-top: 1px solid rgba(200,180,100,0.3);
  border-bottom: 1px solid rgba(200,180,100,0.3);
  height: 24px; width: 60px;
  transform: rotate(-3deg);
  position: absolute; top: -12px; left: 50%; transform: translateX(-50%) rotate(-3deg);
}
```

### Buttons
```css
.zine-btn {
  background: #1a1008; color: #f0ebe0;
  border: none; font-family: monospace; font-weight: 700; uppercase;
  padding: 12px 28px;
  clip-path: polygon(2px 0, 100% 0, calc(100% - 2px) 100%, 0 100%);  /* slightly trapezoid */
}
.zine-btn:hover { background: #cc1100; }
```

### Layout
Content intentionally NOT perfectly aligned. Slight rotations, varying card sizes, grid-breaking elements. Headers can overlap visually. Use `position: relative` with small offsets.

### Implementation
- Google Fonts: Special Elite + VT323
- Embrace imperfection — intentional misalignment is the style
- `filter: contrast(1.1) brightness(0.98)` on images for photocopy feel
- `mix-blend-mode: multiply` on overlapping elements

---

## Sci-Fi HUD Style

Heads-up display aesthetic — transparent panels, corner targeting brackets, scanning animations, data readouts, and a sense of augmented reality overlaid on reality. Use when asked for "sci-fi hud style".

### Typography
Orbitron (Google Fonts, weights 400–700) for all display and labels. Share Tech Mono for data readouts. `uppercase` everywhere. `tracking-[0.15em]`. Labels often include `::` or `//` prefixes.

### Colors
```
Background:    #000a12   (near-black with deep blue tint)
Primary:       #00e5ff   (electric cyan)
Secondary:     #00ff9d   (green)
Warning:       #ffcc00
Danger:        #ff3366
Surface:       rgba(0,229,255,0.05)
Border:        rgba(0,229,255,0.3)
```

### Corner Bracket UI
```css
.hud-panel { position: relative; border: 1px solid rgba(0,229,255,0.2); background: rgba(0,229,255,0.03); }
.hud-panel::before, .hud-panel::after,
.hud-panel > .corner-tl, .hud-panel > .corner-br {
  content: ''; position: absolute; width: 14px; height: 14px;
  border-color: #00e5ff; border-style: solid;
}
.hud-panel::before { top: -1px; left: -1px; border-width: 2px 0 0 2px; }
.hud-panel::after  { bottom: -1px; right: -1px; border-width: 0 2px 2px 0; }
```

### Scan Line Animation
```css
@keyframes scan {
  0% { transform: translateY(-100%); }
  100% { transform: translateY(100vh); }
}
.scan-line {
  position: fixed; left: 0; right: 0; height: 2px;
  background: linear-gradient(transparent, rgba(0,229,255,0.4), transparent);
  animation: scan 4s linear infinite;
  pointer-events: none; z-index: 9999;
}
```

### Data Readout Text
```css
.data-label {
  font-family: 'Share Tech Mono'; font-size: 10px;
  color: rgba(0,229,255,0.6); uppercase; letter-spacing: 0.2em;
}
.data-value { font-family: 'Orbitron'; color: #00e5ff; font-size: 24px; }
```
Pattern: small label above, large cyan number below.

### Targeting Reticle
```css
.reticle {
  width: 60px; height: 60px; position: relative;
  border: 1px solid rgba(0,229,255,0.3);
  border-radius: 50%;
}
.reticle::before { /* crosshairs */ }
```

### Buttons
```css
.hud-btn {
  background: transparent; color: #00e5ff;
  border: 1px solid rgba(0,229,255,0.5);
  font-family: 'Orbitron'; uppercase; letter-spacing: 0.15em; font-size: 11px;
  padding: 12px 28px; clip-path: polygon(8px 0, 100% 0, calc(100% - 8px) 100%, 0 100%);
  box-shadow: 0 0 12px rgba(0,229,255,0.2), inset 0 0 12px rgba(0,229,255,0.05);
}
.hud-btn:hover { background: rgba(0,229,255,0.1); box-shadow: 0 0 24px rgba(0,229,255,0.5); }
```

### Progress/Health Bars
```css
.hud-bar-track { background: rgba(0,229,255,0.1); border: 1px solid rgba(0,229,255,0.3); height: 6px; }
.hud-bar-fill { background: linear-gradient(to right, #00ff9d, #00e5ff); height: 100%; }
```

### Implementation
- Google Fonts: Orbitron + Share Tech Mono
- Scan line as `fixed` element always present
- `overflow: hidden` on sections with corner brackets
- `selection:bg-[#00e5ff] selection:text-black`
- Custom scrollbar: dark track, cyan thin thumb
- All text in cyan family — no white unless for strong contrast

---

## Pixel Style

8-bit retro game aesthetic — pixelated fonts, chunky bordered UI panels, sprite-like icons, and classic game interface elements. Nostalgic and playful. Use when asked for "pixel style".

### Typography
Press Start 2P (Google Fonts) — the quintessential pixel font. Use sparingly at small sizes (it reads poorly above 24px in paragraph form). Body text at `text-xs leading-loose` because the font is dense. Headlines at `text-2xl md:text-4xl`.

### Colors
Classic game palettes — choose one:
```
NES palette:    #0f0f0f, #fcfcfc, #f83800, #0070ec, #fbbc00
Game Boy:       #0f380f, #306230, #8bac0f, #9bbc0f  (4 greens only)
CGA palette:    #000000, #ffffff, #ff5555, #55ffff   (high contrast)
Modern pixel:   #1a1c2c, #5d275d, #b13e53, #ef7d57, #ffcd75, #a7f070, #38b764
```

### Pixel Border
```css
/* CSS pixel border using box-shadow */
.pixel-border {
  box-shadow:
    0 -4px 0 0 #000,   /* top */
    0 4px 0 0 #000,    /* bottom */
    -4px 0 0 0 #000,   /* left */
    4px 0 0 0 #000;    /* right */
  /* Or use image-rendering: pixelated on border images */
}
/* Panel style */
.pixel-panel {
  border: 4px solid #000;
  outline: 4px solid #fff;
  outline-offset: -8px;
  background: #1a1c2c;
  image-rendering: pixelated;
}
```

### Buttons
```css
.pixel-btn {
  background: #38b764; color: #000;
  border: 4px solid #000;
  font-family: 'Press Start 2P'; font-size: 10px; uppercase;
  padding: 12px 20px;
  box-shadow: 4px 4px 0 #000;
  image-rendering: pixelated;
}
.pixel-btn:hover { transform: translate(-2px, -2px); box-shadow: 6px 6px 0 #000; }
.pixel-btn:active { transform: translate(2px, 2px); box-shadow: 2px 2px 0 #000; }
/* Health bar style CTA */
.pixel-btn-red { background: #b13e53; color: white; }
```

### Health/Progress Bar
```css
.pixel-bar {
  border: 4px solid #000; background: #1a1c2c;
  height: 20px; padding: 2px;
  image-rendering: pixelated;
}
.pixel-bar-fill {
  height: 100%; background: #38b764;
  image-rendering: pixelated;
  /* Animate: from 0 to value */
}
```

### Dialogue Box
```css
.pixel-dialogue {
  border: 4px solid #fff; outline: 4px solid #000; outline-offset: -8px;
  background: #1a1c2c; color: #fff;
  font-family: 'Press Start 2P'; font-size: 10px; line-height: 2;
  padding: 20px;
}
.pixel-dialogue::after { content: '▼'; animation: blink 1s step-end infinite; }
```

### Stars/Score Display
`★★★★☆` in pixel font. XP/points in large pixel numbers.

### Implementation
- Google Fonts: Press Start 2P only
- `image-rendering: pixelated` on all game elements
- `cursor: default` or a custom pixel cursor
- Background: dark with pixel grid `background-size: 4px 4px` at 1px lines
- Avoid anti-aliasing: `font-smooth: never`
- `letter-spacing: 0.1em` on Press Start 2P to aid readability

---

## Scandinavian Style

Extreme restraint — cold whites, functional typography, generous negative space, subtle warm accents. Hygge meets modernism. Nothing unnecessary. Use when asked for "scandinavian style".

### Typography
Sora or Figtree (Google Fonts, weights 300–600). Never heavy. `font-light` for body, `font-medium` for headings max. Lowercase preferred for headings — no uppercase aggression. Generous `line-height: 1.8`. `tracking-normal` — no wide spacing.

### Colors
```
Background:  #f9f9f7   (barely-warm white)
Surface:     #f2f1ef
Border:      #e5e3df
Text:        #1c1c1a
Muted:       #8a8880
Warm accent: #c4854a   (terracotta — used very sparingly)
Cold accent: #4a7fa5   (slate blue — alternative)
```
One accent color only. Mostly neutrals.

### The Rule of Restraint
- Maximum 2 font weights
- Maximum 1 accent color
- Maximum 3 elements per card
- Whitespace is structural, not decorative

### Cards
```css
.scandi-card {
  background: #f2f1ef;
  border: 1px solid #e5e3df;
  border-radius: 4px;
  padding: 32px;
}
/* No shadow. No hover lift. Just the border lightening slightly. */
.scandi-card:hover { border-color: #c4854a; }
```

### Buttons
```css
.scandi-btn {
  background: #1c1c1a; color: #f9f9f7;
  border: none; border-radius: 2px;
  font-weight: 400; letter-spacing: 0.05em; font-size: 14px;
  padding: 12px 32px;
}
.scandi-btn:hover { background: #c4854a; }
.scandi-btn-ghost { background: transparent; color: #1c1c1a; border: 1px solid #e5e3df; }
.scandi-btn-ghost:hover { border-color: #1c1c1a; }
```

### Dividers
`<hr>` only. `border-top: 1px solid #e5e3df`. Never decorative dividers.

### Nav
Bare. Product name left in `font-light`. Links `text-sm font-light text-muted hover:text-dark`. CTA: `bg-dark text-white text-sm px-5 py-2`. No backdrop blur, no shadow — just a `border-bottom: 1px solid #e5e3df`.

### Whitespace as Design
- Hero: `pt-40 pb-32` minimum
- Between sections: `py-28`
- Between heading and body: `mt-8`
- Cards in a grid: `gap-10` (extra wide gap)

### Accent Usage
The single terracotta or slate blue accent appears on:
- One CTA button
- Hover states on links/cards
- One highlighted stat number
- Icon fill on 1–2 icons
Never as a background, never on text blocks.

### Implementation
- Google Fonts: Sora (weights 300, 400, 500 only)
- `antialiased`
- `max-w-4xl mx-auto` — narrow, intimate
- No Font Awesome — use inline SVGs or Unicode
- Image treatment: `filter: saturate(0.7) contrast(1.05)` for cool Scandinavian photo feel

---

## Gothic Style

Deep forest greens and near-blacks, ornate serif typography, candle-wax drip textures, wrought-iron motifs, and a sense of ancient dark romance. Use when asked for "gothic style".

### Typography
IM Fell English or Cinzel (Google Fonts) for display — classical, engraved. Crimson Text for body — readable serif with old character. Headlines `font-serif uppercase tracking-[0.1em]`. Body `font-serif leading-relaxed`.

### Colors
```
Background:   #0a0c0a   (near-black with green tint)
Surface:      #111811
Dark green:   #1a2e1a
Forest:       #2d4a2d
Gold:         #b8962e
Blood red:    #8b1a1a
Bone white:   #e8e4d8
Muted:        #6b7a6b
```

### Background Texture
```css
body {
  background: #0a0c0a;
  background-image:
    radial-gradient(ellipse at 50% 0%, rgba(45,74,45,0.4) 0%, transparent 70%);
}
/* Optional: SVG noise grain for parchment/stone texture */
```

### Cards / Panels
```css
.gothic-panel {
  background: #111811;
  border: 1px solid rgba(184,150,46,0.3);
  position: relative;
}
/* Ornamental corner flourish using ::before/::after */
.gothic-panel::before {
  content: '✦'; position: absolute; top: 8px; left: 12px;
  color: rgba(184,150,46,0.4); font-size: 14px;
}
```

### Ornamental Dividers
```html
<!-- Between sections -->
<div class="flex items-center gap-3 my-12">
  <div class="flex-1 h-px" style="background: linear-gradient(to right, transparent, #b8962e)"></div>
  <span class="text-gold font-serif text-xl">✦</span>
  <div class="flex-1 h-px" style="background: linear-gradient(to left, transparent, #b8962e)"></div>
</div>
```

### Drip / Candle Wax Effect
```css
.wax-drip {
  position: relative;
  border-top: 4px solid #8b1a1a;
}
.wax-drip::after {
  content: '';
  position: absolute; top: -4px; left: 20%;
  width: 12px; height: 24px;
  background: #8b1a1a;
  clip-path: polygon(0 0, 100% 0, 80% 100%, 20% 100%);  /* drip shape */
  border-radius: 0 0 50% 50%;
}
```

### Buttons
```css
.gothic-btn {
  background: transparent; color: #b8962e;
  border: 1px solid rgba(184,150,46,0.5);
  font-family: 'Cinzel'; uppercase; letter-spacing: 0.2em; font-size: 11px;
  padding: 14px 40px;
}
.gothic-btn:hover { background: rgba(184,150,46,0.1); border-color: #b8962e; }
.gothic-btn-filled { background: #8b1a1a; color: #e8e4d8; border-color: #8b1a1a; }
```

### Nav
Dark with thin gold bottom border. Product name in Cinzel. Links `font-serif text-sm text-muted hover:text-bone`.

### Implementation
- Google Fonts: Cinzel + Crimson Text
- `antialiased`
- `selection:bg-[#8b1a1a] selection:text-[#e8e4d8]`
- Font Awesome: `fa-crow`, `fa-moon`, `fa-skull` for thematic icons
- Section backgrounds: alternate between `#0a0c0a` and `#111811`
- Gold used only for ornaments and borders — never as fill

---

## Handwritten Style

Sketch-like hand-drawn borders, pencil textures, slightly imperfect linework, and a warm personal studio feel. Like a designer's sketchbook brought to life. Use when asked for "handwritten style".

### Typography
Caveat or Kalam (Google Fonts, weights 400–700) for headlines and labels — authentic handwriting feel. Lato or Open Sans (light) for body — readable contrast to the expressive headings.

### Colors
```
Paper:         #fdfaf4   (warm sketch paper)
Pencil:        #2c2c2c   (near-black, not pure black)
Blue pen:      #2255aa
Red markup:    #cc3322
Yellow hi:     #f5e642
Muted:         #888880
```

### Sketchy Border Effect
Use SVG filters for a hand-drawn wobble on borders:
```css
.sketchy {
  filter: url(#sketchy);
  border-radius: 2px 8px 4px 6px / 6px 2px 8px 4px;  /* irregular */
}
/* SVG filter in page (hidden): */
/* <svg style="display:none"><filter id="sketchy">
  <feTurbulence type="turbulence" baseFrequency="0.05" numOctaves="2" result="noise"/>
  <feDisplacementMap in="SourceGraphic" in2="noise" scale="3" xChannelSelector="R" yChannelSelector="G"/>
</filter></svg> */
```

### Hand-Drawn Cards
```css
.sketch-card {
  background: white;
  border: 2px solid #2c2c2c;
  border-radius: 3px 10px 5px 8px / 8px 3px 10px 5px;  /* imperfect rounding */
  box-shadow: 3px 4px 0 #ddd, 4px 5px 0 #ccc;  /* pencil shadow */
  padding: 24px;
  position: relative;
}
/* Add a slight paper texture via ::before */
```

### Underline Sketchy Highlight
```css
.sketch-underline {
  text-decoration: none;
  background: linear-gradient(to bottom, transparent 60%, #f5e642 60%);
  padding-bottom: 2px;
}
```

### Annotation / Arrow Labels
```html
<!-- Pointing arrow annotation -->
<div class="annotation">
  <span class="font-hand text-sm text-blue-pen">← look here!</span>
</div>
```
Use `transform: rotate(-5deg)` on annotation divs.

### Buttons
```css
.sketch-btn {
  background: white; color: #2c2c2c;
  border: 2px solid #2c2c2c;
  border-radius: 4px 10px 8px 6px / 10px 4px 6px 8px;  /* irregular */
  font-family: 'Caveat'; font-weight: 700; font-size: 18px;
  padding: 10px 28px;
  box-shadow: 3px 3px 0 #2c2c2c;
}
.sketch-btn:hover { transform: rotate(-1deg); box-shadow: 5px 5px 0 #2c2c2c; }
.sketch-btn-filled { background: #2c2c2c; color: white; }
```

### Tape / Pin Accents
Small decorative elements holding cards: colored tape strips (`.tape { background: rgba(255,220,100,0.7); transform: rotate(-3deg); height: 18px; width: 48px; }`).

### Implementation
- Google Fonts: Caveat + Lato
- Apply SVG displacement filter for authentic sketch wobble
- `antialiased` off (slightly) — `font-smooth: never` for more raw feel
- Subtle paper background texture on body
- Grid lines can be sketchy: `opacity: 0.3`

---

## Aurora Style

Flowing silk-like color gradients — aurora borealis colors blending softly. Light, dreamy, and ethereal. Use when asked for "aurora style".

### Typography
Manrope or Cabinet Grotesk (Google Fonts, weights 200–700). Headlines `font-bold tracking-tight`. Body `font-light leading-relaxed`. The gradients do the heavy lifting — type stays elegant and clean.

### Colors
No fixed palette — always multiple blending gradients. Example aurora set:
```
Green-teal:  #00f5a0
Blue-purple: #7b2ff7
Pink:        #f72585
Gold:        #f5a623
Cyan:        #00d4ff
```
Background is always a flowing gradient — never flat.

### Gradient Background
```css
body {
  background: linear-gradient(135deg, #0d1117 0%, #1a0533 30%, #0a1628 60%, #001a12 100%);
  min-height: 100vh;
}
/* Animated aurora blobs */
.aurora-blob {
  position: fixed; border-radius: 50%; filter: blur(80px); opacity: 0.6;
  animation: drift 8s ease-in-out infinite alternate;
}
.aurora-1 { width: 600px; height: 400px; background: #7b2ff7; top: -100px; left: -100px; }
.aurora-2 { width: 500px; height: 300px; background: #00f5a0; bottom: 0; right: -50px; animation-delay: 2s; }
.aurora-3 { width: 400px; height: 400px; background: #f72585; top: 40%; left: 30%; animation-delay: 4s; opacity: 0.3; }

@keyframes drift {
  from { transform: translate(0, 0) scale(1); }
  to   { transform: translate(40px, 30px) scale(1.1); }
}
```

### Glass Cards
Same as Glassmorphism but with aurora-tinted borders:
```css
.aurora-card {
  background: rgba(255,255,255,0.06);
  backdrop-filter: blur(24px);
  border: 1px solid rgba(255,255,255,0.12);
  border-radius: 20px;
  box-shadow: 0 8px 32px rgba(0,0,0,0.3);
}
```

### Gradient Text
```css
.aurora-text {
  background: linear-gradient(90deg, #00f5a0, #7b2ff7, #f72585, #00d4ff);
  background-size: 200% auto;
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  animation: aurora-shift 4s linear infinite;
}
@keyframes aurora-shift { to { background-position: 200% center; } }
```

### Gradient Buttons
```css
.aurora-btn {
  background: linear-gradient(135deg, #7b2ff7, #f72585);
  color: white; border: none; border-radius: 100px;
  box-shadow: 0 4px 24px rgba(123,47,247,0.4);
  font-weight: 600; padding: 14px 32px;
}
.aurora-btn:hover { box-shadow: 0 8px 40px rgba(123,47,247,0.6); transform: translateY(-2px); }
.aurora-btn-outline {
  background: transparent;
  border: 1px solid rgba(255,255,255,0.2);
  color: white; border-radius: 100px;
  backdrop-filter: blur(10px);
}
```

### Divider Lines
```css
.aurora-line {
  height: 1px;
  background: linear-gradient(to right, transparent, #7b2ff7, #00f5a0, #f72585, transparent);
  opacity: 0.5;
}
```

### Implementation
- Google Fonts: Manrope (weights 200, 400, 600, 700)
- All body text white or `white/80`
- Aurora blobs: `pointer-events: none; z-index: 0` — content at `z-index: 1`+
- `antialiased`
- `selection:bg-[#7b2ff7] selection:text-white`
- Avoid hard borders — let gradients define edges

---

## Tropical Style

Warm vacation energy — coral, turquoise, sandy whites, bold tropical colors, and a relaxed resort aesthetic. Fun, bright, and sun-soaked. Use when asked for "tropical style".

### Typography
Nunito or Quicksand (Google Fonts, weights 400–800). Rounded and friendly. Headlines `font-extrabold tracking-tight`. Body `font-medium leading-relaxed`.

### Colors
```
Background:   #fffef9   (warm white)
Coral:        #ff6b5b
Turquoise:    #00c9b1
Sandy:        #f5dfa0
Lime:         #8bc34a
Ocean blue:   #1a85c8
Dark text:    #1a2332
```

### Section Backgrounds
Rotate through warm section backgrounds:
- `#fff8f0` (warm cream)
- `#e8faf8` (light turquoise tint)
- `#fff3e0` (warm orange tint)
- `#f0f9ff` (light sky blue)

### Cards
```css
.tropical-card {
  background: white;
  border-radius: 24px;
  box-shadow: 0 8px 32px rgba(26,133,200,0.12), 0 2px 8px rgba(0,0,0,0.06);
  overflow: hidden;
}
/* Colored top bar per card, cycling through palette */
.tropical-card::before {
  content: ''; display: block; height: 6px;
  background: linear-gradient(90deg, #ff6b5b, #f5dfa0);
}
.tropical-card:hover { transform: translateY(-6px); box-shadow: 0 16px 48px rgba(26,133,200,0.2); }
```

### Buttons
```css
.trop-btn {
  background: #ff6b5b; color: white;
  border-radius: 100px; border: none;
  font-weight: 700;
  box-shadow: 0 4px 16px rgba(255,107,91,0.4);
  padding: 14px 32px;
}
.trop-btn:hover { background: #ff5543; transform: translateY(-2px); box-shadow: 0 8px 24px rgba(255,107,91,0.5); }
.trop-btn-teal { background: #00c9b1; box-shadow: 0 4px 16px rgba(0,201,177,0.4); }
.trop-btn-outline { background: transparent; border: 2px solid #ff6b5b; color: #ff6b5b; border-radius: 100px; }
```

### Decorative Elements
- Palm leaf SVG accents in corners (simple, flat-color)
- Wave SVG dividers between sections
- Sun/circle decorations: `border-radius: 50%; background: #f5dfa0; opacity: 0.3`

### Nav
`background: white; box-shadow: 0 2px 20px rgba(0,0,0,0.06)`. Logo with a coral or teal accent. Links clean `text-dark hover:text-coral`. Coral rounded CTA.

### Stat Chips
`border-radius: 100px; background: [color-tint]; border: 2px solid [color]; padding: 8px 16px`. Each stat a different tropical color tint.

### Implementation
- Google Fonts: Nunito (weights 400, 600, 700, 800)
- `antialiased`
- `max-w-6xl mx-auto px-6`
- Section padding: `py-20`
- Emoji accents welcome: 🌴🌊☀️🐠 used minimally as decorative icons

---

## Grunge Style

Distressed textures, worn edges, splatter marks, rough typography, and a raw underground aesthetic. Feels printed, aged, and weathered. Use when asked for "grunge style".

### Typography
Bebas Neue (Google Fonts) for display — tall, condensed, bold. Courier New/monospace for body text — typewriter worn. Headlines `uppercase tracking-widest`. Mix sizes aggressively.

### Colors
```
Paper:        #d4c9b0   (aged, yellowed)
Ink:          #1a1208   (warm near-black)
Rust:         #8b3a1a
Olive:        #4a5c2a
Cream:        #f0e8d8
Blood:        #6b0f0f
Distressed:   #888070
```

### Texture Overlays
```css
/* Grain texture */
body::before {
  content: ''; position: fixed; inset: 0; pointer-events: none; z-index: 9999;
  background-image: url("grain.svg");
  opacity: 0.25; mix-blend-mode: multiply;
}
/* Aged paper background */
body {
  background: #d4c9b0;
  background-image:
    url("paper-texture.png"),
    radial-gradient(ellipse at 30% 20%, rgba(139,58,26,0.08) 0%, transparent 60%);
}
```

### Distressed Borders
```css
.grunge-border {
  border: 3px solid #1a1208;
  /* Rough edge via filter displacement */
  filter: url(#roughen);
  /* Or: use box-shadow for torn look */
  box-shadow: 2px 2px 0 #888070, 4px 4px 0 #666050;
}
```

### Splatter Element
```css
/* Ink splatter as ::before positioned absolutely */
.splatter::before {
  content: ''; position: absolute;
  background: #1a1208;
  clip-path: polygon(50% 0%, 62% 20%, 90% 10%, 75% 35%, 100% 50%, 75% 55%, 85% 85%, 60% 70%, 55% 100%, 40% 75%, 15% 90%, 35% 60%, 0 50%, 30% 40%, 10% 15%, 40% 25%);
  width: 40px; height: 40px; opacity: 0.15;
}
```

### Cards
```css
.grunge-card {
  background: #f0e8d8;
  border: 2px solid #1a1208;
  box-shadow: 4px 4px 0 #888070;
  padding: 24px;
  position: relative;
  transform: rotate(-0.5deg);  /* slight lean */
}
```

### Buttons
```css
.grunge-btn {
  background: #1a1208; color: #d4c9b0;
  border: 2px solid #1a1208;
  font-family: 'Bebas Neue'; font-size: 18px; uppercase; letter-spacing: 0.1em;
  padding: 12px 32px;
  box-shadow: 3px 3px 0 #888070;
}
.grunge-btn:hover { background: #8b3a1a; transform: translate(-1px, -1px); box-shadow: 5px 5px 0 #666050; }
.grunge-btn-outline { background: transparent; color: #1a1208; border: 2px solid #1a1208; }
```

### Stamp Marks
`background: [color]; color: white; font-weight: 900; uppercase; letter-spacing: 0.2em; padding: 4px 12px; transform: rotate(-8deg); opacity: 0.7; filter: contrast(1.5)` — applied to "SOLD", "FEATURED", "NEW" labels.

### Implementation
- Google Fonts: Bebas Neue
- Apply SVG displacement filter for roughened edges where possible
- `mix-blend-mode: multiply` on texture overlays
- Images: `filter: grayscale(30%) contrast(1.2) sepia(20%)` for aged photo look
- `overflow-x: hidden`

---

## Y2K Style

Early internet nostalgia — Windows 95/98 beveled UI, system gray, pixel-perfect chunky buttons, dialog boxes, taskbars, and everything that felt futuristic in 1999. Use when asked for "y2k style".

### Typography
System UI stack (mimics MS Sans Serif). Or load VT323 for more retro flavor. Headlines in `font-bold text-sm uppercase tracking-widest`. Everything feels like a dialog box label.

### Colors
```
Window bg:    #c0c0c0   (classic Windows gray)
Button face:  #d4d0c8
Title bar:    linear-gradient(to right, #000080, #1084d0)  (blue gradient)
Title text:   #ffffff
Sunken:       #808080 (dark edge) / #ffffff (light edge)  /* inset bevel */
Raised:       #ffffff (top/left) / #808080 (bottom/right) /* outset bevel */
Desktop teal: #008080
Text:         #000000
```

### The Bevel System
```css
/* Raised button / panel */
.win-raised {
  background: #d4d0c8;
  border-top: 2px solid #ffffff;
  border-left: 2px solid #ffffff;
  border-right: 2px solid #808080;
  border-bottom: 2px solid #808080;
}
/* Sunken / pressed / input */
.win-sunken {
  background: #ffffff;
  border-top: 2px solid #808080;
  border-left: 2px solid #808080;
  border-right: 2px solid #ffffff;
  border-bottom: 2px solid #ffffff;
}
/* Outer bevel (window frame) */
.win-window {
  border: 2px solid #000;
  outline: 2px solid #dfdfdf;
}
```

### Title Bar
```css
.win-titlebar {
  background: linear-gradient(to right, #000080, #1084d0);
  color: white; font-weight: 700; font-size: 12px;
  padding: 4px 6px;
  display: flex; align-items: center; justify-content: space-between;
  user-select: none;
}
.win-titlebar-btn {
  width: 16px; height: 14px;
  background: #d4d0c8;
  border-top: 1px solid #fff; border-left: 1px solid #fff;
  border-right: 1px solid #808080; border-bottom: 1px solid #808080;
  font-size: 10px; display: flex; align-items: center; justify-content: center;
}
```

### Window / Dialog
```css
.win-dialog {
  background: #d4d0c8;
  border: 2px solid #dfdfdf;
  outline: 2px solid #000;
  padding: 0;
  /* Contains: .win-titlebar + content area */
}
```

### Buttons
```css
.win-btn {
  background: #d4d0c8;
  border-top: 2px solid #fff; border-left: 2px solid #fff;
  border-right: 2px solid #808080; border-bottom: 2px solid #808080;
  padding: 4px 16px; font-size: 12px; font-weight: 400;
  min-width: 80px;
}
.win-btn:active {
  border-top: 2px solid #808080; border-left: 2px solid #808080;
  border-right: 2px solid #fff; border-bottom: 2px solid #fff;
}
.win-btn-default { outline: 2px solid #000; }  /* default button gets outer border */
```

### Progress Bar
`win-sunken` container, inner fill in royal blue `#000080`, chunky, no border-radius.

### Desktop Taskbar
`height: 32px; background: #d4d0c8; border-top: 2px solid #fff` — start button, open windows, clock.

### Start Button
`font-bold; background: #d4d0c8` with Windows logo and "Start" text. Raised bevel.

### Implementation
- No Google Fonts — system-ui stack only
- `cursor: default` — use system cursor
- All interactive elements need raised/sunken bevel state
- `font-size: 12px` for labels — small, system-like
- Page background: `#008080` (teal) for full desktop effect, or `#c0c0c0` for clean

---

## Kawaii Style

Super cute pastel design — bubble-soft rounding, cheerful character mascot accents, candy colors, bouncy animations, and irresistible friendliness. Use when asked for "kawaii style".

### Typography
Nunito (Google Fonts, weight 700–900) — round and bouncy. Or Baloo 2. Headlines very rounded, large, cheerful. `font-extrabold`. Body `font-bold` — even body text feels punchy and expressive.

### Colors
```
Background:   #fff5f9   (blush white)
Pink:         #ffb3c6
Lavender:     #c9b8ff
Mint:         #b8f0d0
Yellow:       #ffe66d
Peach:        #ffcba4
Sky:          #bde8ff
Dark text:    #3d2c5e   (deep purple — friendlier than black)
```

### Kawaii Shapes
```css
/* Blob card shape */
.kawaii-card {
  background: white;
  border-radius: 50% 50% 50% 50% / 40% 40% 60% 60%;
  /* Or: extremely rounded rectangle */
  border-radius: 40px;
  border: 3px solid #3d2c5e;
  box-shadow: 5px 5px 0 #3d2c5e;
  padding: 28px;
}
/* Speech bubble */
.speech-bubble {
  border-radius: 24px;
  position: relative;
}
.speech-bubble::after {
  content: ''; position: absolute; bottom: -16px; left: 24px;
  border: 8px solid transparent;
  border-top-color: #ffb3c6;
}
```

### Bounce Animation
```css
@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-8px); }
}
.bounce { animation: bounce 2s ease-in-out infinite; }
@keyframes wiggle {
  0%,100% { transform: rotate(-3deg); }
  50% { transform: rotate(3deg); }
}
.wiggle { animation: wiggle 1s ease-in-out infinite; }
```

### Buttons
```css
.kawaii-btn {
  background: #ffb3c6; color: #3d2c5e;
  border: 3px solid #3d2c5e;
  border-radius: 100px;
  font-weight: 800; font-size: 16px;
  box-shadow: 4px 4px 0 #3d2c5e;
  padding: 12px 28px;
}
.kawaii-btn:hover { transform: translate(-2px, -2px) scale(1.05); box-shadow: 6px 6px 0 #3d2c5e; }
.kawaii-btn:active { transform: translate(2px, 2px); box-shadow: 2px 2px 0 #3d2c5e; }
/* Each button gets a different candy color */
```

### Character/Mascot Accents
Simple emoji or flat SVG character faces used decoratively: `text-4xl` positioned absolutely. Stars and sparkles: `✦ ★ ✿` in candy colors.

### Pill Tags
`border-radius: 100px; border: 2px solid #3d2c5e; background: [pastel]; font-weight: 800; padding: 4px 14px` — each a different pastel.

### Section Backgrounds
Rotate through candy-colored sections: pink → lavender → mint → yellow → peach → sky.

### Star Rating
`★★★★☆` in large yellow, with `font-size: 24px` — cheerful and prominent.

### Implementation
- Google Fonts: Nunito (700, 800, 900)
- `antialiased`
- Character accents: large emoji or flat SVG faces as decorative elements
- Card grids: `gap-6`, `grid-cols-2 md:grid-cols-3`
- Bounce/wiggle animations on hero elements and mascots
- Never use sharp corners — minimum `border-radius: 20px`

---

## Manga Style

Bold ink outlines, dynamic speed lines, dramatic panel composition, halftone dot patterns, and high-contrast black/white with occasional color pop. Use when asked for "manga style".

### Typography
Bangers (Google Fonts) for display and sound effects — comic book energy. Nunito or Bold system-sans for body. Headlines `font-display uppercase tracking-wide`. Sound effects at oversized sizes and dramatic angles.

### Colors
```
White:       #ffffff
Black:       #0a0a0a
Halftone:    #f0f0f0
Red pop:     #e8001a   (the ONE accent color — used for critical moments)
Yellow:      #ffe600   (highlight — secondary)
```
90% black and white. Red used for the single most important element per section.

### Speed Lines
```css
.speed-lines {
  background: repeating-conic-gradient(
    from 0deg at 50% 50%,
    transparent 0deg, transparent 3deg,
    rgba(0,0,0,0.08) 3deg, rgba(0,0,0,0.08) 4deg
  );
}
```

### Halftone Pattern
```css
.halftone {
  background-image: radial-gradient(circle, #0a0a0a 1.5px, transparent 1.5px);
  background-size: 8px 8px;
  opacity: 0.1;
}
```

### Panel Cards
```css
.manga-panel {
  background: white;
  border: 4px solid #0a0a0a;
  position: relative;
  overflow: hidden;
}
/* Action panel — skewed border */
.manga-panel-action {
  border: 4px solid #0a0a0a;
  clip-path: polygon(0 0, 95% 0, 100% 100%, 5% 100%);  /* parallelogram */
}
```

### Sound Effect Text
```css
.sfx {
  font-family: 'Bangers'; font-size: clamp(60px, 15vw, 140px);
  color: #e8001a;
  -webkit-text-stroke: 3px #0a0a0a;
  text-shadow: 6px 6px 0 #0a0a0a;
  transform: rotate(-5deg);
  display: inline-block;
}
```

### Buttons
```css
.manga-btn {
  background: #0a0a0a; color: white;
  border: 3px solid #0a0a0a;
  font-family: 'Bangers'; font-size: 20px; uppercase; letter-spacing: 0.05em;
  padding: 10px 28px;
  clip-path: polygon(4px 0, 100% 0, calc(100% - 4px) 100%, 0 100%);
  box-shadow: 4px 4px 0 #888;
}
.manga-btn:hover { background: #e8001a; box-shadow: 6px 6px 0 #0a0a0a; }
.manga-btn-outline { background: transparent; color: #0a0a0a; border-color: #0a0a0a; }
```

### Thought Bubble
`border-radius: 50%; border: 3px solid #0a0a0a; padding: 20px; background: white; position: relative` — with smaller circles as the bubble trail below.

### Implementation
- Google Fonts: Bangers + Nunito
- `overflow: hidden` on sections with speed lines
- Panel borders are always thick `4px solid black`
- Images: `filter: contrast(1.3) grayscale(100%)` for manga photo treatment
- Red used ONCE per page for maximum impact

---

## Dashboard Style

Data-dense admin/analytics interface — sidebar navigation, metric cards, charts, tables, and a professional information-first aesthetic. Use when asked for "dashboard style".

### Typography
Inter (Google Fonts, weights 400–700). `font-medium` for headings. `font-normal text-sm` for body/labels. `font-mono` for numbers, IDs, and code values. `tracking-tight` on metric numbers.

### Colors
```
Background:    #f8fafc   (slate-50)
Sidebar bg:    #0f172a   (slate-900)
Surface:       #ffffff
Border:        #e2e8f0   (slate-200)
Text:          #1e293b   (slate-800)
Muted:         #64748b   (slate-500)
Primary:       #3b82f6   (blue-500)
Success:       #22c55e   (green-500)
Warning:       #f59e0b   (amber-500)
Danger:        #ef4444   (red-500)
```

### Layout
```css
.dashboard-layout {
  display: grid;
  grid-template-columns: 240px 1fr;
  min-height: 100vh;
}
.sidebar { background: #0f172a; padding: 20px 0; }
.main-content { background: #f8fafc; padding: 24px; overflow-y: auto; }
```

### Sidebar
Dark left panel with logo at top. Nav items: `px-4 py-2.5 rounded-lg mx-2 text-sm font-medium`. Active: `bg-blue-600 text-white`. Inactive: `text-slate-400 hover:bg-slate-800 hover:text-white`. Section labels: `text-[10px] uppercase tracking-widest text-slate-500 px-4 py-2 mt-4`.

### Metric Cards
```css
.metric-card {
  background: white; border: 1px solid #e2e8f0;
  border-radius: 12px; padding: 20px;
}
/* Top: icon right-aligned in colored circle */
/* Middle: large number font-bold text-3xl font-mono */
/* Bottom: delta indicator (green up / red down arrow + %) */
```

### Delta Indicators
```css
.delta-up   { color: #22c55e; background: #f0fdf4; padding: 2px 8px; border-radius: 100px; font-size: 12px; font-weight: 600; }
.delta-down { color: #ef4444; background: #fef2f2; }
```

### Data Table
```css
table { width: 100%; border-collapse: collapse; }
thead { background: #f8fafc; border-bottom: 2px solid #e2e8f0; }
th { text-align: left; font-size: 11px; font-weight: 600; uppercase; color: #64748b; letter-spacing: 0.05em; padding: 10px 16px; }
td { padding: 14px 16px; border-bottom: 1px solid #f1f5f9; font-size: 14px; }
tr:hover td { background: #f8fafc; }
```

### Status Badges
`border-radius: 100px; font-size: 11px; font-weight: 600; padding: 2px 10px`:
- Active: `bg-green-100 text-green-700`
- Pending: `bg-amber-100 text-amber-700`
- Error: `bg-red-100 text-red-700`
- Draft: `bg-slate-100 text-slate-600`

### Chart Placeholder
```css
.chart-container {
  background: white; border: 1px solid #e2e8f0; border-radius: 12px;
  padding: 20px;
  /* Header: title left, time-range pills right */
  /* Body: chart area (use CSS bars or SVG) */
}
.bar { background: #3b82f6; border-radius: 4px 4px 0 0; }
```

### Breadcrumb
`text-sm text-slate-400 font-medium` with `/` separator. Current page in `text-slate-900`.

### Implementation
- Google Fonts: Inter
- `antialiased`
- Sidebar fixed, main content scrollable
- Time-range pill group: `border border-slate-200 rounded-lg overflow-hidden` with `bg-blue-50 text-blue-600` for active pill
- Page header: title + subtitle + action buttons right-aligned

---

## Maximalist Style

More is more. Layered patterns on patterns, clashing colors that somehow work, dense content, ornate details at every level, and a sense of abundant visual richness. Use when asked for "maximalist style".

### Typography
Mix two contrasting display fonts deliberately: Bebas Neue (condensed, loud) + Abril Fatface (thick, ornate). Body: DM Sans. Headlines at multiple competing sizes — intentional hierarchy chaos. `font-display` for some, `font-serif` for others.

### Colors
No palette constraints. Use everything:
```
Start with: coral #ff6b6b, gold #ffd700, teal #00cec9,
purple #6c5ce7, lime #00b894, pink #fd79a8,
navy #2d3436, cream #ffeaa7
```
Layer multiple colors on the same section. Pattern on pattern. Contrast is welcome.

### Layered Background
```css
body {
  background:
    repeating-linear-gradient(45deg, rgba(255,215,0,0.1) 0, rgba(255,215,0,0.1) 2px, transparent 0, transparent 50%),
    repeating-linear-gradient(-45deg, rgba(108,92,231,0.1) 0, rgba(108,92,231,0.1) 2px, transparent 0, transparent 50%),
    #ffeaa7;
  background-size: 20px 20px, 20px 20px;
}
```

### Cards — Stacked and Layered
```css
.maxi-card {
  background: white;
  border: 4px solid #2d3436;
  border-radius: 0;  /* or vary: 0, 20px, 50%, mixing */
  box-shadow: 8px 8px 0 #ffd700, 16px 16px 0 #6c5ce7;
  position: relative;
}
/* Decorative stripes, dots, and patterns as ::before/::after */
```

### Overlapping Elements
Use `position: absolute` and `z-index` to create intentional overlaps — text over images, cards over other cards, decorative shapes bleeding between sections.

### Bold Section Backgrounds
Each section uses a dramatically different treatment:
- Section 1: hot coral `#ff6b6b` + stripe pattern
- Section 2: dark navy with gold dots
- Section 3: lime green + geometric shapes
- Section 4: cream + layered shadows

### Typography Mix
```css
/* Section number: massive, overlapping, background watermark */
.section-num {
  font-family: 'Bebas Neue'; font-size: 20vw; opacity: 0.08;
  position: absolute; top: -0.1em; left: -0.05em; color: #2d3436;
}
/* Main headline: Abril Fatface, large */
/* Sub: DM Sans, contrasting weight */
```

### Buttons — Maximalist
Multiple button styles used simultaneously on the same page:
```css
.btn-stack { background: #ff6b6b; color: white; border: 4px solid #2d3436; box-shadow: 6px 6px 0 #2d3436, 12px 12px 0 #ffd700; }
.btn-outlined { background: transparent; border: 4px solid currentColor; }
.btn-pill { border-radius: 100px; background: #6c5ce7; color: white; }
```

### Implementation
- Google Fonts: Bebas Neue + Abril Fatface + DM Sans
- `overflow-x: hidden`
- Decorative shapes: triangles, circles, stars all present simultaneously
- Images: full bleed, large, with color overlays
- No whitespace rules — density is the goal

---

## Corporate Style

Conservative, structured, and trustworthy. Navy blues, clean grids, professional typography, and a buttoned-up B2B feel that communicates reliability above all. Use when asked for "corporate style".

### Typography
Source Sans 3 or IBM Plex Sans (Google Fonts, weights 300–700). No display fonts. `font-semibold` for headings, `font-normal` for body. `tracking-tight` on headings. Everything readable and measured.

### Colors
```
Background:  #ffffff
Surface:     #f8f9fc
Border:      #e4e7eb
Text:        #111827
Muted:       #6b7280
Navy:        #1e3a5f
Blue:        #2563eb
Light blue:  #eff6ff
Green:       #16a34a   (success/positive)
```

### Page Structure
Clean 3-zone layout: sticky nav → hero → alternating content sections → CTA → footer. No surprises. Max-width `max-w-7xl`.

### Cards
```css
.corp-card {
  background: white; border: 1px solid #e4e7eb;
  border-radius: 8px; padding: 24px;
  box-shadow: 0 1px 3px rgba(0,0,0,0.06), 0 1px 2px rgba(0,0,0,0.04);
}
.corp-card:hover { box-shadow: 0 4px 12px rgba(0,0,0,0.1); transform: translateY(-2px); }
/* Blue left border on featured cards */
.corp-card-featured { border-left: 4px solid #2563eb; }
```

### Buttons
```css
.corp-btn {
  background: #2563eb; color: white;
  border-radius: 6px; border: none;
  font-weight: 600; font-size: 15px;
  padding: 12px 28px;
  box-shadow: 0 1px 2px rgba(37,99,235,0.3);
}
.corp-btn:hover { background: #1d4ed8; box-shadow: 0 4px 12px rgba(37,99,235,0.4); }
.corp-btn-outline { background: transparent; border: 1px solid #2563eb; color: #2563eb; border-radius: 6px; }
.corp-btn-navy { background: #1e3a5f; }
```

### Nav
`sticky top-0 bg-white border-b border-[#e4e7eb] shadow-sm`. Logo: navy text, clean. Links: `text-sm font-medium text-gray-600 hover:text-gray-900`. CTA: blue pill button. Professional and non-flashy.

### Feature Grid
3-column grid. Each card: small blue icon square `bg-blue-50 rounded-lg` + bold heading + muted body. No gimmicks — just clear value communication.

### Trust Signals
Client logos: `filter: grayscale(100%) opacity(0.5)` in a flex row. Testimonials: plain blockquote with name + title + company. Stats: large blue numbers, muted labels.

### CTA Section
Navy background `#1e3a5f`. White headline. Subtext in `text-blue-200`. White CTA button + blue outline secondary button.

### Implementation
- Google Fonts: IBM Plex Sans (300, 400, 600)
- `antialiased`
- `max-w-7xl mx-auto px-6`
- Section padding: `py-20`
- No playful elements — every choice communicates stability and professionalism
- Images: professional stock, desaturated slightly

---

## Psychedelic Style

Melting, swirling, mind-bending design. Organic shapes that warp and pulse, rainbow color cycling, distorted typography, and a sense of visual reality dissolving. Use when asked for "psychedelic style".

### Typography
Boogaloo or Lobster (Google Fonts) for headlines — fluid and rounded, almost liquid. Body text `font-sans font-medium`. Headlines can be curved using SVG `textPath`. Random slight rotations on elements.

### Colors
The full rainbow, cycling. Never static:
```
Red:     #ff0040
Orange:  #ff6600
Yellow:  #ffcc00
Green:   #00dd44
Cyan:    #00ddff
Blue:    #4400ff
Violet:  #cc00ff
Pink:    #ff0088
```
Gradients always cycle through 3+ colors.

### Animated Gradient Background
```css
body {
  background: linear-gradient(0deg, #ff0040, #ff6600, #ffcc00, #00dd44, #00ddff, #4400ff, #cc00ff, #ff0040);
  background-size: 100% 800%;
  animation: psyche 8s linear infinite;
}
@keyframes psyche {
  0%   { background-position: 0% 0%; }
  100% { background-position: 0% 100%; }
}
```

### Morphing Blob Shapes
```css
@keyframes morph {
  0%,100% { border-radius: 60% 40% 30% 70% / 60% 30% 70% 40%; }
  25%     { border-radius: 30% 60% 70% 40% / 50% 60% 30% 60%; }
  50%     { border-radius: 50% 60% 30% 60% / 30% 40% 70% 60%; }
  75%     { border-radius: 60% 40% 60% 30% / 60% 70% 30% 40%; }
}
.morphing-blob { animation: morph 8s ease-in-out infinite; }
```

### Wavy/Distorted Text
```css
@keyframes wave-text {
  0% { transform: skewX(0deg) skewY(0deg); }
  25% { transform: skewX(3deg) skewY(-1deg); }
  50% { transform: skewX(-2deg) skewY(2deg); }
  75% { transform: skewX(1deg) skewY(-2deg); }
  100% { transform: skewX(0deg) skewY(0deg); }
}
.wavy-text { animation: wave-text 4s ease-in-out infinite; display: inline-block; }
```

### Cards
```css
.psyche-card {
  background: rgba(255,255,255,0.15);
  backdrop-filter: blur(10px);
  border: 2px solid rgba(255,255,255,0.3);
  border-radius: 60% 40% 50% 30% / 40% 50% 60% 50%;  /* blob */
  animation: morph 10s ease-in-out infinite;
  padding: 32px;
}
```

### Buttons
```css
.psyche-btn {
  background: linear-gradient(135deg, #ff0088, #4400ff, #00ddff);
  background-size: 200% auto;
  animation: aurora-shift 3s linear infinite;
  color: white; border: none; border-radius: 100px;
  font-weight: 700; padding: 14px 32px;
}
```

### Rainbow Text
```css
.rainbow {
  background: linear-gradient(90deg, #ff0040, #ff6600, #ffcc00, #00dd44, #00ddff, #4400ff, #cc00ff);
  background-size: 200% auto;
  -webkit-background-clip: text; -webkit-text-fill-color: transparent;
  animation: aurora-shift 3s linear infinite;
}
```

### Implementation
- Google Fonts: Boogaloo + Open Sans
- `overflow: hidden` on body — blobs bleed edges
- All animations run indefinitely — the page breathes
- White text on animated background
- `selection:bg-white/30 selection:text-white`

---

## Athletic Style

Bold diagonal cuts, high-energy color blocks, condensed impact typography, and the visual language of sports branding. Powerful, direct, and physical. Use when asked for "athletic style".

### Typography
Barlow Condensed or Oswald (Google Fonts, weights 400–800). `font-bold uppercase tracking-wider`. All headlines in condensed caps. Body: Barlow (normal width). `font-medium leading-snug`.

### Colors
Choose a team-style dual palette:
```
Primary:    #e61e2b   (red — or customize to brand)
Secondary:  #f5e642   (gold/yellow)
Dark:       #1a1a1a
Light:      #f5f5f5
White:      #ffffff
```
Two accent colors max. Hard contrast. No pastels.

### Diagonal Cut Sections
```css
.diagonal-cut {
  clip-path: polygon(0 0, 100% 0, 100% 85%, 0 100%);
  /* or: */
  clip-path: polygon(0 5%, 100% 0, 100% 95%, 0 100%);
}
.diagonal-bg {
  position: relative;
}
.diagonal-bg::after {
  content: ''; position: absolute; bottom: -2px; left: 0; right: 0;
  height: 60px;
  background: [next-section-color];
  clip-path: polygon(0 100%, 100% 0, 100% 100%);
}
```

### Color Block Layout
Large full-width sections alternating: dark → light → primary color → dark. No gradients — flat solid color blocks, hard edges.

### Jersey Number / Big Stat
```css
.jersey-num {
  font-family: 'Barlow Condensed'; font-weight: 800;
  font-size: clamp(80px, 20vw, 200px);
  text-transform: uppercase; line-height: 1;
  color: rgba(255,255,255,0.08);  /* giant watermark */
  position: absolute;
}
```

### Cards
```css
.athletic-card {
  background: white;
  border-top: 4px solid #e61e2b;
  border-radius: 0;
  padding: 24px;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
}
.athletic-card:hover { transform: translateY(-4px); box-shadow: 0 12px 32px rgba(0,0,0,0.2); }
```

### Buttons
```css
.sport-btn {
  background: #e61e2b; color: white;
  border: none; border-radius: 0;
  font-family: 'Barlow Condensed'; font-weight: 700;
  uppercase; letter-spacing: 0.1em; font-size: 16px;
  padding: 14px 36px;
  clip-path: polygon(6px 0, 100% 0, calc(100% - 6px) 100%, 0 100%);
  box-shadow: 0 4px 16px rgba(230,30,43,0.4);
}
.sport-btn:hover { background: #cc1a26; box-shadow: 0 8px 24px rgba(230,30,43,0.5); }
.sport-btn-gold { background: #f5e642; color: #1a1a1a; }
```

### Marquee / Ticker
`background: #e61e2b; color: white; font-family: 'Barlow Condensed'; uppercase; letter-spacing: 0.15em`. Items separated by `//` or `|`.

### Implementation
- Google Fonts: Barlow Condensed + Barlow
- `overflow: hidden`
- No border-radius on structural elements
- `clip-path` for diagonal cuts — always include a fallback
- Jersey number watermarks behind section content
- Font Awesome for trophy, lightning bolt, fire icons

---

## Cottagecore Style

Soft floral patterns, watercolor-inspired washes, botanical illustration accents, handwritten feel, and the warm comfort of a storybook cottage. Use when asked for "cottagecore style".

### Typography
Playfair Display (headlines, weights 400–700, italic for key phrases) + Lato Light (body, weight 300). Headlines `font-serif font-normal italic`. Body `font-sans font-light leading-loose`. Labels in `font-serif text-sm tracking-wide`.

### Colors
```
Background:    #fdf8f2   (warm cream parchment)
Sage green:    #8aaa7a
Dusty rose:    #d4848a
Lavender:      #b8a9c9
Warm tan:      #c9a87c
Blush:         #f2d4c8
Mushroom:      #b8a090
Dark text:     #3d2b1f   (warm dark brown)
Muted:         #8a7060
```

### Botanical Accent Pattern
```css
.botanical-bg {
  background-image: url("leaves.svg"), url("flowers.svg");  /* SVG botanicals */
  background-repeat: no-repeat;
  background-position: top right, bottom left;
  background-size: 200px, 180px;
  opacity: 0.15;
}
```
Simple flat SVG leaf/flower shapes at very low opacity in page corners and section edges.

### Watercolor Cards
```css
.cottage-card {
  background: #fdf8f2;
  border: 1px solid rgba(200,160,120,0.3);
  border-radius: 16px 8px 16px 8px;  /* slightly irregular */
  box-shadow: 0 4px 20px rgba(61,43,31,0.08);
  padding: 28px;
  position: relative;
}
/* Subtle watercolor wash on card ::before */
.cottage-card::before {
  content: ''; position: absolute; inset: 0; border-radius: inherit;
  background: linear-gradient(135deg, rgba(138,170,122,0.05), rgba(212,132,138,0.05));
  pointer-events: none;
}
```

### Section Dividers
SVG wavy botanical border — or simple: `border-top: 1px solid rgba(201,168,124,0.4)` with a small centered floral `✿` motif.

### Buttons
```css
.cottage-btn {
  background: #8aaa7a; color: white;
  border: none; border-radius: 100px;
  font-family: 'Playfair Display'; font-style: italic; font-size: 16px;
  padding: 14px 36px;
  box-shadow: 0 4px 16px rgba(138,170,122,0.35);
}
.cottage-btn:hover { background: #7a9a6a; transform: translateY(-2px); box-shadow: 0 8px 24px rgba(138,170,122,0.45); }
.cottage-btn-rose { background: #d4848a; box-shadow: 0 4px 16px rgba(212,132,138,0.35); }
.cottage-btn-outline { background: transparent; border: 1.5px solid #8aaa7a; color: #3d2b1f; border-radius: 100px; }
```

### Floral Tag Pills
`border-radius: 100px; border: 1px solid [color]; background: [color-tint]; padding: 4px 14px; font-serif italic`. Small `✿` before text.

### Quote / Poem Block
`border-left: 3px solid #d4848a; padding: 16px 24px; font-serif italic text-lg text-dark/70`. Attribution in `font-sans text-sm text-muted`.

### Nav
Minimal. No hard lines. `border-bottom: 1px solid rgba(201,168,124,0.3)`. Logo in Playfair italic. Links `font-serif text-sm italic text-muted hover:text-dark`. Sage CTA button.

### Implementation
- Google Fonts: Playfair Display + Lato
- `antialiased`
- Botanical SVG accents in corners (transparent, decorative only)
- `max-w-5xl` container for intimate feel
- Section padding: `py-20`
- Images: warm-toned, `filter: saturate(0.8) sepia(10%)` for cottage photo feel

---

## Japanese Style

Wabi-sabi imperfection, generous negative space treated as structure, ink brush strokes, asymmetric balance, and the visual philosophy that emptiness carries meaning. Use when asked for "japanese style".

### Typography
Cormorant Garamond (headlines — elegant, high contrast strokes) + Source Sans 3 (body, weight 300). Headlines `font-serif font-light tracking-[0.3em] uppercase`. Sometimes vertical text via `writing-mode: vertical-rl`. Large single characters as decorative elements.

### Colors
```
Washi white:   #f7f4ef   (warm, slightly textured)
Sumi black:    #1a1612   (warm near-black, like ink)
Persimmon:     #d9603b   (one accent — Japanese persimmon red)
Bamboo:        #7a9966
Midnight:      #1e2440
Muted:         #8a8070
Gold:          #c9922a   (optional second accent — gold leaf)
```

### Washi Paper Texture
```css
body {
  background: #f7f4ef;
  background-image: url("washi-grain.svg");
  /* CSS approximation: subtle noise */
  background-image: repeating-linear-gradient(
    0deg, transparent 0px, transparent 3px,
    rgba(0,0,0,0.01) 3px, rgba(0,0,0,0.01) 4px
  );
}
```

### Negative Space as Structure
Sections deliberately asymmetric. Content floated to one side, large empty space on the other. Never centered — always slightly off-balance in a considered way.

### Ink Brush Dividers
```css
.brush-line {
  height: 2px;
  background: linear-gradient(to right, #1a1612, #1a1612 60%, transparent 100%);
  border-radius: 0 0 50% 50%;
  /* Simulates brush stroke ending that feathers out */
}
```

### Large Kanji / Character Watermark
```css
.kanji-watermark {
  font-family: serif; font-size: 20vw; line-height: 1;
  color: rgba(26,22,18,0.04);
  position: absolute; right: -0.1em; top: 0;
  writing-mode: vertical-rl; user-select: none;
  pointer-events: none;
}
```
Use a Japanese character or a Roman letter with extreme scale.

### Cards
```css
.wabi-card {
  background: white;
  border-top: 2px solid #d9603b;  /* persimmon top accent only */
  padding: 32px 28px;
  box-shadow: 0 2px 12px rgba(26,22,18,0.06);
  border-radius: 2px;  /* almost no rounding */
}
```

### Buttons
```css
.japan-btn {
  background: transparent; color: #d9603b;
  border: 1px solid #d9603b;
  border-radius: 2px;
  font-family: 'Source Sans 3'; font-weight: 300;
  letter-spacing: 0.3em; text-transform: uppercase; font-size: 12px;
  padding: 12px 36px;
}
.japan-btn:hover { background: #d9603b; color: white; }
.japan-btn-filled { background: #1a1612; color: #f7f4ef; border-color: #1a1612; }
```

### Vertical Text Accent
```css
.vertical-label {
  writing-mode: vertical-rl; text-orientation: mixed;
  font-family: 'Cormorant Garamond'; font-size: 11px;
  letter-spacing: 0.3em; text-transform: uppercase;
  color: #8a8070;
}
```
Used as section labels running vertically along the left margin.

### Implementation
- Google Fonts: Cormorant Garamond + Source Sans 3 (weight 300)
- `antialiased`
- `max-w-5xl mx-auto` — restraint in width
- Sections: `py-32` — generous vertical space
- Images: `filter: saturate(0.6) contrast(1.1)` — muted, contemplative
- Never use bold — `font-light` or `font-normal` maximum

---

## Longform Style

Rich magazine editorial layout — full-bleed hero photography, typographic hierarchy for long reads, pull quotes, drop caps, sidebar annotations, and immersive scrolling narrative. Use when asked for "longform style".

### Typography
Merriweather (body — optimized for reading, 300–700) + Playfair Display (headlines — editorial drama). Subheads: `font-sans font-semibold uppercase tracking-widest text-sm`. Body: `font-serif font-light text-lg leading-[1.9]`. Headlines: `font-serif font-bold tracking-tight leading-none`.

### Colors
```
Background:    #ffffff
Text:          #1a1a1a
Muted:         #6b6b6b
Accent:        #c0392b   (editorial red — for section markers, pull quotes)
Border:        #e8e8e8
Hero overlay:  rgba(0,0,0,0.4)
Byline gray:   #888888
```

### Full-Bleed Hero
```css
.hero-fullbleed {
  height: 90vh; min-height: 600px;
  background-size: cover; background-position: center;
  position: relative;
  display: flex; align-items: flex-end;
}
.hero-fullbleed::after {
  content: ''; position: absolute; inset: 0;
  background: linear-gradient(to top, rgba(0,0,0,0.8) 0%, transparent 60%);
}
/* Headline overlaid on bottom of image */
.hero-headline {
  position: relative; z-index: 1; color: white;
  padding: 48px; max-width: 800px;
}
```

### Reading Column
```css
.reading-col {
  max-width: 680px; margin: 0 auto;
  padding: 48px 24px;
  font-family: 'Merriweather'; font-size: 19px; line-height: 1.9;
}
```

### Drop Cap
```css
.drop-cap::first-letter {
  float: left;
  font-family: 'Playfair Display'; font-size: 5.5rem; font-weight: 700;
  line-height: 0.75; margin-right: 12px; margin-top: 8px;
  color: #1a1a1a;
}
```

### Pull Quote
```css
.pull-quote {
  border-top: 3px solid #c0392b;
  border-bottom: 1px solid #e8e8e8;
  padding: 24px 0;
  margin: 48px 0;
  font-family: 'Playfair Display'; font-style: italic;
  font-size: clamp(22px, 3vw, 32px);
  font-weight: 400; line-height: 1.4;
  color: #1a1a1a;
}
```

### Section Header (chapter-style)
```css
.chapter-head {
  font-family: 'Merriweather'; font-size: 11px;
  font-weight: 700; uppercase; letter-spacing: 0.25em;
  color: #c0392b; padding-bottom: 12px;
  border-bottom: 1px solid #e8e8e8; margin-bottom: 32px;
}
```

### Image Caption
`font-sans text-sm text-muted leading-snug border-top border-[#e8e8e8] pt-2 mt-2`.

### Sidebar Annotation
```css
.annotation {
  position: absolute; right: -200px; width: 180px;
  font-sans font-normal text-sm text-muted line-height-[1.5];
  border-left: 2px solid #c0392b; padding-left: 12px;
}
```

### Byline / Meta
`font-sans text-sm text-muted uppercase tracking-widest` for author. `•` separator. Date.

### Implementation
- Google Fonts: Merriweather (300, 400, 700) + Playfair Display (400, 700)
- `antialiased`
- Single reading column `max-w-[680px]` for body text
- Wide-format callouts break out of column via negative margins: `margin: 0 -80px`
- Image treatment: editorial, slightly high-contrast
- No sidebar nav — this is immersive, distraction-free reading

---
> Source: [chrismccoy/claude-design-styles](https://github.com/chrismccoy/claude-design-styles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-08 -->
