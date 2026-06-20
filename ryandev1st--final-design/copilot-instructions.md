## final-design

> Use when creating high-fidelity design prototypes, interactive UI demos, production frontend, or needing design direction advisory; when encountering vague design requests; when building animations, iOS/Android mockups, slides, or exporting video/GIF; when enforcing premium UI quality, anti-slop rules, component architecture, CSS performance, or motion design.


# Frontend Design & Prototyper

Merges Huashu Design (prototyping, asset protocols, design advisor, narrative-driven animation, multi-format export) with Design Taste (baseline-driven config, Tailwind/React architecture, Framer Motion, Bento 2.0 dashboards, engineering constraints). All original rules preserved.

---

## 1. Active Baseline Configuration

Global variables driving all generation. User overrides via explicit chat prompts.

| Variable | Default | Range |
|----------|---------|-------|
| `DESIGN_VARIANCE` | 8 | 1 (Perfect Symmetry) → 10 (Artsy Chaos) |
| `MOTION_INTENSITY` | 6 | 1 (Static) → 10 (Cinematic/Magic Physics) |
| `VISUAL_DENSITY` | 4 | 1 (Art Gallery/Airy) → 10 (Pilot Cockpit/Packed Data) |

Defaults: `(8, 6, 4)`. Always listen to user overrides. These values drive Sections 6–8 logic.

---

## 2. Fact Verification — Priority 0 (Override All Other Steps)

**Before any clarifying questions, before writing code, before making assumptions:** if a task involves a specific product, technology, version, or real‑world entity whose existence/release‑status/specs are uncertain, you MUST `WebSearch` it. Do not rely on training memory. Write verified facts to `product-facts.md`.

**If web search is unavailable in the current environment**, explicitly ask the user for product specs, screenshots, and links — never silently assume.

**Banned phrases** (immediately trigger search):
- “I remember X hasn’t been released”
- “X is probably version Y”
- “I think the specs are …”

This rule takes precedence over everything — a wrong fact makes all subsequent design work useless.

---

## 3. Core Philosophy

**Medium Shifting** – HTML is the tool, but the output changes. Embody the appropriate expert: slide designer for decks, animator for motion, UX designer for app prototypes. A slide deck must not feel like a SaaS dashboard; an animation must not look like a static web page.

**Honest Placeholder** – A grey block with a label is infinitely better than a bad AI attempt. Never draw crude SVG faces, icons, or product silhouettes. Use real images or clearly labelled placeholders. “No image yet” is a valid design state; a distorted SVG person is not.

**System First, No Filler** – Every element earns its place. Blank space is a design problem, solved with composition, not by inventing decorative stats or icon slop.

**Variations, Not Final Answers** – Provide 3+ distinct design variants (visual, interaction, layout, motion) from by-the-book to novel. Let the user mix and match.

---

## 4. Core Asset Protocol (Brand/Product Work Mandatory)

For branded work, identity depends on assets in this order:

| Priority | Asset | Requirement |
|----------|-------|-------------|
| 1 | Logo | **Mandatory for all brands** |
| 2 | Product photos/renderings | **Mandatory for physical products** |
| 3 | UI screenshots | **Mandatory for digital products** |
| 4 | Color values | Supporting |
| 5 | Fonts | Supporting |

**Iron rule**: Never use CSS silhouettes or SVG hand-drawn shapes in place of real product images. A missing logo means stop and ask the user.

### 4.1 Five-Step Process

**Step 1 — Ask** (batch all questions, one round):
- Logo (SVG / high-res PNG) — mandatory
- Product photos / official renders — physical products
- UI screenshots — digital products
- Color hexes / brand palette
- Fonts (Display / Body)
- Brand guidelines / Figma / website link

**Step 2 — Search** official channels: `brand.com/press-kit`, product pages, YouTube launch films, App Store screenshots, website CSS. Use `curl`, `grep`.

**Step 3 — Download** with quality threshold:
- **Logo**: direct SVG/PNG; if unavailable, extract inline SVG from homepage HTML; last resort: official social media avatar.
- **Product images**: official hero images (≥2000px), press kit, YouTube screencaps. AI generation (e.g., nano-banana-pro) as last resort. Never CSS shapes.
- **UI screenshots**: App Store, product demos.

