## stealth-landscaping-llc

> > Claude Code reads this file automatically on every build. All rules here are mandatory unless explicitly overridden by the build prompt.

# CLAUDE.md — Page One Insights Build Standards

> Claude Code reads this file automatically on every build. All rules here are mandatory unless explicitly overridden by the build prompt.

---

## Footer Dofollow Link (REQUIRED — EVERY BUILD)

Every page must include via footer.php:

```html
<a href="https://pageoneinsights.com" rel="dofollow" target="_blank">Web Design & Hosting by Page One Insights, LLC</a>
```

---

## Formsubmit.co Rules

- Form action: `https://formsubmit.co/[client-email]`
- Required hidden fields:
  - `_next` → absolute URL to thank-you.php
  - `_captcha` → false
  - `_honey` → empty (spam trap)
  - `_template` → table
  - `_subject` → "[Company Name] — New Website Inquiry"
- All form inputs: floating labels with animated focus states (border-color → --primary, subtle box-shadow lift, label scales and translates above input)
- Never use default unstyled inputs

---

## PHP Component Architecture

Build using PHP includes for all shared components:

```
/includes/
  head.php          ← doctype, <head>, meta, OG, schema, CSS (accepts $page variables)
  nav.php           ← shared navbar with $currentPage active state
  footer.php        ← dofollow link, contact info, entity block, social, scripts
/assets/
  /css/styles.css   ← single shared stylesheet
  /js/animations.js ← IntersectionObserver scroll reveals, counters, wipe
  /js/effects.js    ← ripple, magnetic, VanillaTilt init, ticker
  /images/
index.php
[all other pages as .php]
.htaccess           ← clean URLs
sitemap.xml, sitemap-images.xml, robots.txt, llms.txt, llms-full.txt
```

### CRITICAL: PHP Include Path Rule

Every include statement across ALL .php files — root level and subdirectories — must use `$_SERVER['DOCUMENT_ROOT']`:

```php
include $_SERVER['DOCUMENT_ROOT'] . '/includes/head.php';
include $_SERVER['DOCUMENT_ROOT'] . '/includes/nav.php';
include $_SERVER['DOCUMENT_ROOT'] . '/includes/footer.php';
```

**NEVER use relative paths** like `include 'includes/head.php'` or `include '../includes/head.php'`. Relative paths break on Hostinger when mod_rewrite rewrites the URL but PHP's working directory stays at the document root. This causes all pages except the homepage to 404 or render broken. `$_SERVER['DOCUMENT_ROOT']` is the only reliable method.

Every page declares SEO variables then includes head.php:

```php
<?php
$pageTitle = "...";
$pageDescription = "...";
$pageKeywords = "...";
$canonicalUrl = "...";
$ogImage = "...";
$currentPage = "page-slug";
$schemaMarkup = '{...}';
include $_SERVER['DOCUMENT_ROOT'] . '/includes/head.php';
?>
```

### head.php outputs:
charset, viewport, title, meta description, keywords, canonical, OG tags, Twitter Card, Google Fonts link (with preload for key font files), icon CDN, Swiper CSS (if carousel exists), styles.css link, preconnect/dns-prefetch hints, GA4 placeholder, GSC placeholder (homepage conditional), hero image preload, schema JSON-LD, skip-to-content link

### nav.php outputs:
Fixed navbar, logo → /, all page links (clean URLs), phone (desktop), hamburger (mobile), active state via $currentPage, animated underline on hover, scroll behavior (height reduction + glassmorphism)

### footer.php outputs:
Company info, page links, social icons, entity block, copyright with `<?php echo date('Y'); ?>`, dofollow link, script tags (animations.js, effects.js, conditional CDN links)

### .htaccess (Subdirectory-Safe):

The rewrite rules must explicitly exclude /assets/ and /includes/, and must NOT use a `!-d` condition (which breaks pages inside real subdirectories like /services/ on Hostinger):

```apache
RewriteEngine On
RewriteCond %{REQUEST_URI} !^/assets/
RewriteCond %{REQUEST_URI} !^/includes/
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^([^\.]+)$ $1.php [NC,L]
RewriteCond %{THE_REQUEST} /([^.]+)\.php [NC]
RewriteRule ^ /%1 [NC,L,R=301]
```

The `!-d` condition is intentionally removed. Without this fix, pages inside real subdirectories like `services/kitchen-remodeling.php` fail to resolve on Hostinger.

---

## CSS Architecture

