## claude-skill-landing-page

> Build complete, deployment-ready landing pages with an explicit frontend skill stack: Frontend Design for art direction, Design Taste Frontend for anti-slop implementation, Imagegen Frontend Web for section-level visual references when useful, and this skill for research, conversion structure, build, browser QA, and deployment. Use whenever the user asks for a landing page, sales page, campaign page, one-page marketing site, website redesign, or wants a URL/PDF/doc/brief converted into a web page. Trigger even when the user only asks to improve a landing page's frontend, imagery, responsiveness, or deployment.


# Landing Page Builder

Build high-quality, deployment-ready landing pages by extracting branding and content from client websites and documents, then producing a responsive HTML/CSS page with verified images and clean structure.

This skill exists because landing pages require careful coordination of branding, content, images, and responsive design — and the most common failure mode is treating source documents as text-only, missing the images that carry half the marketing message.

## The Golden Rules

**1. Images are content, not decoration.** When a marketing PDF shows a classroom photo next to a paragraph about student wellbeing, that photo IS the message. A landing page without those images is incomplete regardless of how good the copy is. Extract and integrate images in the first pass, every time.

**2. Keep styling maintainable within the chosen stack.** For a static page, put CSS in `styles.css`, not inline in HTML. For an existing framework project, preserve its established CSS modules, component styles, utility classes, or token system. Do not migrate styling architecture without a reason.

**3. Never invent content.** Use only what the source document provides. Do not make up image captions, visual directions, testimonial quotes, statistics, or client names. If content is missing, flag it and ask — do not fill gaps with AI-generated placeholder copy.

**4. Use the frontend stack deliberately.** Before coding, load the available supporting skills named below. They are complementary roles, not duplicate instructions to paste together blindly. If a named skill is unavailable, continue with the available stack and record the gap; never pretend it ran.

---

## Frontend Skill Stack (Required Routing)

The screenshot-provided frontend tools map to this landing-page workflow as follows. Read `references/frontend-skill-stack.md` for the complete routing and conflict rules.

1. **`frontend-design`** — establish the subject-specific visual thesis, typography, palette, layout concept, signature element, and motion intent. Use for new builds and meaningful visual reshaping.
2. **`design-taste-frontend`** — apply anti-slop implementation discipline, responsive layout mechanics, accessible interaction states, component choices, performance checks, and the final design pre-flight.
3. **`imagegen-frontend-web`** — use when generated visual references would materially reduce ambiguity: unknown visual direction, image-led campaigns, custom illustration/art direction, or a user asking for mockups. It generates references; it does not replace real brand assets or implementation.
4. **`design-taste-frontend-v1`** — legacy fallback only. Use it when an existing project explicitly depends on v1 behaviour; do not load v1 and current Design Taste together.
5. **`imagegen-frontend-mobile`** — not part of a website landing-page build. Use only for a separate mobile-app screen deliverable, never as the page's responsive-web design step.
6. **`Aidesigner Frontend`** — if this exact skill becomes installed and discoverable, use it as an optional second critique of the approved art direction. It is not currently assumed available and must never block the workflow.

Do not run every tool for ceremony. The minimum new-landing-page stack is `frontend-design` + current `design-taste-frontend` + `landing-page`. Add `imagegen-frontend-web` when visual-reference generation is justified. Responsive mobile web remains part of the implementation and QA; it is not delegated to the mobile-app image skill.

### Required pre-code design gate

Before implementation, produce an internal build brief containing:
- the page's audience, single conversion goal, and source-of-truth content;
- one coherent visual thesis tied to the actual subject, not a generic SaaS aesthetic;
- palette and typography roles with licensing/availability checked;
- section order and the job of every section;
- asset map: preserve, obtain, generate, or omit;
- one signature visual idea and a restrained motion plan;
- desktop, tablet, and mobile behaviour for the first fold and complex sections;
- explicit avoid-list based on the brand and reference audit.

If the user supplied a reference, extract principles without cloning its identity. If the source is an existing client site or approved template, preserve approved copy, brand assets, component grammar, and section order unless the user explicitly authorises a redesign.

---

## Anti-Slop Rules (MANDATORY)

These patterns make AI-generated pages look cheap and generic. NEVER use them:

### Banned Copy Patterns
- "Revolutionize", "Transform your", "Unleash the power of", "Leverage", "Empower"
- "Cutting-edge", "Next-generation", "State-of-the-art", "Best-in-class"
- "Seamlessly", "Effortlessly", "Supercharge"
- "In today's fast-paced world", "In an era of"
- Starting paragraphs with "Imagine..." or "Picture this..."
- Generic CTAs like "Learn More" or "Get Started" without context — use specific CTAs: "Get Instant Quote", "Book a Demo", "See Pricing"

### Banned Visual Patterns
- Gradient mesh blobs or aurora backgrounds (the #1 tell of AI-generated pages)
- Purple-to-blue gradients as primary design language
- Floating 3D objects with no connection to the product
- Generic abstract geometric patterns as hero backgrounds
- Overly rounded cards (border-radius > 1.5rem) with heavy drop shadows
- Identical card layouts repeated 3+ times with only icon changes
- Stock photo hero images that don't match the business (e.g., generic handshake photos)

### Banned Code Patterns
- Inline styles in HTML elements
- Using `!important` in CSS (except for utility overrides)
- Hardcoded pixel values for font sizes — use `clamp()` or rem
- Missing `alt` attributes on images
- Using `<div>` when semantic elements exist (`<section>`, `<article>`, `<nav>`, `<header>`, `<footer>`)

---

## Workflow Overview

The build follows seven phases. Resolve obvious facts from supplied sources instead of asking the user to repeat them. Ask only when a missing decision materially changes the result.

1. **Context & Research** — Read project/client instructions, meeting decisions, brief, approval state, live brand, references, copy, and design grammar
2. **Content Extraction** — Pull all approved text and images from supplied sources
3. **Asset Verification** — Build a provenance-aware inventory of usable media
4. **Frontend Direction & Blueprint** — Apply Frontend Design and Design Taste; lock conversion flow, tokens, responsive behaviour, assets, and acceptance checks; generate web references only when justified
5. **Build** — Use the simplest stack that satisfies the brief and implement complete production code
6. **Browser QA** — Build/test, inspect screenshots at desktop/tablet/mobile, fix defects, and repeat
7. **Deploy & Live Verify** — Deploy only when requested, then inspect the real URL and report the verified alias

---

## Phase 1: Context & Research

Before writing code, read the nearest project/client instructions, approved brief, meeting decisions, and source links. Then understand the client's existing brand and collect every asset you'll need.

### Brand Extraction

Fetch the client's website and extract:

- **Colors**: Inspect CSS custom properties, computed styles on key elements (headings, buttons, backgrounds, accents). Record as hex values.
- **Fonts**: Check Google Fonts links, `font-family` declarations, and `@font-face` rules. Do not assume — many sites use fonts that look like common ones but aren't.
- **Logo**: Find the logo image URL (usually in the header or footer). Get the highest-resolution version available.
- **Design language**: Note spacing patterns, border-radius usage, shadow styles, section background treatments.

### Asset Collection

This is critical. Crawl the site's image directories to build a complete catalog:

1. Fetch the homepage and key pages, extract all `<img>` `src` attributes and CSS `background-image` URLs
2. Look for patterns in the image paths (e.g., `/wp-content/uploads/YYYY/MM/` for WordPress sites)
3. For each image, record:
   - Full URL
   - Descriptive name (what does the image show?)
   - Dimensions (from the filename or by fetching headers)
   - Category (photo, illustration, icon, logo, sticker)

Save this catalog — you'll reference it throughout the build.

**Example catalog entry:**
```
URL: https://example.com/wp-content/uploads/2023/08/image-13-1024x768.jpeg
Description: Students working together in classroom
Dimensions: 1024x768
Category: photo
Use for: Problem section, Gallery
```

---

## Phase 2: Content Extraction

When the user provides a PDF, Word doc, or other content document:

### Text Content
- Extract all headings, body copy, statistics, quotes, and CTAs
- Preserve the document's section structure — it usually maps to page sections
- Note any specific phrasing that should be kept verbatim (taglines, endorsed claims, testimonial quotes)

### Image Content (DO NOT SKIP THIS)
- Identify every image in the document
- For each image, determine what it shows and which section it belongs to
- Match document images to actual hosted URLs from the client's website (from your Phase 1 catalog)
- If a document image doesn't match any cataloged URL, note it as "needs source" and ask the user

The reason this matters: marketing documents use images strategically. A PDF that shows character illustrations alongside brain science content is telling you those illustrations belong in the Brain Science section of the landing page. Ignoring them produces a text-heavy page that misses the point.

---

## Phase 3: Image Catalog & Verification

Before building anything, verify every image URL you plan to use.

### Verification Process

For each image URL:
1. Fetch the URL and check the HTTP status code
2. A 200 response with correct content-type means it's good
3. A 404 or other error means the URL is wrong

### Fixing Broken URLs

When an image URL returns 404:
1. Go back to the live site and search for the image by partial filename
2. Fetch the page source that originally contained the image
3. Look for the correct path — filenames often have slight variations (e.g., `GYM_Kid-at-Desk-300x295.png` vs `GYM_Kid_AtDesk-222x300.png`)
4. Never guess filenames. Always verify against the actual server.

### Final Catalog

Produce a verified list mapping each content section to its images:

```
Hero: logo.png (verified), character-teaching.png (verified), sticker-owl.png (verified)
Problem: classroom-photo.jpeg (verified)
Brain Science: character-1.png (verified), character-2.png (verified), character-3.png (verified), character-4.png (verified)
Shift: school-kits.jpg (verified), home-products.jpeg (verified)
Gallery: classroom.jpeg, confidence.png, founders.jpeg (all verified)
```

---

## Phase 4: Frontend Direction & Blueprint

Load the required frontend skill stack and complete the pre-code design gate from the top of this file. Decide whether generated web reference images are useful. Lock the conversion flow, section jobs, visual tokens, media plan, motion intent, responsive behaviour, and acceptance checks before implementation.

Self-reject the direction if it could be reused unchanged for an unrelated client. One coherent art direction is better than averaging several skill aesthetics.

## Phase 5: Build

### Choose the implementation stack

Do not force every landing page into one technology. Choose based on the actual project:

- **Existing repository or template:** preserve its framework, runtime, components, routing, and asset pipeline unless migration is explicitly requested.
- **Static campaign page:** semantic HTML, external CSS, and minimal JavaScript are preferred when no component architecture or app behaviour is needed.
- **Component-led or interactive page:** React/Vite, Next.js, or the repository's existing stack is valid when reusable components, integrations, state, or complex motion justify it.
- Verify dependencies before importing them. Prefer existing project libraries over adding fashionable packages.

The old default below applies only to a genuinely new static page.

### Static-page file structure

Keep it simple — a single directory with two files:

```
project-name/
├── index.html
└── styles.css
```

For a simple static page, avoid unnecessary build tools and framework dependencies. This is a default, not a prohibition against using the project's real stack.

### Design System Setup (styles.css)

Start the stylesheet with the extracted brand system:

```css
:root {
  /* Brand colors — extracted from client site */
  --primary: #0f4661;
  --primary-dark: #0a3347;
  --accent: #f37b78;
  --accent-hover: #e06560;
  /* ... all brand colors as custom properties */

  /* Neutral scale — always define these */
  --gray-50: #f8f9fa;
  --gray-100: #f0f2f5;
  --gray-200: #e2e5ea;
  --gray-300: #c8cdd5;
  --gray-400: #9ca3b0;
  --gray-500: #6b7280;
  --gray-600: #4b5563;
  --gray-700: #374151;
  --gray-800: #1f2937;
  --gray-900: #111827;

  /* Utility tokens */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.06);
  --shadow-md: 0 4px 20px rgba(0,0,0,0.08);
  --shadow-lg: 0 8px 40px rgba(0,0,0,0.12);
  --shadow-xl: 0 20px 60px rgba(0,0,0,0.16);
  --radius-sm: 0.375rem;
  --radius-md: 0.75rem;
  --radius-lg: 1rem;
  --radius-xl: 1.5rem;

  /* Transitions */
  --transition: 0.3s cubic-bezier(0.4, 0, 0.2, 1);
  --transition-bounce: 0.4s cubic-bezier(0.34, 1.56, 0.64, 1);
}

body {
  font-family: 'ExtractedFont', -apple-system, BlinkMacSystemFont, sans-serif;
  color: var(--gray-700);
  background: var(--white, #ffffff);
  line-height: 1.6;
  -webkit-font-smoothing: antialiased;
}
```

### Typography Rules (CRITICAL)

Bad typography is the #1 reason AI pages look amateur. Follow these strictly:

```css
/* Minimum sizes — NEVER go below these */
body { font-size: clamp(1rem, 0.95rem + 0.25vw, 1.125rem); } /* 16-18px */
h1 { font-size: clamp(2.25rem, 1.5rem + 3vw, 4rem); }       /* 36-64px */
h2 { font-size: clamp(1.75rem, 1.25rem + 2vw, 3rem); }       /* 28-48px */
h3 { font-size: clamp(1.25rem, 1rem + 1vw, 1.75rem); }       /* 20-28px */

/* Line heights */
h1, h2 { line-height: 1.15; letter-spacing: -0.02em; }
h3 { line-height: 1.3; }
p { line-height: 1.7; max-width: 65ch; } /* Readable line length */

/* Font weights — use contrast */
h1 { font-weight: 500; }  /* Start restrained; use heavier only when the brand calls for it */
h2 { font-weight: 500; }
h3 { font-weight: 500; }
body { font-weight: 400; } /* Regular for body */
```

Use BEM naming: `.section__element--modifier`. This keeps specificity flat and styles predictable.

### Animation & Motion

Add subtle scroll-reveal animations. These make pages feel polished without being distracting:

```css
/* Scroll reveal — add .reveal class to sections */
.reveal {
  opacity: 0;
  transform: translateY(30px);
  transition: opacity 0.8s cubic-bezier(0.16, 1, 0.3, 1),
              transform 0.8s cubic-bezier(0.16, 1, 0.3, 1);
}
.reveal.revealed {
  opacity: 1;
  transform: translateY(0);
}

/* Stagger children with delay classes */
.reveal--delay-1 { transition-delay: 0.1s; }
.reveal--delay-2 { transition-delay: 0.2s; }
.reveal--delay-3 { transition-delay: 0.3s; }

/* Button hover — subtle lift */
.btn { transition: transform var(--transition), box-shadow var(--transition); }
.btn:hover { transform: translateY(-2px); box-shadow: var(--shadow-lg); }

/* Image hover for galleries */
.gallery__item img { transition: transform 0.5s ease; }
.gallery__item:hover img { transform: scale(1.05); }
```

Add this JavaScript at the bottom for scroll reveals:
```javascript
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('revealed');
    }
  });
}, { threshold: 0.1 });
document.querySelectorAll('.reveal').forEach(el => observer.observe(el));
```

### Conversion structure, not a section template

Choose sections because each answers a real conversion question. Do not mechanically include all sections or repeat identical card rows. A common flow is:

1. **Header** — Sticky nav with logo, section links, and primary CTA button
2. **Hero** — Headline, subheadline, CTA buttons, and a visual element (character illustration, product image, or background)
3. **Trust Bar** — Stats, endorsements, partner logos (builds immediate credibility)
4. **Problem** — Pain points the audience faces. Use a 2-column grid: text on one side, supporting photo on the other. Images here make the problem feel real, not abstract.
5. **Approach/Solution** — How the product/service addresses the problem. Cards or feature blocks work well.
6. **Vision** — The desired future state. Emotionally driven, often with a background image.
7. **Evidence/Science** — Data, research, methodology. If character illustrations or graphics exist, place them here — they make dry content approachable.
8. **Testimonials** — Social proof from real users. Cards with quotes, names, and roles.
9. **Offer/Pricing** — What the user gets. If there are two tiers, use a "Two Paths" comparison layout. Highlight the recommended option.
10. **Gallery** — If images are available, a masonry or grid gallery showcases the product/service in action.
11. **Contact/CTA** — Final conversion section. Form, email, phone, or booking link.
12. **Footer** — Logo, links, copyright, social media.

### Image Integration Rules

This is where most landing page builds fail. Follow these rules:

- **Problem section**: Always pair problem text with a relevant photo (classroom, workplace, real-world context). A 2-column grid (60% text, 40% image) works well.
- **Evidence/Science section**: If the client has character illustrations, mascots, or infographics, place them here. Use a horizontal flex row showing 3-4 characters with labels.
- **Offer section**: Show product photos or kit photos alongside the offer description.
- **Gallery**: Use a CSS grid (3 columns desktop, 2 tablet, 1 mobile) with `object-fit: cover` and consistent heights. Add hover zoom effects.

The general principle: every section that makes a claim should have a supporting image. "Our program is used in real classrooms" needs a classroom photo. "Kids love our characters" needs character illustrations.

### Responsive Design

Build mobile-first with these breakpoints:

```css
/* Tablet */
@media (max-width: 1024px) {
  /* 2-column grids → stack or reduce
     Character images → smaller (100px)
     Gallery → 2 columns */
}

/* Mobile */
@media (max-width: 768px) {
  /* Everything single column
     Character images → 80px
     Gallery → single column
     Hamburger menu active
     Hero → stack vertically */
}
```

Key responsive patterns:
- Side-by-side grids stack vertically on mobile
- Image heights reduce at each breakpoint (desktop 300px → tablet 260px → mobile 220px)
- Font sizes use `clamp()` for fluid scaling
- Container padding increases slightly on mobile for comfortable reading

### HTML Template

The HTML should follow this structure:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Page Title] — [Brand Name]</title>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=[Font]:[weights]&display=swap" rel="stylesheet">
  <link rel="stylesheet" href="styles.css">
