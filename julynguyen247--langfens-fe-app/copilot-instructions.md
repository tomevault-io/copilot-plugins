## langfens-fe-app

> - Next.js 15 (App Router) + React 19 + TypeScript 5 (strict mode)

# Langfens - IELTS Practice Platform

## Tech Stack
- Next.js 15 (App Router) + React 19 + TypeScript 5 (strict mode)
- Tailwind CSS 4 + shadcn/ui (via `class-variance-authority`)
- Framer Motion (component animations)
- Zustand (state management)
- Axios (HTTP client, auto-refresh interceptor)
- Three.js + React Three Fiber + Drei (3D penguin scene — landing page only)
- GSAP + ScrollTrigger (scroll-driven animations — landing page only)
- lottie-web / lottie-react (penguin mascot)
- canvas-confetti (celebration effects)
- Build: `--turbopack` for dev/build

---

## Code Style Rules (STRICT)

### CSS Approach
- **Use Tailwind inline classes** for all styling. Avoid creating custom CSS classes.
- **No separate CSS files** for components. All styles live in JSX via `className` and `style` props.
- `globals.css` is reserved for: CSS variables (scoped), `@keyframes`, pseudo-element rules (::before/::after), and `@layer base`.
- Font families applied via `style={{ fontFamily: 'var(--font-heading)' }}` — NOT via CSS utility classes like `.font-heading`.

### Component Patterns
- Use **shadcn/ui** (`src/components/ui/`) for all new app pages
- Use Next.js `<Image>` for all images
- Use Framer Motion for app page animations
- Use GSAP for landing page scroll animations
- Legacy: `src/components/Button.tsx`, `src/components/Input.tsx` — do NOT use in new code

---

## Global Design System (Duolingo-Soft)

### Visual Rules for NEW Code (STRICT)
- **No icons** on new/redesigned pages. No Material Symbols, no lucide-react, no react-icons, no emoji icons. Use text labels, numbers, or CSS shapes.
- **No gradients** on new pages. Solid colors only.
- **No emojis** in UI content.
- Prefer shadcn primitives. Keep formal, standard.

> **Reality:** Legacy pages (home, analytics, achievements, leaderboard) still use Material Symbols, react-icons (Feather/Heroicons), and gradients on achievement tiers. Do NOT break them — apply the strict rules only to **new** code.

### Icon Usage (Material Symbols)

When icons are needed on legacy pages, use the **Material Icon Component pattern**:

```tsx
// Material Icon Component
function Icon({ name, className = "" }: { name: string; className?: string }) {
  return <span className={`material-symbols-rounded ${className}`}>{name}</span>;
}

// Usage
<Icon name="emoji_events" className="text-2xl text-amber-500" />
```

**Rules:**
- Use `material-symbols-rounded` class (loaded via `material-symbols-css` in AppShell)
- Prefer `text-xl` size for standard icons, `text-2xl` for featured icons
- Always include color class (`text-amber-500`, `text-slate-400`, etc.)
- Use `aria-label` on parent elements when icon meaning is not obvious
- Do NOT use text abbreviations (e.g., "S", "A", "T") as icons — use descriptive Material Icon names

**Common Material Icons:**
| Purpose | Icon Name |
|---------|-----------|
| Achievement/Trophy | `emoji_events` |
| Streak/Fire | `local_fire_department` |
| Score/Grade | `grade` |
| Time/Schedule | `schedule` |
| Tests/Quiz | `quiz` |
| Skill/Psychology | `psychology` |
| Check | `check_circle` |
| Error | `error` |
| Search | `search` |
| Arrow Forward | `arrow_forward` |
| Chevron Left/Right | `chevron_left`, `chevron_right` |
| Close | `close` |
| Lock | `lock` |
| Star | `star` |

**Icon containers:**
- Standard: `w-10 h-10 rounded-full bg-slate-100 flex items-center justify-center`
- Small: `w-8 h-8 rounded-lg bg-slate-100 flex items-center justify-center`