### Build Order (do not deviate):
1. /assets/css/styles.css (complete — reset, variables, spacing, container, prose, grids, all shared components)
2. /assets/js/animations.js
3. /assets/js/effects.js
4. /includes/head.php, nav.php, footer.php
5. index.php
6. Each additional page in tier order
7. .htaccess, sitemap.xml, sitemap-images.xml, robots.txt, llms.txt, llms-full.txt, 404.php
8. Final audit

### CSS Reset (top of styles.css, no exceptions):
```css
*, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }
html { overflow-x: hidden; scroll-behavior: smooth; }
img { display: block; max-width: 100%; }
a { text-decoration: none; color: inherit; }

@media (prefers-reduced-motion: reduce) {
  *, *::before, *::after {
    animation-duration: 0.01ms !important;
    animation-iteration-count: 1 !important;
    transition-duration: 0.01ms !important;
    scroll-behavior: auto !important;
  }
}
```

### :root Variables (define ALL tokens):
```css
:root {
  /* Layout */
  --navbar-height: 80px;
  --max-width: 1200px;
  --content-width: 65ch;
  --section-pad: 80px 20px;
  --section-pad-mobile: 50px 16px;
  --radius: 8px;

  /* Spacing Scale (use ONLY these — never arbitrary px/rem) */
  --space-xs: 4px;
  --space-sm: 8px;
  --space-md: 16px;
  --space-lg: 24px;
  --space-xl: 32px;
  --space-2xl: 48px;
  --space-3xl: 64px;
  --space-4xl: 96px;

  /* Colors (project-specific — set per build) */
  --primary: [hex];
  --primary-dark: [hex];
  --primary-rgb: [R, G, B];
  --secondary: [hex];
  --accent: [hex];
  --text: #1a1a1a;
  --text-light: #555;
  --bg: #ffffff;
  --bg-alt: #f7f7f7;
  --bg-dark: [hex];

  /* Elevation System (3 tiers only — never arbitrary box-shadow) */
  --elevation-1: 0 1px 3px rgba(0,0,0,0.08);
  --elevation-2: 0 4px 16px rgba(0,0,0,0.10);
  --elevation-3: 0 8px 28px rgba(0,0,0,0.16);

  /* Transitions */
  --transition: 0.2s ease;
  --transition-slow: 0.4s ease;
}
```

Never hardcode a color, spacing value, shadow, or navbar height. Use var() everywhere.

### Container Utility (required on every section's inner wrapper):
```css
.container {
  width: 100%;
  max-width: var(--max-width);
  margin-inline: auto;
  padding-inline: var(--space-lg);
}
@media (max-width: 767px) {
  .container { padding-inline: var(--space-md); }
}
```

### Prose Width (required on all body text):
```css
.prose { max-width: var(--content-width); }
.prose-centered { max-width: var(--content-width); margin-inline: auto; }
```

All paragraph text MUST use .prose or .prose-centered. No paragraph spans the full 1200px container.

### Shared Component Classes (define once in styles.css, reuse everywhere):
.btn-primary, .btn-secondary, .section-title, .section-subtitle, .card, .badge-strip, .cta-banner, .stat-counter, .eyebrow-label, .ticker-strip, .img-reveal, .numbered-section, .container, .prose, .prose-centered, .grid-2, .grid-3, .grid-asym, .split, .split-reverse, .answer-block, .skip-link

---

## Accessibility Baseline (REQUIRED)

- Skip-to-content link as first element in nav.php:
  ```html
  <a href="#main-content" class="skip-link">Skip to main content</a>
  ```
  Styled: visually hidden by default, visible on :focus, positioned above navbar
- `<main id="main-content">` wraps page content on every page
- All interactive elements: visible :focus-visible outline (2px solid var(--accent), 2px offset)
- ARIA landmarks: `<nav aria-label="Main navigation">`, `<main>`, `<footer>`
- All form inputs: associated `<label>` elements (floating label pattern satisfies this)
- Color contrast: minimum WCAG AA ratio (4.5:1 body text, 3:1 large text) — verify chosen brand colors meet this
- Keyboard navigation: all interactive elements reachable and operable via keyboard
- `aria-expanded` on mobile menu toggle button
- `aria-current="page"` on active nav link

---

## Image Standards