</head>
<body>
  <!-- Each section clearly commented -->
  <!-- ==================== HEADER ==================== -->
  <header class="header">...</header>

  <!-- ==================== HERO ==================== -->
  <section class="hero">...</section>

  <!-- ... remaining sections ... -->

  <!-- Minimal JS for hamburger menu and smooth scroll -->
  <script>
    // Hamburger toggle
    // Smooth scroll for anchor links
    // Sticky header shadow on scroll
  </script>
</body>
</html>
```

---

## Phase 6: Browser QA

After building, verify everything works before presenting to the user. This catches the silent failures (broken images, layout breaks) that undermine trust.

### Image Verification

Use the preview server to check every image:

1. Start a local server (`npx serve` or equivalent)
2. Take a screenshot to check overall appearance
3. For each image, check if it loaded:
   - In the preview, evaluate: `document.querySelectorAll('img').forEach(img => { if (img.naturalWidth === 0) console.log('BROKEN:', img.src) })`
   - Any image with `naturalWidth === 0` is broken — fix the URL
4. Force reload (`location.reload()`) before each screenshot to avoid stale cache

### Responsive and interaction check

Resize the viewport and verify at three widths:
- **Desktop** (1440px or the user's actual viewport): verify hierarchy, section rhythm, media crops, and maximum line lengths
- **Tablet** (768px): verify deliberate reflow rather than accidental wrapping
- **Mobile** (390px and 375px where practical): verify first fold, menu, CTA visibility, readable type, media crops, and touch targets
- Exercise menus, accordions, forms, anchors, carousels, and reduced-motion behaviour rather than checking screenshots alone
- Check horizontal overflow, console errors, broken media, focus visibility, and keyboard operation

### Fix Before Presenting

If any image is broken or any layout is wrong, fix it BEFORE showing the user. The user should see a polished result, not a work-in-progress. The typical fixes:
- Broken image → fetch correct URL from live site
- Layout overflow → add `overflow: hidden` or adjust grid
- Text overflow → reduce font size at breakpoint

---

## Phase 7: Deploy & Live Verify

### Vercel Deployment

When the user wants to deploy:

```bash
cd project-directory
npx vercel --yes --name [project-name] --prod
```

If the user wants a specific subdomain, use `--name` to set it (e.g., `--name growyourmind-redesign` creates `growyourmind-redesign.vercel.app`).

### Post-Deployment Verification

After deployment, verify the live URL:
1. Fetch the exact production/preview alias and confirm status, title, and expected page marker
2. Open the live URL in a real browser and capture desktop plus mobile screenshots
3. Check images/video, navigation, forms, console errors, overflow, and responsive behaviour on the deployed build
4. Confirm the alias points to this deployment; HTTP 200 alone is not proof that the correct project/version is live
5. Never claim deployment success from CLI output alone when browser verification is still blocked

---

## Common Pitfalls & How to Avoid Them

These are lessons from real projects. Each one caused a round of rework when missed.

| Pitfall | Prevention |
|---------|-----------|
| Treating PDFs as text-only | Extract images in the same pass as text. Map each image to its section. |
| Guessing image filenames | Never construct URLs from memory. Fetch the live page and extract actual `src` attributes. |
| Not verifying image URLs | Check every URL before using it. A 404 in production looks unprofessional. |
| Stale preview cache | Force `location.reload()` before every verification screenshot. |
| Missing responsive images | Test at 375px, 768px, and 1280px. Images that look fine at desktop can overflow on mobile. |
| Text-heavy sections | If a section makes a visual claim, it needs an image. "Real classroom results" without a classroom photo falls flat. |
| Assuming the font | Check the live site's actual font-family. Poppins and Inter look similar but aren't interchangeable. |

---

## Reference Files

For detailed section templates and CSS patterns, see:
- `references/section-patterns.md` — HTML/CSS templates for each section type
- `references/verification-checklist.md` — Step-by-step QA checklist

---

## SEO Essentials

Every landing page MUST include these. Missing them means the page is invisible to search engines:

```html
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>[Primary Keyword] — [Brand Name] | [Value Prop]</title>
  <meta name="description" content="[150-160 char description with primary keyword naturally included]">
  <link rel="canonical" href="https://[domain]/[page]">

  <!-- Open Graph for social sharing -->
  <meta property="og:title" content="[Title]">
  <meta property="og:description" content="[Description]">
  <meta property="og:image" content="[Hero image URL — 1200x630px ideal]">
  <meta property="og:url" content="[Canonical URL]">
  <meta property="og:type" content="website">

  <!-- Favicon -->
  <link rel="icon" href="[favicon URL]" type="image/png">