### Colors
| Token | Value | Usage |
|-------|-------|-------|
| `--primary` | `#2563EB` (blue-600) | CTA buttons, primary actions |
| `--primary-hover` | `#1D4ED8` (blue-700) | Button hover states |
| `--primary-dark` | `#1E40AF` (blue-800) | Brand headings |
| `--primary-light` | `#DBEAFE` (blue-100) | Highlight backgrounds |
| `--background` | `#F8F9FA` | Page background |
| `--surface` / `--card` | `#FFFFFF` | Cards, containers |
| `--foreground` | `#111827` | Primary text |
| `--text-body` | `#374151` | Body text |
| `--text-muted` | `#6B7280` | Secondary info |
| `--border` | `#E5E7EB` | Card/section borders |
| `--destructive` | `#EF4444` | Error states |

**Skill badge colors:**
- Reading: `bg-blue-50 text-blue-700`
- Listening: `bg-purple-50 text-purple-700`
- Writing: `bg-amber-50 text-amber-700`
- Speaking: `bg-emerald-50 text-emerald-700`

### Typography
- **Font**: Fredoka — used for EVERYTHING (headings + body). Do NOT use Nunito, Inter, Merriweather, Geist, or any other font. Loaded globally in `layout.tsx`.
- **Monospace**: JetBrains Mono — for stats, scores, data displays. Also loaded globally.
- `font-sans` and `font-serif` both resolve to Fredoka
- H1: `text-3xl sm:text-4xl lg:text-5xl font-extrabold`
- H2: `text-2xl sm:text-3xl font-bold`
- H3: `text-xl font-semibold`
- Body: `text-base text-slate-600`
- Small: `text-sm text-slate-500`
- **No uppercase** — use sentence case everywhere. No `uppercase` CSS class.

### Layout & Spacing
- Container: `max-w-7xl mx-auto px-4 sm:px-6 lg:px-8`
- Section padding: `py-8` to `py-16 lg:py-20`
- Card padding: `p-6` or `p-8`
- Grid gaps: `gap-4` (tight), `gap-6` (normal), `gap-8` (relaxed)

### Border Radius & Shadows (Duolingo-soft 3D)
- Cards: `rounded-[2rem]`, `border-[3px]`, hard drop shadow `shadow-[0_4px_0_rgba(0,0,0,0.08)]`, hover lifts
- Buttons: `rounded-full` (pill shape), `border-b-[4px]` for 3D depth, active presses down
- Badges: `rounded-full`, `border-b-[2px]` subtle depth
- Elevated (modals): `shadow-lg`

### Animations (Framer Motion)
- Reveal: `initial={{ opacity: 0, y: 10-20 }}` → `animate={{ opacity: 1, y: 0 }}`
- Hover: `whileHover={{ y: -2 }}`, `whileTap={{ scale: 0.98 }}`
- Duration: 0.4–0.6s, ease: `easeOut`

---

## Landing Page (`/`) — Dark Cinematic Extension

The landing page extends the global Duolingo-soft design system with a **dark ocean cinematic theme**. All code lives in `src/app/landing/`. Same fonts and component styles as app pages, plus cinematic effects.