**"5–10–2–8" quality filter**: search 5 rounds, gather 10 candidates, select 2 rated ≥8/10 across:
1. Resolution (≥2000px; ≥3000px for print/large screen)
2. Copyright clarity (official > public domain > free stock; suspected theft = 0)
3. Brand fit (consistent with emotional keywords)
4. Stylistic consistency (the 2 chosen assets don't clash when placed together)
5. Independent narrative power (each asset carries meaning, not decoration)

If no asset reaches 8/10, use honest placeholder labeled "product image pending." **Never force low-quality assets.**

**Step 4 — Verify & Extract**:
- Confirm files exist and open correctly.
- Extract colors: `grep -hoE '#[0-9A-Fa-f]{6}' assets/... | sort | uniq -c | sort -rn | head -20`, filter black/white.
- Guard against demo brand color contamination (e.g., a client's tool screenshot showing another brand's red is not the tool's color).

**Step 5 — Solidify as `brand-spec.md`**:

- Logo paths and usage constraints (never stretch, recolor, add stroke)
- Product/UI asset paths with dimensions
- Color palette: Primary, Background, Ink, Accent, Forbidden color ranges
- Typography stacks: Display, Body, Mono
- Signature details (the "120% execution" element)
- Forbidden patterns (explicit bans)
- Emotional keywords (3–5 adjectives)
- All HTML MUST reference these asset files via `<img>`. CSS variables injected from spec only: `:root { --brand-primary: ...; }`. No ad-hoc hex values. Changing a color requires updating the spec first.

---

## 5. Design Direction Advisor (Fallback for Vague Briefs)

Trigger: "make a nice page," "which style?", no design context. Do not guess; present options.

**Phase 1**: Understand audience, core message, emotional tone (max 3 clarifying questions).

**Phase 2**: Restate the need in 100–200 words, ending with "Based on this, I've prepared 3 differentiated directions."

**Phase 3**: Recommend 3 directions from different schools. Each must include a designer/studio reference, 50–100 words on why it fits, 3–4 signature visual traits, and 3–5 emotional keywords.

| School | Character | References |
|--------|-----------|-------------|
| Information Architecture | Rational, data-driven, restrained | Pentagram, Tufte, Maeda |
| Motion Poetics | Dynamic, immersive, tech-aesthetic | Field.io, UENO, Active Theory |
| Minimalist | Order, empty space, refined | Kenya Hara, Naoto Fukasawa, Dieter Rams |
| Experimental/Avant-Garde | Generative, visual tension | Sagmeister & Walsh, Studio Dumbar |
| Eastern Philosophy | Warmth, poetry, speculative | Kenya Hara (MUJI), Kashiwa Sato |

Never recommend 2+ directions from the same school. Differentiation must be obvious.

**Phase 4**: Show existing showcases from `assets/showcases/` if matching.

**Phase 5**: Generate 3 live HTML demos (one per direction) using the user's real content (not Lorem ipsum). Screenshot with Playwright at 1200×900. Present all 3 side-by-side.

**Phase 6**: User selects or mixes ("A's palette + C's layout"). Proceed to standard workflow.

---

## 6. Architecture & Technology Conventions

**Default stacks (user-overridable)**:

| Context | Stack |
|---------|-------|
| Prototypes, demos, slides | Single-file HTML, inline React + Babel standalone. **Must include Tailwind CSS via CDN** (`<script src="https://cdn.tailwindcss.com"></script>`) to support utility classes. No build step required. Assets base64-embedded. Double-click to open — no server required. |
| Production frontend | React/Next.js with Tailwind CSS. Check `package.json` before importing any library. V3 projects: no v4 syntax. V4 projects: use `@tailwindcss/postcss` or Vite plugin. |

**React/Next.js specifics**:
- Server Components by default. Interactive UI (`useState`, `useEffect`, Framer Motion, global state) MUST be extracted into leaf components with `'use client'` at the top.
- Global state only to avoid deep prop-drilling. Wrap providers in a `'use client'` component.
- `useEffect` animations must include strict cleanup functions.

**Styling rules**:
- Tailwind for 90% of styling.
- `min-h-[100dvh]` for full-height sections. Never `h-screen` (iOS Safari viewport bug).
- CSS Grid for layouts (`grid-cols-*`, `gap-*`). Never `calc()` flex math.
- Standard breakpoints: `sm`, `md`, `lg`, `xl`.
- Page containers: `max-w-[1400px] mx-auto`.
- Mobile: high-variance designs (≥4) MUST collapse to single-column `w-full px-4 py-8` below `md`.

**Icons**: `@phosphor-icons/react` or `@radix-ui/react-icons` only. Consistent `strokeWidth` (1.5 or 2.0). **BANNED**: Emojis in code, markup, text, alt text.

**Images**: Fetch real images actively — Wikimedia Commons, Unsplash, Museum APIs (Met, Art Institute Chicago), user's local folders. Test: "If I remove this image, does content lose meaning?" If no, omit it. Decorative stock is AI slop. Placeholder only when all channels fail.

**Output Integrity**: NEVER truncate code with `// ... rest of code`, `// ...`, `/* implement here */`, or similar placeholders. Deliver complete files. If token limits are hit, reduce commentary, not implementation.

---

## 7. Design Engineering (Bias Correction)

Counter AI's statistical biases toward generic patterns. For `DESIGN_VARIANCE ≥ 4`, actively select from positive archetypes (Section 7.7) and the advanced pattern library (Section 7.8) rather than falling back on center-alignment and equal-card grids.

For complete list of banned elements (typography, colors, patterns), see Section 11.

### 7.1 Typography — positive rules
- Prefer Geist, Outfit, Cabinet Grotesk, or Satoshi for premium/creative work. Serif permitted only for creative/editorial brand designs (e.g., Newsreader, Source Serif, EB Garamond as display).
- Dashboard/Software UIs: high-end sans-serif pairings (Geist + Geist Mono, or Satoshi + JetBrains Mono). Serif banned.
- Body: `text-base text-gray-600 leading-relaxed max-w-[65ch]`.

### 7.2 Color Calibration — positive rules
**CRITICAL: The default LLM “AI Purple/Blue/Neon gradient” aesthetic is BANNED.** This is the single most common generic trait and must be actively subverted. Use zinc/slate neutral bases with a singular high‑contrast accent (Emerald, Electric Blue, Deep Rose, Rust, Bronze, etc.). Max 1 accent color, saturation < 80%. Stick to one palette per project; no mixing warm/cool grays. Colors defined via `oklch()` or spec‑derived hex values; never invent ad‑hoc colors.

### 7.3 Layout Diversification
- When `DESIGN_VARIANCE > 4`: centered hero sections are banned. Use split-screen (50/50), left-aligned content with right-aligned asset, or asymmetric white-space structures.
- **BANNED: The 3-equal-cards-in-a-row feature grid.** Use 2-column zig-zag, asymmetric grid, or horizontal scrolling instead.

### 7.4 Material, Shadows, and Card Discipline
- When `VISUAL_DENSITY > 7`: no generic card containers. Use `border-t`, `divide-y`, or negative space for logic grouping. Data metrics breathe without boxes unless elevation (z-index) is functionally required.
- Use cards only when elevation communicates hierarchy.
- When using shadows, tint them to the background hue. Never pure black.
- **"Liquid Glass" refraction**: for glassmorphism, go beyond `backdrop-blur`. Add 1px inner border (`border-white/10`) and inner shadow (`shadow-[inset_0_1px_0_rgba(255,255,255,0.1)]`) to simulate physical edge refraction.

### 7.5 Interactive UI States
Mandatory full-cycle generation (AI defaults to static success states):
- **Loading**: Skeletal loaders matching layout dimensions. No generic circular spinners.
- **Empty**: Beautifully composed states explaining how to populate data.
- **Error**: Clear, inline error reporting.
- **Tactile feedback**: On `:active`, use `-translate-y-[1px]` or `scale-[0.98]` to simulate physical press.

### 7.6 Forms
- Labels always above inputs.
- Helper text optional but should exist in markup.
- Error text below input.
- Standard `gap-2` for input blocks.

### 7.7 Positive Archetype Pool (DESIGN_VARIANCE ≥ 4)

To prevent the "bland default" look, actively choose from these structural and atmospheric archetypes when variance is 4 or higher. Roll a direction (or let user pick):

| Vibe Archetype | Visual DNA | Best For |
|----------------|------------|----------|
| **Ethereal Glass** | Ultra-blurred backdrops, soft pastel gradients, 1px inner borders, generous white space, thin sans-serif | Premium SaaS, wellness apps, creative agencies |
| **Editorial Luxury** | High contrast, large serif display, gold/copper accent, strict grid, large leading | Fashion, editorial, high-end portfolios |
| **Organic Minimal** | Warm neutrals, rounded corners (24px+), asymmetric white space, plant/stone imagery, soft shadows | Sustainability, health, lifestyle brands |
| **Industrial Brut** | Monospace/machine typography, raw borders, no shadows, tight spacing, desaturated cyan/orange accents | Dev tools, data dashboards, hardware startups |
| **Retro Avant-Garde** | Grain textures, bold clashing colors, distorted typography, collage-like overlapping | Music, youth brands, experimental projects |
| **Academic Classical** | Serif body, restrained palette (sepia/navy), strict justified text, rule lines, generous margins | Research, legal, publishing |

| Layout Archetype | Structure | When |
|------------------|-----------|------|
| **The Split Stage** | 50/50 screen (text left, media right) with subtle overlap and scroll-triggered reveal | Hero sections, product showcases |
| **The Z-Axis Cascade** | Elements step forward/backward via z-index + transform, creating depth as you scroll | Feature lists, timeline narratives |
| **The Masonry Gallery** | Imagery/content in staggered columns with variable heights, no rigid rows | Portfolios, case studies, discovery feeds |
| **The Bento Grid** | Asymmetric tile-based grouping (Apple Control Center style); see Section 9 for complete micro-animation card specs | Dashboards, feature indexes, SaaS homepages |
| **The Curtain Reveal** | Full-screen panel that splits apart on scroll, revealing the next section | Dramatic storytelling, launch pages |

### 7.8 Advanced Interaction Patterns & Motion Library (from Taste Creative Arsenal)

When `MOTION_INTENSITY ≥ 5` and user wants distinctive, non‑generic interactions, pull from these concrete, high‑end patterns. The environment‑lock column tells you which build context supports the pattern.

| Category | Pattern | Description | Env |
|----------|---------|-------------|-----|
| Navigation | **Mac OS Dock Magnification** | Icons scale fluidly on hover, mimicking physical proximity | Next.js (Framer Motion) |
| Navigation | **Magnetic Button** | Button subtly pulls toward the cursor; uses `useMotionValue` | Next.js |
| Navigation | **Gooey Menu** | Sub‑items detach from the main button like viscous liquid | CSS/standalone (SVG filters) |
| Navigation | **Dynamic Island** | Pill‑shaped component morphs to show status / alerts | Both |
| Navigation | **Contextual Radial Menu** | Circular menu expands precisely at click coordinates | Both |
| Navigation | **Floating Speed Dial** | FAB that springs into a curved line of secondary actions | Both |
| Navigation | **Mega Menu Reveal** | Full‑screen dropdown stagger‑fades complex content | Next.js |
| Cards | **Parallax Tilt Card** | 3D‑tilting card tracking mouse coordinates | Next.js (Framer Motion) |
| Cards | **Spotlight Border Card** | Border illuminates dynamically under cursor | CSS/standalone |
| Cards | **Holographic Foil Card** | Iridescent, rainbow light reflections shift on hover | CSS/standalone |
| Cards | **Tinder Swipe Stack** | Physical stack of cards user can swipe away | Next.js |
| Cards | **Morphing Modal** | Button seamlessly expands into its own full‑screen dialog | Next.js (layoutId) |
| Scroll | **Sticky Scroll Stack** | Cards stick to top and physically stack over each other | Both |
| Scroll | **Horizontal Scroll Hijack** | Vertical scroll translates into smooth horizontal gallery | Both |
| Scroll | **Zoom Parallax** | Central background zooms in/out seamlessly while scrolling | CSS/standalone |
| Scroll | **Liquid Swipe Transition** | Page wipes like viscous liquid | Next.js |
| Scroll | **Scroll Progress Path** | SVG vector lines draw themselves as user scrolls | CSS/standalone |
| Galleries | **Coverflow Carousel** | 3D carousel with center focus, edges angled back | Next.js |
| Galleries | **Drag‑to‑Pan Grid** | Boundless grid freely draggable in any direction | Next.js |
| Galleries | **Accordion Image Slider** | Narrow strips expand fully on hover | CSS/standalone |
| Galleries | **Hover Image Trail** | Mouse leaves a trail of popping/fading images | Next.js |
| Galleries | **Glitch Effect Image** | Brief RGB‑channel shift distortion on hover | CSS/standalone |
| Typography | **Kinetic Marquee** | Endless text bands reverse direction or speed up on scroll | Both |
| Typography | **Text Mask Reveal** | Massive typography acts as transparent window to video background | CSS/standalone |
| Typography | **Text Scramble Effect** | Matrix‑style character decoding on load or hover | Next.js (useCycle) |
| Typography | **Gradient Stroke Animation** | Outlined text with gradient continuously running along stroke | CSS/standalone |
| Effects | **Particle Explosion Button** | CTAs shatter into particles on success | Next.js (canvas) |
| Effects | **Skeleton Shimmer** | Shifting light reflections across placeholder boxes | CSS/standalone |
| Effects | **Ripple Click Effect** | Visual waves rippling precisely from click coordinates | Both |
| Effects | **Animated SVG Line Drawing** | Vectors draw their own contours in real‑time | CSS/standalone |
| Effects | **Mesh Gradient Background** | Organic, lava‑lamp‑like animated color blobs | CSS/standalone |

**Usage rule**: Implement the pattern only when it fits the narrative; never force it. Always respect the motion and performance constraints of the target environment (Sections 8, 10).

---

## 8. Motion & Creative Proactivity

Motion implementations are strictly dependent on the execution environment. **Framer Motion is only available in production (Next.js) builds.**

### 8.1 CSS-Only (all environments, MOTION_INTENSITY 4–7)
- `transition: all 0.3s cubic-bezier(0.16, 1, 0.3, 1)`.
- `animation-delay` cascades for staggered load-ins.
- Animate only `transform` and `opacity`. Use `will-change: transform` sparingly.

### 8.2 Prototype / Standalone HTML (MOTION_INTENSITY 8–10)
- Use the built-in animation engine: `assets/animations.jsx` (Stage, Sprite, useTime, Easing, interpolate). This avoids the external dependency problem.
- For scroll-driven storytelling, use GSAP (ScrollTrigger) via CDN. Never mix GSAP with Framer Motion in the same component tree (even if both were available).
- ThreeJS for 3D/canvas backgrounds; same isolation rule.
- All GSAP/ThreeJS instances must be wrapped in `useEffect` with strict cleanup (kill ScrollTriggers, dispose renderers).
- Perpetual loops must be `React.memo`-isolated, using `window.__ready` / `window.__recording` guards for video export safety.

### 8.3 Production (Next.js, MOTION_INTENSITY 8–10)
- Use Framer Motion exclusively for UI-level micro-interactions.
- Spring physics: `type: "spring", stiffness: 100, damping: 20`. No linear easing.
- **Perpetual micro-interactions**: infinite loops (pulse, typewriter, float, shimmer, carousel) embedded in cards, avatars, status dots.
- **Magnetic hover**: buttons pulling toward cursor via `useMotionValue` and `useTransform`. Never `useState` for continuous animation.
- **Staggered orchestration**: `staggerChildren` variants or CSS cascade `animation-delay: calc(var(--index) * 100ms)`. Parent and children must be in the same client component tree; async data passed as props.
- **Layout transitions**: `layout` and `layoutId` for re-ordering, resizing, shared-element transitions. Wrap dynamic lists in `<AnimatePresence>`.
- **GSAP/ThreeJS exception**: For full-page scroll-telling or 3D backgrounds, GSAP/ThreeJS may be used but strictly isolated from Framer components. Wrap in `useEffect` with full cleanup.

### 8.4 Narration-Driven Animation (Long-Form Explainer Video)
For "5–20 minute explainer," "voiceover tutorial," "concept video with narration":
- **The whole piece is one continuous motion narrative, not independent scenes.** Select 1–2 hero elements that persist across scenes, morphing in position/size/form. Scene transitions are morphs, not cuts.
- **Each scene with independent layout + fade-up cue + full-page opacity switch = narrated PowerPoint = zero quality.** This is the #1 failure mode.
- Write narration script first: markdown, `## scene-id` segments, `[[cue:xx]]` for key sentences.
- Run TTS (Doubao, configured in `.env`) to produce `voiceover.mp3` + `timeline.json` (real measured timestamps, not character-count estimates).
- Animate HTML using `assets/narration_stage.jsx` (`NarrationStage`, `Scene`, `Cue`, `useNarration`, `useSceneFade`, `Subtitles`). Hero element goes directly inside `<NarrationStage>`, not inside `<Scene>`.
- `<Subtitles />` always on (dark ink text + white glow, auto-segmented ≤12 chars per line, never break across sentences).
- Final export: record silent MP4 → mix voiceover → optionally add BGM. Use `scripts/render-narration.sh`. Confirm audio stream exists with `ffprobe`.

### 8.5 Video Export (Animation → MP4/GIF)
Default delivery for animations is **MP4 with audio** (SFX + BGM). Silent equals half-finished.
- Record 25fps base MP4 (intermediate, not deliverable).
- Derive 60fps MP4 + palette-optimized GIF.
- Add scene-appropriate BGM (6 mood options: tech, ad, educational, tutorial + alt variants).
- Add SFX: 37 premade assets in `assets/sfx/<category>/*.mp3`. Density: hero/promo ~6 cues per 10s; tool demo ~0–2 cues per 10s.
- BGM occupies low frequencies, SFX occupies highs — use ffmpeg ducking templates from `references/audio-design-rules.md`.
- Skip audio only if user explicitly requests "no audio," "silent export," or "I'll voice it myself."
- Watermark: "Created by Huashu-Design" (bottom-right, monospace, 11px, 0.4 opacity) on all animation exports. For unofficial tributes to third-party brands, prefix with "非官方出品 · ". Never watermark slides, prototypes, infographics, or web pages.

---

## 9. Bento 2.0 Dashboard Paradigm

For SaaS dashboards and feature grids, enforce this aesthetic:

**Core specs**:
- Background: `#f9fafb`.
- Cards: pure white `#ffffff`, `border-slate-200/50`, `rounded-[2.5rem]`, tinted diffuse shadow `shadow-[0_20px_40px_-15px_rgba(0,0,0,0.05)]`.
- Labels: titles and descriptions placed outside and below cards.
- Padding: generous `p-8` or `p-10` inside cards.
- Fonts: strictly Geist, Satoshi, or Cabinet Grotesk. `tracking-tight` for headers.

**Environment lock**: Bento 2.0 micro-animations (layoutId, spring physics) rely on Framer Motion and are strictly for **Production (Next.js)** builds. For standalone HTML prototypes, fall back to CSS transitions per Section 8.1.

**Five mandatory card archetypes** (each with perpetual micro-motion):  
*All perpetual motion must strictly follow the memoization guardrails in Section 10.*

| # | Archetype | Animation Spec |
|---|-----------|----------------|
| 1 | **Intelligent List** | Vertical stack of items auto-sorting via `layoutId`, simulating AI prioritization in real time. |
| 2 | **Command Input** | Multi-step typewriter cycling through complex prompts; blinking cursor; "processing" state with shimmering loading gradient. |
| 3 | **Live Status** | Breathing status indicators; pop-up notification badge with overshoot spring effect, 3-second dwell, then vanish. |
| 4 | **Wide Data Stream** | Seamless horizontal infinite carousel (`x: ["0%", "-100%"]`) of data cards/metrics; effortless-feeling speed. |
| 5 | **Contextual UI (Focus Mode)** | Staggered highlight of a text block, then float-in of a floating action toolbar with micro-icons. |

---

## 10. Performance Guardrails

- **Animate only `transform` and `opacity`.** Never animate `top`, `left`, `width`, or `height`.
- **Noise/grain filters**: apply exclusively to fixed, `pointer-events-none` pseudo-elements (`fixed inset-0 z-50 pointer-events-none`). Never on scrolling containers — causes continuous GPU repaints and mobile performance degradation.
- **z-index restraint**: use only for systemic layer contexts (sticky navbars, modals, overlays). Never arbitrary `z-50` or `z-10` unprompted.
- **Motion performance**: perpetual animations isolated in memo'd client components. Dynamic lists wrapped in `<AnimatePresence>`.

---

## 11. Anti-Slop Enforcement (Unified Checklist)

### Visual & CSS
- **BANNED: Neon or outer glows.** Use inner borders or tinted shadows.
- **BANNED: Pure black (`#000000`).** Use Off-Black, Zinc-950, or Charcoal.
- **BANNED: Oversaturated accents.** Desaturate to blend with neutrals.
- **BANNED: Excessive gradient text** on large headers.
- **BANNED: Custom mouse cursors** (outdated, ruin performance/accessibility).
- **BANNED: The arbitrary purple/blue "AI aesthetic" gradients.**

### Typography
- **BANNED: Inter font.** Use Geist, Outfit, Cabinet Grotesk, or Satoshi.
- **Oversized H1s:** control hierarchy with weight and color, not just scale.
- **Serif**: permitted only for creative/editorial work; **banned for dashboards/software UI.**

### Layout & Spacing
- Alignments and padding mathematically perfect. No floating elements with awkward gaps.
- **BANNED: The 3-column equal-card feature row.** Replace with zig-zag, asymmetric grid, or horizontal scroll.

### Content & Data
- **NEVER USE: Generic placeholder names** ("John Doe," "Sarah Chan," "Jack Su"). Use creative, realistic names.
- **NEVER USE: Generic SVG "egg" avatars** or Lucide user icons for avatars. Use photo placeholders.
- **NEVER USE: Fake round numbers** (`99.99%`, `50%`, `1234567`). Use organic data (`47.2%`, `+1 (312) 847-1928`).
- **NEVER USE: Startup-sludge names** ("Acme," "Nexus," "SmartFlow"). Invent premium, contextual brand names.
- **NEVER USE: Filler words** ("Elevate," "Seamless," "Unleash," "Next-Gen"). Use concrete verbs.

### External Resources & Components
- **NEVER USE: Unsplash.com links** (unreliable). Use `https://picsum.photos/seed/{random_string}/800/600` or SVG UI avatars for placeholders.
- **shadcn/ui**: must customize radii, colors, and shadows. Never use default styling.

---

## 12. App Prototypes (iOS/Android)

When building mobile mockups:

### 12.1 Architecture
- Default: single-file inline React, base64-embedded assets. Double-click to open.
- Split into external files only if >1000 lines or multi-agent parallel work.

### 12.2 Device Frame
- **Mandatory**: use `assets/ios_frame.jsx` (iPhone 15 Pro bezel, Dynamic Island 124×36px top:12 centered, status bar with time/signal/battery avoiding island, Home Indicator). Never hand-code `.dynamic-island`, `.status-bar`, or `.home-indicator` in project HTML.
- Usage: `<IosFrame time="9:41" battery={85}><YourScreen /></IosFrame>`.
- Content renders from top 54px; bottom safe area for Home Indicator handled automatically.
- Exceptions: only when user explicitly requests non-Pro notch, Android device, or custom form factor.

### 12.3 Deliverables
Ask user which format:
- **Overview** (design review default): all screens tiled side-by-side, each in its own IosFrame, labels below. Not interactive.
- **Flow Demo** (user journey demo): single interactive IosFrame with `AppPhone` state machine. Tab bar, buttons, cards are clickable via `onEnter`/`onClose`/`onTabChange`/`onOpen` callback props.

### 12.4 Pre-Ship Testing
Run Playwright click tests **only if the execution environment supports CLI / terminal commands** (e.g., local IDE, Claude Code). If not (cloud-only chat), manually inspect all interactive states and the browser console. Tests to perform: enter detail view, tap annotation points, switch all tabs. Verify `pageerror` count = 0.

---

## 13. Slide Decks

- **Default basis**: multi-file HTML + `assets/deck_index.html` aggregator (keyboard nav, fullscreen presentation, print consolidation). This is the source artifact for any slide project — always produce it first, regardless of final format.
- **Single-file alternative** (≤10 slides, pitch decks): use `assets/deck_stage.js` web component with auto-scale, keyboard nav, slide counter, localStorage state, speaker notes.
- **Showcase protocol** (≥5 slides): deliver 2-slide showcase first to lock visual grammar before batch-producing remaining slides.
- **Derivatives**: PDF (`scripts/export_deck_pdf.mjs`) or editable PPTX (`scripts/export_deck_pptx.mjs`) are optional add-ons. PPTX requires 4 strict HTML constraints (see `references/editable-pptx.md`); if visual freedom is more important, export PDF instead.
- Fixed-aspect-ratio content: use JS auto-scale with letterboxing; never CSS `scale()` alone.

---

## 14. Workflow (Junior Designer Model)

### Step 1 — Understand
- **Fact-check first (Priority 0, Section 2):** always verify uncertain product/tech/version with `WebSearch`; write `product-facts.md`. If web search is unavailable, explicitly ask the user.
- Clarifying questions (one batch, max one round). Use `references/workflow.md` templates.
- If user says "just do it," respect that. Produce 1 main + 1 high-contrast variant. Label all assumptions explicitly.

### Step 2 — Collect Assets
- If branded work: execute full Core Asset Protocol (Section 4). Write `brand-spec.md`.
- Self-check: do I have logo? Product images/UI screenshots? Real hex values? Missing → stop and ask.
- If no design context exists at all → trigger Design Direction Advisor (Section 5).

### Step 3 — Answer Position Four Questions (per screen/slide/page)
1. **Narrative role**: hero / transition / data / quote / ending?
2. **Viewing distance**: 10cm phone / 1m laptop / 10m projection?
3. **Visual temperature**: quiet / excited / calm / authoritative / gentle / somber?
4. **Content fit**: thumbnail-test 3 quick layouts — does content fit without squeezing?

Vocalize design system (colors, typography, layout rhythm, component patterns) based on answers. System serves the answers, not vice versa. Wait for user confirmation before writing code.

### Step 4 — Junior Pass
- Write HTML with assumptions, placeholders, reasoning comments visible.
- Show user early (even if just grey blocks + labels). Wait for confirmation.

### Step 5 — Full Pass
- Fill placeholders. Apply motion per baseline config and environment. For `DESIGN_VARIANCE ≥ 4`, select a positive archetype from Section 7.7 and, if appropriate, a high‑end pattern from Section 7.8. Generate 3+ cross-dimensional variations.
- If the interaction would benefit from live parameter adjustment (colors, spacing, layout), integrate **Tweaks** – a real-time control panel using CSS custom properties and toggles. See `references/tweaks-system.md` for the full localStorage-based implementation.
- Show again mid-way. Do not wait until "finished."

### Step 6 — Verify
- Playwright screenshot (if CLI available) or manual visual pass. Browser console check.
- App prototypes: run click test per Section 12.4.

### Step 7 — Export
- Animations → MP4 with audio (SFX + BGM) (Section 8.4/8.5).
- Slides → PDF/PPTX if requested.
- Watermark on animation exports only.

### Step 8 — Summarize
- Minimal: caveats + next steps.

### Optional — Expert Critique
- If user asks "review," "score," or "is this good?": run 5-dimension critique (Philosophy Consistency / Visual Hierarchy / Detail Execution / Functionality / Innovation; each 0–10). Output totals + Keep + Fix (with severity: ⚠️ Fatal / ⚡ Important / 💡 Optimization) + Quick Wins (top 3 things doable in 5 minutes). Critique the design, not the designer.

---

## 15. Exception Handling

| Condition | Action |
|-----------|--------|
| Brief too vague | Offer 3 directions (not 10 questions). |
| User refuses Q&A | Best-judgment 1 main + 1 variant. Label assumptions. |
| Design context contradicts itself | Stop. Point out specific contradiction. Let user decide. |
| Starter component fails (404/integrity) | Check `references/react-setup.md` error table. Fallback: plain HTML+CSS, no React. |
| <30 minutes deadline | Skip Junior pass. Single solution. Warn "not early-validated." |
| HTML >1000 lines | Split into multiple .jsx files, `Object.assign(window, ...)` to share. |
| Product is AI/Data/Context-aware (tracker, copilot, dashboard) | Raise information density: ≥3 visible product-differentiating data points per screen. Density serves the product's intelligence signal — don't strip it in the name of minimalism. |

---

## 16. Technical Redlines (React + Babel)

- Pinned versions from `references/react-setup.md`.
- **Never** write `const styles = {...}` — naming collisions across components. Always use unique prefix: `const terminalStyles = {...}`.
- Multiple `<script type="text/babel">` blocks: scopes don't share. Use `Object.assign(window, {...})` to export.
- **Never** use `scrollIntoView` — breaks container scroll. Use alternative DOM scroll methods.
- Fixed-size content (slides/video): implement JS auto-scale + letterboxing.
- **Never** truncate code. Deliver full files. No `// ...`, `// rest of code`, `/* implement here */`.

---

## 17. Assets & Reference Routing

### Core Setup & Workflow
| Need | Read/Use |
|------|----------|
| Ask questions, set direction | `references/workflow.md` |
| Anti-slop details, scale | `references/content-guidelines.md` |
| React+Babel setup, errors | `references/react-setup.md` |
| No design context fallback | `references/design-context.md` |
| 20 design philosophies (full) | `references/design-styles.md` |
| Scene templates by output type | `references/scene-templates.md` |
| Verification & Playwright | `references/verification.md` |
| Critique & scoring | `references/critique-guide.md` |
| Tweaks live-tuning system | `references/tweaks-system.md` |

### Motion & Video Pipeline
| Need | Read/Use |
|------|----------|
| Animation pitfalls (14 rules) | `references/animation-pitfalls.md` |
| Animation best practices | `references/animation-best-practices.md` |
| Voiceover pipeline (narration) | `references/voiceover-pipeline.md` |
| Video export flow | `references/video-export.md` |
| SFX library (37 assets) | `references/sfx-library.md` |
| Audio rules (BGM+SFX) | `references/audio-design-rules.md` |
| Apple gallery showcase style | `references/apple-gallery-showcase.md` |
| Gallery ripple + multi-focus | `references/hero-animation-case-study.md` |

### Components & Frames
| Need | Use |
|------|-----|
| iOS frame component | `assets/ios_frame.jsx` |
| Android frame component | `assets/android_frame.jsx` |
| macOS window component | `assets/macos_window.jsx` |
| Browser window component | `assets/browser_window.jsx` |
| Design canvas (variations) | `assets/design_canvas.jsx` |
| Animation engine | `assets/animations.jsx` |
| Narration stage | `assets/narration_stage.jsx` |

### Export Scripts
| Need | Use |
|------|-----|
| Slide aggregator (multi-file) | `assets/deck_index.html` |
| Slide single-file component | `assets/deck_stage.js` |
| Render video | `scripts/render-video.js` |
| Convert formats | `scripts/convert-formats.sh` |
| Add music | `scripts/add-music.sh` |
| Narrate pipeline | `scripts/narrate-pipeline.mjs` |
| Mix voiceover | `scripts/mix-voiceover.sh` |
| Render narration | `scripts/render-narration.sh` |
| Export deck PDF | `scripts/export_deck_pdf.mjs` |
| Export deck PPTX | `scripts/export_deck_pptx.mjs` |

All paths relative to this skill's root directory.

---
> Source: [RyanDev1st/final-design](https://github.com/RyanDev1st/final-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