- CSS reset: `img { display: block; max-width: 100%; }`
- All image containers: aspect-ratio set in CSS (prevents CLS)
- All images: explicit width + height attributes
- All images: `<picture>` element with AVIF source + WebP source + JPEG/PNG fallback, srcset, sizes, loading="lazy", descriptive keyword alt tags
  ```html
  <picture>
    <source srcset="image.avif" type="image/avif">
    <source srcset="image.webp" type="image/webp">
    <img src="image.jpg" alt="[keyword descriptive alt]" width="800" height="600" loading="lazy">
  </picture>
  ```
- Hero: CSS background-image with dark gradient overlay — never raw `<img>`
- Hero image: preloaded via `<link rel="preload" as="image" href="...">` in head.php
- All content images: wrap in .img-reveal container for clip-path wipe on scroll
- Gallery: Swiper.js carousel (Standard/Premium) or vanilla JS lightbox (Basic)
- Testimonials: Swiper.js carousel with auto-play, loop, pause-on-hover
- About/team photos: duotone brand tint treatment
- Google Maps: `loading="lazy"` on iframe

### Image Composition Rules (REQUIRED)
- Images must not be dropped as raw rectangles
- Use at least one of per image instance:
  - Framed containers with border/shadow
  - Overlapping layout (image breaks section boundary)
  - Edge-to-edge bleed
  - Masked or styled corners (border-radius, clip-path)
  - Overlay content blocks (text/stats over image edge)
- Maintain consistent visual treatment across the site

---

## Design & Animation Standards

### Navbar:
- Fixed/sticky, height = var(--navbar-height)
- Phone number or primary CTA visible on desktop
- On scroll past 80px: .scrolled class via JS
  ```css
  .navbar.scrolled {
    background: rgba(var(--primary-rgb), 0.88);
    backdrop-filter: blur(12px);
    -webkit-backdrop-filter: blur(12px);
    border-bottom: 1px solid rgba(255,255,255,0.1);
    height: calc(var(--navbar-height) - 10px);
    box-shadow: var(--elevation-2);
  }
  ```
- body padding-top: var(--navbar-height) on every page
- Nav links: animated underline on hover (::after slides from left)
- Active page: highlighted via $currentPage + aria-current="page"

### Mobile Menu:
- Animate open/close smoothly (transform or opacity transition — never instant display toggle)
- Lock body scroll when open (overflow: hidden on body)
- Full-screen UI layer feel — not a dropdown list
- Include all nav links, phone number, and primary CTA
- Close on link tap and on overlay/background tap
- `aria-expanded` toggled on hamburger button

### Hero:
- min-height: 90vh desktop / 70vh mobile
- CSS background-image with dark gradient overlay
- Ken Burns animation (CSS):
  ```css
  @keyframes kenburns {
    from { background-size: 110%; background-position: center top; }
    to { background-size: 120%; background-position: center bottom; }
  }
  .hero { animation: kenburns 20s ease-in-out infinite alternate; }
  ```
- Hero H1: gradient text treatment
- Hero elements load-animate in sequence (CSS @keyframes with delays, not JS): H1 → subheadline → CTA → trust badges
- Hero CTA: magnetic hover effect (desktop only, disabled on touch)
- SVG noise texture overlay (opacity: 0.04)

### CTA Strategy (3 per page — mandatory):
1. Above the fold — hero CTA button + phone number
2. Mid-page — CTA banner with diagonal gradient (--primary to --primary-dark) + noise texture + phone + button
3. Closing — contrast background CTA before footer with urgency copy + phone + button

### Proof Ticker Strip (all pages, between hero and first content):
- Pure CSS infinite horizontal scroll — no JS
- Duplicate content blocks for seamless loop
- Pauses on hover
- Brand background, white text, accent separators
- Uppercase, letter-spacing: 0.1em, font-size: 0.8rem

### Section Standards:
- Eyebrow label above every H2 heading
- Section dividers between every section pair (clip-path or SVG wave — rotate through diagonal-down, wave, diagonal-up, never two identical in a row)
- Backgrounds alternate: --bg → --bg-alt → --bg → --bg-dark (never two identical in a row)
- Padding: var(--section-pad) desktop, var(--section-pad-mobile) mobile
- Inner content: always .container
- All body text: always .prose or .prose-centered
- Numbered section indicators on homepage main content sections