### What's different from app pages:
- **Dark background** (#040B14) instead of light (#F8F9FA)
- **Cinematic effects**: particles, 3D penguin, vignettes, caustic overlays, custom cursor
- **GSAP scroll animations** instead of Framer Motion
- **Per-section accent colors** on features, steps, stats

### Theme
Scoped under `.landing-ocean` class on root div. CSS variables defined in `globals.css` (minimal — only vars + keyframes + pseudo-elements).

### Colors
| Token | Value | Usage |
|-------|-------|-------|
| `--ocean-bg` | `#040B14` | Page background |
| `--ocean-bg-light` | `#0A1628` | Card backgrounds |
| `--ocean-primary` | `#2563EB` (blue-600) | CTAs, accents |
| `--ocean-primary-light` | `#3B82F6` | Highlights |
| `--ocean-primary-dark` | `#1E40AF` | 3D button bottom borders |
| `--ocean-primary-glow` | `rgba(37,99,235,0.3)` | Glow shadows |
| `--ocean-accent` | `#06D6A0` | Teal — secondary accent |
| `--ocean-gold` | `#F59E0B` | Stars, achievements |
| `--ocean-text` | `#F0F4F8` | Primary text |
| `--ocean-text-secondary` | `#94A3B8` | Secondary text |
| `--ocean-text-muted` | `#64748B` | Muted text |
| `--ocean-border-glow` | `rgba(37,99,235,0.35)` | Hover border glow |

**Per-section accent colors:** `#2563EB` (blue), `#06D6A0` (teal), `#FF9600` (orange), `#8B5CF6` (purple), `#F59E0B` (gold), `#EC4899` (pink)

### Typography
| Role | Font | Applied via |
|------|------|-------------|
| Headings + Body | **Fredoka** (400-700) | `style={{ fontFamily: 'var(--font-heading)' }}` |
| Stats/scores | **JetBrains Mono** (400-700) | `style={{ fontFamily: 'var(--font-code)' }}` |

Fonts loaded globally in `src/app/layout.tsx`. `--font-heading` and `--font-body` resolve to `--font-sans` (Fredoka) via globals.css `.landing-ocean` scope.

**Type scale:**
- Hero H1: `text-4xl sm:text-5xl lg:text-7xl font-bold`
- Section H2: `clamp(2.5rem, 5vw, 4.5rem) font-bold` (via SectionHeading)
- Card H3: `text-2xl font-bold`
- Labels: `text-sm font-bold tracking-wide` (sentence case, NO uppercase)
- Body: `text-lg`, secondary text color

### Card Style (Chunky Duolingo 3D)
All cards use inline Tailwind (no `.ocean-card` class):
```
bg-[var(--ocean-bg-light)] border-[3px] border-[rgba(255,255,255,0.07)]
rounded-[2rem] shadow-[0_5px_0_rgba(0,0,0,0.35),0_0_0_1px_rgba(255,255,255,0.04)]
transition-all duration-150
hover:-translate-y-[3px] hover:scale-[1.01]
hover:border-[var(--ocean-border-glow)]
hover:shadow-[0_7px_0_rgba(0,0,0,0.35),0_0_25px_var(--ocean-primary-glow)]
```

### Button Styles (Pill-shaped 3D press)
All buttons use inline Tailwind in `Button.tsx` (no `.btn-ocean` class):
- **Primary**: `bg-[var(--ocean-primary)]` pill, `border-b-[5px]` for 3D depth, sentence case, bouncy hover
- **Ghost**: transparent pill, `border-b-[4px]`, hover turns primary
- **Active**: `translate-y-[3px]` + `border-b-[2px]` (press down)
- **Radius**: `rounded-full` (pill shape)

### Sections (7 + header)
| Section | Component | Accent Color |
|---------|-----------|-------------|
| Header | `OceanHeader` | blue (+ streak badge) |
| Hero | `HeroSection` | blue (gradient accent) |
| Features | `FeaturesSection` | per-feature colors |
| How It Works | `HowItWorksSection` | blue/teal/purple per step |
| Stats | `StatsSection` | blue/purple/teal per card |
| Testimonials | `TestimonialsSection` | avatar colors per card |
| CTA | `CTASection` | blue (confetti on click) |
| Footer | `FooterSection` | — |

### Cinematic Effects
- **Particles**: `OceanParticleCanvas` — bubbles, plankton, dust, volumetric light rays
- **Noise**: inline SVG turbulence overlay (in LandingPage.tsx)
- **Vignette**: `ocean-vignette` pseudo-element (globals.css — cannot be inlined)
- **Caustic**: `CausticOverlay` — inline styles + keyframe from globals.css
- **Custom cursor**: `CustomCursor` — bioluminescent dot+ring+trail
- **Loading screen**: `LoadingScreen` — 2s intro, "Almost there..."
- **Confetti**: `useOceanConfetti` — 6 colors: `#2563EB, #06D6A0, #FF9600, #F59E0B, #EC4899, #8B5CF6`

### 3D Penguin Scene
- R3F Canvas: fixed, z-1, pointer-events none, dynamic import (no SSR)
- PenguinModel: Lottie → canvas → THREE.CanvasTexture → Billboard
- PenguinController: 8 scroll-driven keyframe positions
- Skipped on `minimal` device tier

### GSAP Animation Config
In `src/app/landing/lib/animation-config.ts`:
- Entrance easing: `back.out(1.4)` (bouncy pop-in — Duolingo style)
- Durations: 0.3s (fast), 0.6s (normal), 1.0s (slow)
- Stagger: 0.05-0.2s

### Device Tier Degradation
Via `useDeviceCapability` hook:
| Tier | Criteria | Effects |
|------|----------|---------|
| `full` | Desktop, motion enabled | All effects |
| `reduced` | Mobile or prefers-reduced-motion | Fewer particles, no cursor |
| `minimal` | Low CPU/RAM | No R3F, no particles |

### Gamification Elements
- **Streak badge** in OceanHeader (CSS clip-path flame + number)
- **Avatar initials** in TestimonialsSection (colored circles)
- **Score badges** with gradient bg + hard shadow (game achievement feel)
- **Bouncy animations**: `duo-bounce`, `pop-in`, `wiggle`, `float` keyframes

---

## Project Structure
- `src/app/` — Pages (App Router)
- `src/app/landing/` — Cinematic Duolingo landing page (self-contained)
  - `sections/` — OceanHeader, HeroSection, FeaturesSection, HowItWorksSection, StatsSection, TestimonialsSection, CTASection, FooterSection
  - `three/` — PenguinScene, PenguinModel, PenguinController, OceanEnvironment
  - `effects/` — OceanParticleCanvas, CustomCursor, ScrollProgressBar, SectionDots, LoadingScreen, CausticOverlay
  - `hooks/` — useScrollProgress, useCountUp, useSectionInView, useOceanConfetti, useMediaQuery, useReducedMotion
  - `ui/` — Button, GradientText, ScrollIndicator, SectionHeading, StepCard, FeatureVisual
  - `lib/` — gsap-provider, animation-config, utils (lerp, clamp, mapRange)
  - `data.ts`, `LandingPage.tsx`
- `src/app/components/` — Shared effects and interactions
  - `effects/` — useDeviceCapability, useLandingEffectsStore, useScrollVelocity, useIdleDetection
  - `mascot/` — MascotWrapper, useMascotReactions
  - `interactions/` — InteractiveEffects, useConfetti
  - `gamification/` — GamificationHUD, ProgressRing (currently unused)
  - ParticleCanvas.tsx, useMouseParallax.ts, GoogleButton.tsx (root level)
- `src/components/` — Shared components
  - `LangfensHeader.tsx` — Main app header
  - `Button.tsx`, `Input.tsx` — Legacy (do not use in new code)
- `src/components/ui/` — shadcn/ui: badge, button, card, separator
- `src/types/` — TypeScript types
- `src/utils/` — API services (`api.customize.ts`, `api.ts`, `audio.ts`)
- `src/lib/` — Utilities (`utils.ts`, `mapApiQuestionToUi.ts`, `practice.meta.ts`)
- `src/app/store/` — Zustand stores (loading, userStore, useAttemptStore, practiceStore)

## API Architecture
Backend is microservices. 13 services defined in `src/utils/api.customize.ts`:
- auth, exam, attempt, vocabulary, speaking, writing, chatbot, dictionary, gamification, analytics, notification, studyplan, course
- Gateway: `NEXT_PUBLIC_GATEWAY_URL` env var
- Bearer token auth from localStorage, auto-refresh on 401
- Business logic: `src/utils/api.ts` (100+ functions)

## Important Notes
- Landing page (`/`) and auth pages (`/auth/*`) do NOT show the main header
- `src/app/page.tsx` re-exports `LandingPage` from `src/app/landing/LandingPage.tsx`
- `AppShell.tsx` conditionally removes `bg-gray-50` when on `/`
- Other pages still use legacy components — do NOT break them
- The app is bilingual (Vietnamese primary, English secondary) — landing page is English only
- Target audience: IELTS students, primarily Vietnamese high school/university students
- Path alias: `@/*` → `src/*`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/julynguyen247)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/julynguyen247)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
