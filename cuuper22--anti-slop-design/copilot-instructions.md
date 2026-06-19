## anti-slop-design

> >


# Anti-Slop Design System

Every AI model in existence has converged on the same aesthetic: purple gradient,
Inter font, 8px border radius, centered hero, three perfectly equal columns, a
sprinkle of Heroicons, and a dark mode that's just `#000000` with white text.
It's the design equivalent of elevator music. This skill exists because I got
tired of every AI-generated UI looking like it was built by the same sleep-deprived
intern at a Series A startup in 2023. Anti-slop covers 8 domain profiles (SaaS,
e-commerce, editorial, fintech, healthcare, creative, dev-tools, and general),
ships a 15-rule anti-slop checklist that catches generic output before it reaches
the user, and follows progressive disclosure: you're reading the hub doc now,
which points to `domain-map.json` for domain profiles, `references/` for
platform-specific guidance, `assets/` for tokens and textures, and `templates/`
for starting points. Read this file first. Everything else flows from here.


---

## Design Thinking Protocol

Before generating ANY design output — a React component, a landing page, an
artifact, a CLI app, whatever — run through these five steps. Every time. No
shortcuts. The whole point is that generic output is the default failure mode,
and the only way to avoid it is a deliberate process.

### Step 1: Identify Domain

Read the user's prompt. What are they building? A fintech dashboard? An
e-commerce checkout? A developer tool? Match keywords to one of the 8 domains
defined in `domain-map.json`:

- **saas** — B2B dashboards, admin panels, analytics platforms
- **ecommerce** — Product pages, carts, checkout flows, catalogs
- **editorial** — Blogs, magazines, news sites, content-heavy layouts
- **fintech** — Banking, trading, crypto, financial dashboards
- **healthcare** — Patient portals, EHR interfaces, wellness apps
- **creative** — Portfolio sites, agency pages, design studios
- **devtools** — IDEs, documentation, API explorers, terminal apps
- **general** — The fallback. Use this only if the prompt genuinely doesn't
  fit any other category. "Make me a website" with zero context? Fine, general.
  But try harder before defaulting here.

If the prompt is ambiguous — "build me a dashboard" could be SaaS, fintech, or
healthcare — ask the user. Don't guess. A fintech dashboard and a healthcare
dashboard should look nothing alike.

### Step 2: Load Domain Profile

Open `domain-map.json`. Find the matching domain key. Extract the full profile:

- **Color palette** — OKLCH values. Not hex. Not HSL. OKLCH, because it's
  perceptually uniform and you can derive tints/shades without things going
  muddy. Each domain has primary, secondary, accent, neutral, success, warning,
  error, and info colors with 10 lightness steps each.
- **Typography stack** — Primary and secondary font families, weights, and
  the fluid type scale to use. No, you cannot substitute Inter.
- **Border radius** — The `shape.borderRadius` value. Could be 2px for fintech,
  12px for creative, 6px for SaaS. It is NOT 8px for everything.
- **Motion level** — `none`, `minimal`, `moderate`, or `expressive`. Fintech
  gets `minimal`. Creative gets `expressive`. This controls whether you use
  transitions, spring animations, or nothing at all.
- **Density** — `compact`, `normal`, or `spacious`. Affects spacing multipliers.
- **Shadows** — Shadow style and intensity. Some domains use sharp shadows,
  some use diffuse, some use almost none.

### Step 3: Select Platform Reference