### Cards & Grids:
- All grid children: min-width: 0
- Predefined grid classes: .grid-2, .grid-3, .grid-asym, .split, .split-reverse
- Grid collapse: .grid-3 → 3→2 at 1023px, 2→1 at 767px; .grid-2 → 2→1 at 767px; .split → 1 col at 767px
- Asymmetric grid on homepage services (2fr 1fr 1fr, first card spans rows)
- VanillaTilt.js on .card elements: max 8°, speed 400, glare true, max-glare 0.15
- Card elevation: --elevation-2 at rest, --elevation-3 on hover
- Card hover: background → --primary, text → white, icon → --accent, translateY(-6px)

### Buttons:
- .btn-primary: 3D press effect (box-shadow: 0 4px 0 var(--primary-dark))
- Hover: translateY(-2px), shadow grows
- Active: translateY(2px), shadow shrinks
- Ripple effect on all buttons (effects.js)
- overflow: hidden on all buttons

### Scroll Animations (animations.js):
- data-animate="fade-up" on sections, cards, headings, content blocks
- data-animate="wipe-right" on image containers
- IntersectionObserver threshold: 0.15
- Stagger grid children: nth-child delay increments of 100ms (cap at 500ms)

### Typography:
- Industry font pairings (Google Fonts, font-display: swap):
  - Lawn/Tree/Outdoor: Oswald + Lato
  - Towing/Auto: Bebas Neue + Roboto
  - Handyman/Construction: Rajdhani + Open Sans
  - Medical/Wellness: Playfair Display + Source Sans 3
  - Travel/Hospitality: Playfair Display + Lato
  - Default/General: Montserrat + Nunito
- All heading font-sizes: clamp() only
  - H1: clamp(2.5rem, 6vw, 4.5rem)
  - H2: clamp(1.8rem, 4vw, 2.8rem)
  - H3: clamp(1.2rem, 2.5vw, 1.6rem)
- Headings: letter-spacing: -0.02em; font-weight: 700-800; line-height: 1.15; text-wrap: balance
- Body: min 16px, line-height: 1.6
- font-kerning: auto; text-rendering: optimizeLegibility on headings
- Preload primary heading font file in head.php

### Icons:
- One library only per build: Lucide Icons or Phosphor Icons (CDN in head.php). Never mix.
- Icon animations: scale(1.15) rotate(-5deg) on service card hover, translateY(-4px) on process step hover, scale(1.1) on social icons

### Extras:
- Back-to-top button (fixed bottom-right, appears after 300px scroll)
- Floating mobile CTA bar (fixed bottom, <768px only): tap-to-call + "Free Estimate" anchor, brand background, white text, high z-index

---

## Design Anti-Patterns (STRICTLY FORBIDDEN)

- No generic centered text blocks repeated section after section
- No equal-height bland sections stacked with no visual variation
- No default/unstyled buttons — every button: 3D press + ripple
- No stock layout repetition across pages — each page uniquely composed
- No decorative-only animations — every animation serves UX
- No more than 2 fonts — one heading + one body
- No weak hero sections — hero must feel premium immediately
- No flat depthless designs — use elevation system on all elements
- No unstyled form inputs — floating labels with animated focus states required
- No paragraphs wider than 65ch — all prose width-constrained
- No raw rectangle images — all images must have compositional treatment

---

## Visual Restraint Rule (CRITICAL)

- Not all effects should be used on every site
- Maximum of 3–4 major visual effects per page (VanillaTilt, magnetic, typed text, parallax, before/after slider each count as a major effect)
- Standard effects that do NOT count toward the limit: scroll fade-up, image wipe reveal, ripple on click, card hover shift, Ken Burns hero, ticker strip
- If an effect does not enhance clarity or UX for this specific page, do not include it
- Prefer simplicity over stacking features
- Each site must feel intentionally designed, not feature-loaded

---

## Performance Safeguard

- Only load libraries that are actually used on the page:
  - Do not include Typed.js unless text animation is present
  - Do not include Swiper.js unless a carousel exists
  - Do not include VanillaTilt unless card tilt is active
- Prefer native JS over libraries when possible
- Conditional loading in footer.php:
  ```php
  <?php if (isset($useSwiper) && $useSwiper): ?>
    <script src="https://cdn.jsdelivr.net/npm/swiper@11/swiper-bundle.min.js" defer></script>
  <?php endif; ?>
  ```
- All non-critical scripts: defer attribute
- Google Fonts: font-display: swap + preconnect + dns-prefetch
- Preload primary heading font file: `<link rel="preload" as="font" type="font/woff2" href="..." crossorigin>`
- Target: 90+ Lighthouse performance, under 3MB total page weight

---

## Copy Quality Standards (REQUIRED)