</head>
```

Additional SEO rules:
- Use ONE `<h1>` per page (the hero headline)
- Use `<h2>` for section titles, `<h3>` for subsection titles — proper hierarchy
- All images must have descriptive `alt` attributes (not "image1" or "photo")
- Use semantic HTML: `<header>`, `<nav>`, `<main>`, `<section>`, `<footer>`
- Add `loading="lazy"` to images below the fold
- Add `loading="eager"` to the hero image

---

## Performance Rules

These prevent the page from being slow:

- **Images**: Use Unsplash/source images with `w=` parameter for proper sizing. Never load a 4000px image for a 600px container.
- **Fonts**: Maximum 2 font families. Load only the weights you use (e.g., `wght@400;600;700` not `wght@100..900`).
- **CSS**: For a new static page, keep styles in one external file and avoid adding a CSS framework without a real benefit. In an existing project, preserve the established styling system.
- **JavaScript**: Use minimal JavaScript for a static page. React/Next/Vite are valid when the existing repository or required interactions justify them; do not add a framework merely to render static copy.
- **Above the fold**: The hero section must render without waiting for external resources. Use `font-display: swap` and preconnect to Google Fonts.

---

## Quick Start Checklist

When you receive a landing page request, run through this mentally:

- [ ] Do I have the client's website URL? (Need it for brand extraction)
- [ ] Do I have content documents? (PDF, doc, transcript)
- [ ] Did I read project/client instructions and identify the approved source of truth?
- [ ] Did I load `frontend-design` and current `design-taste-frontend` before coding?
- [ ] Did I decide whether `imagegen-frontend-web` is genuinely useful and generate/approve references if used?
- [ ] Did I avoid loading Design Taste v1 with the current version?
- [ ] Did I keep `imagegen-frontend-mobile` out of responsive website work?
- [ ] Is the visual thesis specific to this subject and expressed as one coherent system?
- [ ] Have I extracted ALL images from those documents?
- [ ] Have I built a verified image catalog with working URLs?
- [ ] Have I mapped every image to its content section?
- [ ] Have I tested all three breakpoints?
- [ ] Have I verified every image loads?
- [ ] Does the implementation preserve the existing stack, or use external CSS for a genuinely static page?
- [ ] Do I have proper meta tags (title, description, OG)?
- [ ] Are font sizes readable (16px+ body, proper heading scale)?
- [ ] Did I avoid all Anti-Slop patterns?
- [ ] Do animations work (scroll reveal, hover states)?
- [ ] Does the user want deployment? Where?
- [ ] If deployed, did I verify the exact live alias in a browser at desktop and mobile sizes?

---
> Source: [Aston1690/claude-skill-landing-page](https://github.com/Aston1690/claude-skill-landing-page) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