Based on what the user is building (not what domain they're in), load the
correct reference file from `references/`:

- Building a React dashboard? → `references/web-react.md`
- Building a landing page? → `references/web-landing.md`
- Building a Claude artifact? → `references/web-artifacts.md`
- Building an iOS app? → `references/mobile-native.md`
- Building a CLI tool? → `references/cli-terminal.md`
- And so on. See the Platform Routing Table below for the full mapping.

The reference file contains platform-specific constraints, patterns, and
anti-patterns. Read it. It's there for a reason.

### Step 4: Apply Anti-Slop Checklist

This is the core of the system. Before you show anything to the user, run
every output decision through the 15-rule checklist below. Every. Single. One.
If any rule is violated, fix it. Don't ship slop with a "you might want to
change the colors" disclaimer. Just fix it.

The checklist is not optional. It's not a suggestion. It's the whole point.

### Step 5: Assemble from Assets

Now build the thing:

1. Start with the appropriate template from `templates/`
2. Copy CSS tokens from `assets/css/` (reset, fluid scales, motion tokens)
3. Inject domain colors from `assets/tokens/domain-tokens/{domain}.json`
4. Apply domain typography, spacing, and shape values
5. Select SVG textures if the design calls for them
6. Customize the template with domain-specific values
7. Run the checklist one more time. Yes, again.


---

## Anti-Slop Checklist

Fifteen rules. Memorize them. Tattoo them on your forearm. Every single one
exists because AI models consistently get it wrong. These are not edge cases —
they are the most common failures in AI-generated design output.

### Rule 1: Typography

- **Check:** Is the primary font Inter, Roboto, or Open Sans?
- **Fix:** Use the domain font stack from domain-map.json. Every domain has a
  curated primary + secondary font pairing. Inter is fine as a *body* font for
  a dev-tools project. It is not fine as the primary font for an editorial
  magazine layout. Load the domain's `typography.primary` and
  `typography.secondary` values. Use them.

### Rule 2: Color

- **Check:** Is there a purple-to-blue gradient anywhere? Any gratuitous
  gradient that screams "AI made this"?
- **Fix:** Use the domain's OKLCH palette. Gradients are fine when they serve
  a purpose (data visualization heat maps, progress indicators). They are not
  fine as the background of every hero section and button. If you need a
  gradient, build it from two adjacent colors in the domain palette.

### Rule 3: Layout

- **Check:** Is it centered-hero → three-equal-columns → CTA? The Holy Trinity
  of boring?
- **Fix:** Use asymmetric layouts. Bento grids. Editorial layouts with varied
  column widths. Full-bleed sections mixed with contained content. The domain
  profile includes a `layout.style` hint — use it. An editorial site should
  feel like a magazine, not a SaaS landing page.

### Rule 4: Background (Light)

- **Check:** Is the background pure `#FFFFFF`?
- **Fix:** Use warm off-whites from `assets/css/primitives-light.css`. Values
  like `oklch(0.985 0.002 90)` or `oklch(0.975 0.005 80)`. Pure white is
  harsh and sterile. Off-whites feel intentional and designed.

### Rule 5: Background (Dark)

- **Check:** Is the dark mode background pure `#000000`?
- **Fix:** Use dark primitives in the `oklch(0.13–0.17)` lightness range. Pure
  black creates too much contrast with text and makes everything look like a
  terminal from 1985. Use something like `oklch(0.145 0.005 260)` — dark enough
  to be obviously dark mode, warm enough to not feel like staring into the void.

### Rule 6: Border Radius

- **Check:** Is `border-radius: 8px` applied to everything?
- **Fix:** Use `domain.shape.borderRadius` from the domain profile. A fintech
  app should have tight, precise corners (2-4px). A creative portfolio can go
  rounder (12-16px). Consistency within a domain matters more than any specific
  value. Also: vary radius by component size — a card and a button inside that
  card should not have the same border-radius.

### Rule 7: Shadows

- **Check:** Are you using Material Design elevation shadows everywhere?
  (`0 2px 4px rgba(0,0,0,0.1)` on literally everything?)
- **Fix:** Check `domain.shadows` in the profile. Some domains use sharp,
  offset shadows (editorial, creative). Some use extremely subtle shadows
  (fintech, healthcare). Some barely use shadows at all (dev-tools). Match
  the domain. And for the love of clarity, don't put a shadow on every card
  just because it's a card.

### Rule 8: Animation

- **Check:** Is everything bouncing and springing around like a toddler on
  sugar?
- **Fix:** Check `domain.motion` in the profile. `minimal` means subtle
  transitions only — opacity and transform, 150-200ms, ease-out. `moderate`
  allows entrance animations and micro-interactions. `expressive` lets you go
  wild with spring physics and staggered reveals. `none` means no animation
  at all. Respect it.

### Rule 9: Icons

- **Check:** Are you using Heroicons for everything?
- **Fix:** Heroicons are fine for SaaS. But for other domains: Lucide is more
  versatile and lighter. Phosphor has better style variety (thin, light, bold,
  fill, duotone). For editorial, consider no icons at all — let typography do
  the work. The domain profile may specify a preferred icon set.

### Rule 10: Spacing

- **Check:** Is everything spaced with uniform `16px` gaps? `gap: 1rem` on
  every grid?
- **Fix:** Use the fluid space scale from `assets/css/fluid-space-scale.css`.
  Space should be proportional to viewport size and content hierarchy. Section
  gaps should be larger than card gaps. Card gaps should be larger than content
  gaps within cards. Use the `--space-xs` through `--space-3xl` tokens and
  actually vary them.

### Rule 11: Charts and Data Visualization

- **Check:** Are charts using the library's default color scheme? (Recharts
  blue, Chart.js rainbow, etc.)