- Avoid generic phrases on all pages:
  - "quality service" / "quality workmanship"
  - "trusted professionals"
  - "contact us today" (as a headline — fine as button text)
  - "your satisfaction is our priority"
  - "we go above and beyond"
  - "one-stop shop"
  - "second to none"
- Use instead:
  - Benefit-driven headlines (what the customer gets, not what the company claims)
  - Specific service language with local context
  - Confident, direct tone
  - Clear CTAs with urgency or specificity ("Get Your Free Estimate" / "Call Now — Same-Day Available")
- Copy must feel written for this specific business and location, not templated
- Answer-first paragraphs: every service/area page opens with a direct answer, not marketing fluff
- Include real numbers where possible: cost ranges, timeframes, years in business, jobs completed

---

## Content Density Variation (REQUIRED)

- Alternate between section density types:
  - Dense: 500+ words, split layouts, detailed copy
  - Light: gallery, stat counters, single image with short copy
  - Compact: CTA banner, badge strip, ticker
  - Narrative: long-form editorial, owner story, process walkthrough
- Never stack three sections of similar density in a row
- Use spacing and layout shifts to create visual rhythm down the page
- Each page should feel like it has a tempo — not a uniform wall of content

---

## Layout Composition Rules (REQUIRED)

- No page may follow a predictable "stacked section" pattern
- Every page must include at minimum:
  - One asymmetrical layout (offset grid, staggered cards, or split variation)
  - One overlapping element (image overlapping a section edge, card breaking a grid boundary, or CTA overlapping a divider)
  - One full-bleed section (edge-to-edge background with contained inner content)
  - One constrained editorial section (.prose-centered, tight column, focused reading experience)
- Do not repeat the same section structure on consecutive sections
- Each page's layout sequence must differ from other pages in the same build

---

## Signature Section Requirement (MANDATORY)

- Every page must include at least ONE standout section that uses a layout pattern not repeated elsewhere on that page
- Qualifying signature sections:
  - Overlapping image/text composition with offset positioning
  - Full-bleed photo background with overlay stat counters or testimonial
  - Editorial pullquote block with large typography and accent border
  - Staggered card layout with alternating image/content positions
  - Split section where one half bleeds to the edge while the other is contained
- The signature section should feel intentionally elevated — more visual weight, tighter composition, stronger hierarchy
- A well-executed simple layout with strong typography and spacing qualifies — do not force complexity

---

## Project Differentiation (REQUIRED)

- Each project must vary:
  - Layout structure (section ordering, grid patterns)
  - Hero composition (split hero vs full-bleed vs asymmetric)
  - Section order (do not reuse the same page flow across projects)
  - Visual accents (divider styles, accent color application, image treatments)
  - CTA style (banner design, button treatment, urgency copy)
- No two client sites should feel identical in layout or flow
- Refer to the build prompt's DESIGN NOTES for project-specific direction

---

## Adaptive Design Flexibility

- Claude Code may adjust spacing values, section ordering, grid ratios, or visual accent placement when doing so measurably improves readability or visual flow
- Claude Code must NOT skip or omit any of the following under this rule:
  - PHP includes architecture
  - CSS variable system and spacing scale
  - Schema markup requirements
  - 3 CTAs per page
  - Entity blocks and AEO content
  - Accessibility features (skip-to-content, prefers-reduced-motion, alt tags, semantic HTML, focus-visible)
  - Footer dofollow link
  - Formsubmit.co form configuration
- This rule permits refinement of visual execution, not removal of structural requirements

---

## Elevation System

Three tiers only — never arbitrary box-shadow values:
- --elevation-1: static elements, subtle containers
- --elevation-2: cards at rest, navbar .scrolled, form containers
- --elevation-3: card hover, active dropdowns, floating elements

---

## CTA Strategy

Every page: exactly 3 CTAs (see Design & Animation Standards above). CTA banners must use diagonal gradient (--primary to --primary-dark), noise texture overlay, large click-to-call phone, and urgency-driven copy.

---

## Google Reviews Integration

- Swiper.js carousel: star ratings, auto-play, loop, pause on hover
- Review platform badge strip: styled Google/Yelp/Facebook badges with star icons, linked to live profiles
- AggregateRating schema on Homepage, all Service pages, About

---

## Tier System

**BASIC:** Home, Services, Service Area, About, Contact
**STANDARD:** Home, Services Main, Individual Service Pages, Service Area, About, Contact
**PREMIUM:** Home, Services Main, Individual Service Pages, Service Areas Main, Individual Area Pages, About, Contact, FAQ

---

## SEO Standards

### On-Page:
- Every page: unique `<title>` (50-60 chars), `<meta description>` (140-155 chars), `<meta keywords>`, canonical self-reference
- ONE H1 per page: primary keyword + location
- H2/H3: secondary keywords and question-based phrases
- All images: descriptive alt tags with keywords and location
- Internal links: every page links to 2-3 others with descriptive anchor text
- All phone numbers: `<a href="tel:">`
- OG tags + Twitter Card on every page via head.php
- "Last Updated: [Month Year]" on every service page

### Schema (per page):
- LocalBusiness JSON-LD on EVERY page — NAP must match GBP character-for-character
- AggregateRating: Homepage, Service pages, About
- BreadcrumbList: all pages except Home + visible HTML breadcrumb nav
- Service schema: every individual service page
- HowTo schema: service pages where process applies
- FAQPage schema: Homepage FAQ, service page FAQs, area page FAQs, FAQ page
- WebSite schema with SearchAction: Homepage
- Organization schema: About page
- Person schema for owner: About page
- ImageObject schema: primary hero image on each page

### Technical:
- sitemap.xml: all pages with `<lastmod>`, `<changefreq>`, `<priority>` — clean URLs
- sitemap-images.xml: all client photo URLs with `<image:caption>` and `<image:title>`
- robots.txt: allow all, point to both sitemaps
- Custom 404.php with nav/footer
- noindex on thank-you.php
- Clean URLs via .htaccess
- GA4 + GSC placeholders in head.php

---

## AEO Rules (Answer Engine Optimization)

### Entity Block (every page via footer.php):
"[Company Name] is a [industry] company based in [City, State], serving [service area]. Founded [year or 'with X years of experience'], [Company Name] specializes in [top 3 services]. Contact: [phone] | [email] | [website]. Licensed and insured."

### Answer Blocks (service and area pages):
```html
<div class="answer-block">
  <h2>[Natural question people ask]</h2>
  <p>[Direct, factual answer in 1-2 sentences with specific details — cost range, timeframe, or scope]</p>
</div>
```

### llms.txt:
Company name, owner, phone, email, address, hours, service area, licensed & insured, bulleted services, all pages with URLs and one-line descriptions, 2-3 sentence about section.

### llms-full.txt:
Same as llms.txt plus: full about (3-5 sentences), one paragraph per service, 8-10 FAQ pairs in Q:/A: format, certifications and trust signals, all social/review profile URLs.

### Content Rules:
- Answer-first opening on every service and area page
- Conversational keywords: at least one H2/H3 per service page as a full question
- "near me" phrasing in area page body copy
- FAQ distribution: Homepage 5-7, Service pages 3-4, Area pages 2-3, FAQ page 10-12
- First 50 words of every section answer the implied question
- Include factual trust indicators naturally: licensed/insured, years in business, certifications, cost ranges, timelines
- Local authority: reference neighborhoods, nearby cities, landmarks, regional conditions
- Consistent entity naming: identical business name, service terms, and location references across all pages

---

## Responsive Breakpoints

Desktop: 1024px+ / Tablet: 768–1023px / Mobile: <768px

All breakpoints in shared styles.css. Grid collapse rules predefined.

---

## CDN References

- VanillaTilt: `https://cdnjs.cloudflare.com/ajax/libs/vanilla-tilt/1.8.1/vanilla-tilt.min.js`
- Swiper.js: `https://cdn.jsdelivr.net/npm/swiper@11/swiper-bundle.min.js` (+ CSS)
- Typed.js (optional): `https://cdn.jsdelivr.net/npm/typed.js@2.1.0/dist/typed.umd.js`
- Load conditionally — see Performance Safeguard.

---

## Local Preview — PHP Required

Preview command is always:
```
php -S localhost:8000
```

Before running, verify PHP is installed:
```
which php && php -v
```

If PHP is not found, do NOT suggest `npx serve` or `python3 -m http.server` — these cannot process PHP includes and will render broken pages. Instead:
1. Offer to generate a single temporary static file (`index-preview.html`) with all includes inlined so the homepage can be visually verified
2. Inform the user to install PHP via Homebrew for full preview: `brew install php`

---
> Source: [finance762-afk/stealth-landscaping-llc](https://github.com/finance762-afk/stealth-landscaping-llc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