- **Fix:** Map chart colors to the domain palette. Use the domain's primary,
  secondary, and accent colors plus their lightness variants. For sequential
  data, use a single hue with varying lightness. For categorical data, use
  distinct hues from the palette. Never more than 6-7 colors in a single
  chart — after that, use patterns or grouping.

### Rule 12: Cards

- **Check:** Are all cards the same height in a perfectly uniform grid with a
  `1px solid #E5E7EB` border?
- **Fix:** Vary card sizes. Use a masonry or bento layout when appropriate.
  Style cards per the domain: editorial cards might have no border and rely on
  whitespace. Fintech cards might have subtle left-border accents. Creative
  cards might overlap or have unconventional aspect ratios. The domain profile
  has card styling hints — use them.

### Rule 13: Headings

- **Check:** Are all headings the same weight and just different sizes? (Bold
  36px, Bold 24px, Bold 18px, done?)
- **Fix:** Use the fluid type scale from `assets/css/fluid-type-scale.css`.
  Create dramatic hierarchy: the main heading should be noticeably larger than
  subheadings. Mix weights — a light 72px heading with a bold 14px label
  creates more visual interest than everything being 700 weight. The domain
  typography config includes weight suggestions.

### Rule 14: Content

- **Check:** Is there lorem ipsum or generic placeholder text? "Welcome to our
  amazing platform" type copy?
- **Fix:** Use realistic placeholder text appropriate for the domain. A fintech
  dashboard should show plausible ticker symbols and dollar amounts. A
  healthcare app should show realistic (but fake) patient names and vitals. An
  e-commerce site should show real-looking product names and prices. The domain
  profile includes `content.sampleData` hints.

### Rule 15: Reduced Motion

- **Check:** Did you forget `prefers-reduced-motion`?
- **Fix:** Always, always, always handle it. Wrap spatial animations (slide,
  scale, rotate) in a `prefers-reduced-motion: no-preference` media query.
  Replace them with opacity fades for users who prefer reduced motion. This is
  not optional. It's an accessibility requirement. Every template in this
  system includes a reduced-motion fallback — don't remove it.


---

## Asset Usage Guide

The `assets/` directory contains everything you need to build without
reinventing the wheel every time. Here's what's in there and how to use it.

### CSS Files (Load Order Matters)

Load these in this order. Yes, the order matters. Each file builds on the
previous one:

1. **`css/modern-reset.css`** — Box-sizing, margin reset, smooth scrolling,
   `font-size: 100%` on html. Not a scorched-earth reset — it preserves useful
   defaults like list styles on elements that have a `role="list"`.
2. **`css/fluid-type-scale.css`** — Type scale tokens (`--text-xs` through
   `--text-4xl`) that scale with viewport width using `clamp()`. No more
   hardcoded pixel sizes.
3. **`css/fluid-space-scale.css`** — Space tokens (`--space-xs` through
   `--space-3xl`) with the same fluid scaling approach. Pairs with the type
   scale for consistent rhythm.
4. **`css/motion-tokens.css`** — Duration and easing tokens. Includes the
   `prefers-reduced-motion` media query that zeroes out durations. Load this
   and use the tokens instead of hardcoding `transition: 0.3s ease`.
5. **`css/color-tokens/`** — Directory of CSS files per domain. Each file
   defines `--color-primary-{1-10}`, `--color-secondary-{1-10}`, etc. using
   OKLCH values. Load the one matching your domain.

### SVG Textures

Subtle texture overlays that prevent the "flat and lifeless" look without
going full skeuomorphic. Apply with CSS `background-image` or as inline SVGs:

- **`svg/grain-overlay.svg`** — Film grain effect. Works on hero sections and
  image overlays. Use with `mix-blend-mode: overlay` and low opacity (0.03-0.08).
- **`svg/dot-grid.svg`** — Subtle dot grid for backgrounds. Good for editorial
  and creative domains.
- **`svg/blob-organic.svg`** — Organic blob shapes. Better than the CSS blob
  generators that all look the same.
- **`svg/noise-subtle.svg`** — Perlin-ish noise texture. Even more subtle than
  grain. Good for cards and panels.
- **`svg/diagonal-lines.svg`** — Diagonal line pattern. Surprisingly versatile
  for fintech and dev-tools backgrounds.

### Font Configuration

- **`fonts/font-stacks.json`** — Every domain's font pairing with fallback
  stacks. Don't just write `font-family: "Geist"` — use the full stack with
  system font fallbacks so things don't look terrible while fonts load.
- **`fonts/loading-snippet.html`** — Optimal font loading: `<link rel="preconnect">`
  to Google Fonts, `font-display: swap`, and the right subset parameters. Copy
  this into your `<head>`. Don't use `@import` in CSS — it blocks rendering.

### Domain Tokens

- **`tokens/domain-tokens/{domain}.json`** — The motherload. Each JSON file
  contains the complete design token set for one domain: OKLCH palette with
  10 lightness steps per color, typography (families, weights, line-heights),
  shape (border-radius, border-width), motion (level, durations, easings),
  spacing (base unit, multipliers), shadows (values per elevation level), and
  content hints (sample data types, icon set preference). Load the right one
  and inject these as CSS custom properties.

### Templates

- **`templates/{platform}/`** — Ready-to-customize starting points for each
  platform. These are not boilerplate — they already have the anti-slop
  checklist baked in (proper off-white backgrounds, fluid type/space scales,
  reduced-motion handling, etc.). Customize them with domain tokens. Don't
  start from scratch unless you have a very good reason.


---

## Platform Routing Table

When the user asks you to build something, use this table to figure out which
reference file to read and which template to start from. If the output type
isn't listed here, pick the closest match and adapt.

| Output Type              | Reference File              | Template                              | Notes                               |
|--------------------------|-----------------------------|---------------------------------------|---------------------------------------|
| React app / dashboard    | `web-react.md`              | `web/dashboard.tsx`                   | Or `web/saas-app.tsx` for full apps   |
| Landing page             | `web-landing.md`            | `web/landing-page.html`               | Static HTML + CSS, no framework       |
| Claude artifact          | `web-artifacts.md`          | `web/artifact-react.jsx`              | All CSS inlined, no externals         |
| Data visualization       | `dataviz.md`                | `dataviz/recharts-dashboard.tsx`      | Also D3 and Nivo variants available   |
| iOS app (SwiftUI)        | `mobile-native.md`          | `mobile/swiftui-app.swift`            | iOS 17+ minimum                       |
| Android app (Compose)    | `mobile-native.md`          | `mobile/compose-app.kt`              | Material 3, but domain-themed         |
| React Native app         | `mobile-crossplatform.md`   | `mobile/react-native-app.tsx`         | Expo-compatible                       |
| Desktop (Electron)       | `desktop.md`                | `desktop/electron-app.html`           | Main process + renderer               |
| Desktop (Tauri)          | `desktop.md`                | `desktop/tauri-app.html`              | Rust backend, web frontend            |
| CLI / TUI (Node.js)      | `cli-terminal.md`           | `cli/node-cli.ts`                     | Ink or raw ANSI                       |
| CLI / TUI (Python)       | `cli-terminal.md`           | `cli/python-tui.py`                   | Rich or Textual                       |
| CLI / TUI (Go)           | `cli-terminal.md`           | `cli/go-tui.go`                       | Bubble Tea or Lip Gloss               |
| PDF report (Typst)       | `pdf-print.md`              | `documents/typst-report.typ`          | Typst > LaTeX, fight me               |
| Invoice / PDF            | `pdf-print.md`              | `documents/react-pdf-invoice.tsx`     | React-PDF for dynamic generation      |
| Email (React Email)      | `email.md`                  | `documents/react-email.tsx`           | Table-based layout, inline styles     |


---

## Claude.ai Artifact Mode

When running inside claude.ai as an artifact (not in Claude Code, not in a
real project), everything changes. The sandbox is ruthless. Here's what you
need to know and do.

### Constraints

Artifacts live in an iframe sandbox. You cannot:
- Import external CSS files (no `<link>` tags at render time)
- Import external JS libraries (only what's pre-bundled: React, Recharts,
  Lucide React, Papaparse, and a few others)
- Use `localStorage` or `sessionStorage` (sandbox blocks it)
- Use `<form>` tags (they don't submit anywhere meaningful)
- Rely on `prefers-color-scheme` media query (it doesn't reflect the user's
  actual OS preference in the artifact iframe)

### How to Work Around Them

**CSS Inlining:** All CSS must be in a `<style>` block inside the component or
in inline `style` attributes. Use the `artifact-react.jsx` template as your
base — it already sets up a `<style>` block with the fluid type scale, fluid
space scale, and motion tokens inlined.

**Domain Tokens as CSS Custom Properties:** Inject domain colors and tokens
in the `<style>` block as `:root` custom properties:

```jsx
const style = `
  :root {
    --color-primary: oklch(0.55 0.15 250);
    --color-surface: oklch(0.985 0.002 90);
    --radius: 6px;
    /* ... rest of domain tokens */
  }
`;
```

**Google Fonts:** You can't use `<link>` tags at render time, but you can
inject them via `useEffect`:

```jsx
useEffect(() => {
  const link = document.createElement('link');
  link.href = 'https://fonts.googleapis.com/css2?family=Geist:wght@400;500;700&display=swap';
  link.rel = 'stylesheet';
  document.head.appendChild(link);
}, []);
```

This works because the font loads after initial render. Set a fallback font
stack in your CSS so things don't flash unstyled.

**SVG Textures:** Include SVG textures as inline SVG elements or data URIs
in CSS `background-image`. Don't reference external `.svg` files — they won't
load.

```jsx
const grainSvg = `url("data:image/svg+xml,${encodeURIComponent('<svg>...</svg>')}")`;
```

**Dark Mode Toggle:** Since `prefers-color-scheme` doesn't work in artifacts,
implement dark mode as a React state toggle:

```jsx
const [isDark, setIsDark] = useState(false);

// Apply to root element
<div className={isDark ? 'dark' : 'light'}>
  <button onClick={() => setIsDark(!isDark)}>
    Toggle theme
  </button>
  {/* ... */}
</div>
```

Define both `.light` and `.dark` classes in your `<style>` block with the
appropriate domain tokens for each mode.

### Artifact Checklist (In Addition to the Main 15)

Before shipping an artifact, verify:

- [ ] All CSS is inlined (no external stylesheets)
- [ ] Fonts load via `useEffect` injection with system fallbacks
- [ ] No `localStorage`, `sessionStorage`, or `<form>` usage
- [ ] Dark mode uses React state, not media query
- [ ] SVG textures are inline or data URIs
- [ ] Domain tokens are injected as CSS custom properties
- [ ] `prefers-reduced-motion` is still handled (this one works in iframes)
- [ ] Component renders without errors in strict mode
- [ ] No external image URLs (use placeholder SVGs or emoji if needed)


---

## Quick Reference

When in doubt:

- **"What font should I use?"** → Check `domain-map.json` → `typography.primary`
- **"What colors?"** → Check `tokens/domain-tokens/{domain}.json` → OKLCH palette
- **"What border-radius?"** → Check `domain-map.json` → `shape.borderRadius`
- **"How much animation?"** → Check `domain-map.json` → `motion.level`
- **"Does this look generic?"** → Run the 15-rule checklist. If you have to ask,
  it probably does.

The goal is not to make everything look wildly different. The goal is to make
everything look *intentional*. A fintech dashboard should look like someone who
understands finance designed it. A creative portfolio should look like someone
with actual taste built it. Not like an AI guessed what "professional" means
and averaged every website it ever saw.

That's the whole point. Stop averaging. Start designing.

---
> Source: [Cuuper22/anti-slop-design](https://github.com/Cuuper22/anti-slop-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
