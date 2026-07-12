## strategyverse-website

> ﻿# Strategyverse Website — Project Reference

﻿# Strategyverse Website — Project Reference

## Standing Rules

- **All new content and changes must be AEO compliant.** This applies to every new blog post, page edit, schema update, and structural change. See the AEO Compliance Checklist section below for what this means in practice.

---

## Overview

Strategyverse Consulting is a **Strategic Public Relations** company based in Noida, India. This is a static HTML/CSS/JS website (no build tools, no frameworks) hosted on **GitHub Pages** at:

**https://strategyverse.in** (custom domain, DNS pointed from GitHub Pages)

Repository: `https://github.com/StrategyVerse/StrategyVerse-website`

---

## Brand Identity

### Colours

| Token            | Hex       | Usage                                    |
| ---------------- | --------- | ---------------------------------------- |
| `--blue`         | `#377FCC` | Primary brand colour, buttons, links     |
| `--blue-dark`    | `#2a6ab3` | Hover / active states                    |
| `--blue-deeper`  | `#1a3a5c` | Hero gradient, deep accents              |
| `--amber`        | `#F5A623` | Secondary accent, section labels, checks |
| `--amber-dark`   | `#d98e1a` | Amber hover states                       |
| `--dark`         | `#1a1a2e` | Heading text, dark backgrounds           |
| `--dark-light`   | `#16213e` | Hero gradient end                        |
| `--gray-100`     | `#f7f8fa` | Light section backgrounds                |
| `--gray-200`     | `#e9ecef` | Borders, dividers                        |
| `--gray-400`     | `#adb5bd` | Muted text                               |
| `--gray-600`     | `#6c757d` | Body text, subtitles                     |
| `--gray-800`     | `#343a40` | Default body text colour                 |
| `--white`        | `#ffffff` | Backgrounds, text on dark                |

### Fonts

- **Headings:** `Playfair Display` (weights 600, 700, 800) — serif, elegant
- **Body:** `Inter` (weights 400, 500, 600, 700) — clean sans-serif
- Loaded from Google Fonts on every page

### Typography Scale

```
h1  → clamp(2.2rem, 5vw, 3.5rem)
h2  → clamp(1.8rem, 4vw, 2.8rem)
h3  → clamp(1.2rem, 2.5vw, 1.5rem)
body → 16px, line-height 1.7
section-label → 0.85rem, uppercase, 3px letter-spacing, amber colour
```

### Shadows

```
--shadow-sm: 0 2px 8px rgba(0,0,0,0.08)
--shadow-md: 0 4px 20px rgba(0,0,0,0.12)
--shadow-lg: 0 8px 40px rgba(0,0,0,0.16)
```

### Buttons

- `.btn-primary` — Blue (#377FCC), white text, pill shape (border-radius 50px)
- `.btn-outline` — Transparent, white border, used on dark backgrounds
- `.btn-amber` — Amber (#F5A623), dark text
- `.btn-arrow` — Appends animated `→` that translates right on hover

---

## Blog Cover Illustration Style (Signature House Style)

All blog cover images use a **signature illustration style** unique to Strategyverse — not stock photos. Every cover must follow this system so the Insights grid reads as one unmistakable visual language.

### The fixed signature (constant on every cover)

- **Medium:** highly detailed scratchboard / wood-engraving illustration in the vintage-etching tradition (Gustave Doré feel) — dense cross-hatching, flowing parallel linework, extreme contrast.
- **Palette:** monochrome duotone — deep midnight **navy ink (`#1a1a2e`)** on a **warm off-white paper** background (with a thin paper border), never pure black.
- **The one-amber rule:** exactly **one element glows warm amber (`#F5A623`)** and it is the ONLY colour in the entire frame — the focal "signal/insight." Everything else is navy linework on off-white.
- **Composition:** a lone figure in a vast, dramatic landscape; epic scale; contemplative, aspirational mood; clean negative space. No text or lettering baked into the image (the page renders the H1 separately).

### What varies: topic-matched metaphor per post

Each article gets its own conceptual scene chosen to fit the topic, drawn in the fixed style above. Examples used: navigation (boat through a maelstrom toward an amber dawn = strategy/turbulence), voice (figure broadcasting through a horn = communications), rising (staircase of books to an amber sun = thought leadership), beacon on a cliff (signal through noise).

### Production

- **Tool/model:** image-generation MCP, model **`flux-2-max`**, aspect ratio **16:9**, PNG/JPG. ~14 credits per image — generate deliberately, don't regenerate idly.
- **Prompt template:** `Highly detailed scratchboard and wood-engraving illustration in the style of vintage etching and Gustave Doré — dense intricate cross-hatching and fine flowing parallel linework, extreme high contrast, dramatic and cinematic. Monochrome duotone in deep midnight navy ink (color #1a1a2e) on a warm off-white paper background with a thin paper border, NOT pure black. The scene: [TOPIC METAPHOR]. One element — [the amber element] — glows warm amber (color #F5A623) and it is the ONLY color in the entire image; everything else is navy engraved linework on off-white. Epic sense of scale, contemplative and aspirational mood, editorial illustration. No text, no lettering.`
- **Storage:** save each cover to `images/blog/<slug>.jpg` and reference it locally (replaces the old Pexels URLs).
- **Per-post references to update:** blog hero `<img>` src + alt, `og:image`, `twitter:image`, Article schema `image`, the Insights card thumbnail in `insights/index.html`, and (for drafts) the `IMAGE:`/`IMAGE_ALT:` metadata comment.
- **og:image note:** these raster covers double as the social-share image, so no SVG/preview problem.

---

## Folder Structure

```
StrategyVerse Website/
├── css/
│   └── styles.css              # Single stylesheet for the entire site (~27 KB)
├── js/
│   └── main.js                 # All interactivity (~5.5 KB)
├── images/
│   ├── logo-white.png          # White logo (used in navbar & footer on dark bg)
│   ├── logo-blue.png           # Blue logo (used as favicon)
│   ├── founder-praveen-singh.jpg
│   ├── clients/                # 14 client logos (PNG/JPG/SVG/WEBP)
│   │   ├── Patel-Engineering-Logo.png
│   │   ├── Mettler_Toledo-Logo.wine_-e1754026397339.png
│   │   ├── lohum_cleantech_pvt_ltd_logo-e1754026357152.jpg
│   │   ├── 6389e993c5a15cd75816ef7d_Hubler-home-logo-Rect.png
│   │   ├── Manu-Bhoomi-Logo-e1747199312137.png
│   │   ├── Logo-main-e1747199299597.png
│   │   ├── Seed-To-Soul-Logo-01-e1754025868470.png
│   │   ├── credentiai_logo-e1754026718623.jpg
│   │   ├── MTaI.png
│   │   ├── SunWheel-Software-Solutions.png
│   │   ├── Pakka Limited.svg
│   │   ├── Power Ministry.png
│   │   ├── New-Age-Markets-in-Electricity.webp
│   │   └── YEStack.svg
│   ├── testimonials/           # 5 testimonial portraits
│   │   ├── Then Powe Minister Mr. RK Singh.jpg
│   │   ├── Mr. Manu Garg, Founder, Garg Technologies & Concepts.jpg
│   │   ├── Dr. Monika Mathur, Content Specialist, Mettler-Toledo.jpg
│   │   ├── Mr. Prakash Rawat, Co-Founder, ManuBhoomi.jpg
│   │   └── Mr. Rishi Jha, Chief Growth Officer, Hubler.jpeg
│   └── coverage/               # 13 media coverage clips
│       ├── Business-Standard-March-14th-2016.jpg
│       ├── Myforexeye-ET_Nov-18-2022.jpg
│       ├── Hubbler-in-BW-Disrupt.png
│       ├── Hubbler-in-The-Financial-Express.png
│       ├── Hubbler-in-Times-of-India.png
│       ├── Lohum-in-Business-World-Strategic-Article.png
│       ├── Lohum-in-Financial-Express-Strategic-Article.png
│       ├── Lohum-in-Hindustan-Times-Strategic-Article-e1753707186827.png
│       ├── MN-Dastur-in-The-Economic-Times.png
│       ├── Medical-Technology-Association-of-India-in-The-Economic-Times.png
│       ├── Patel-Engineering-in-Assam-Tribune-Strategic-Article.jpeg
│       ├── Shri-RK-Singh-on-CNN-News18.png
│       └── Shri-RK-Singh-on-NDTV.png
│
├── index.html                  # Home page (root)
├── about/
│   └── index.html              # About Us page → /about/
├── services/
│   └── index.html              # Services page → /services/
├── insights/
│   └── index.html              # Blog listing page → /insights/
├── contact/
│   └── index.html              # Contact page → /contact/
├── privacy-policy/
│   └── index.html              # Privacy Policy → /privacy-policy/
├── terms-of-service/
│   └── index.html              # Terms of Service → /terms-of-service/
├── blog/
│   ├── pitch-slapping/index.html           # By Praveen Singh
│   ├── ideation-in-pr/index.html           # By Praveen Singh
│   ├── select-pr-agency/index.html         # By Praveen Singh
│   ├── service-pr-clients/index.html       # By Praveen Singh
│   ├── questioning-in-pr/index.html        # By Praveen Singh
│   ├── whats-wrong-pr/index.html           # By Praveen Singh
│   ├── truth-press-release/index.html      # By Praveen Singh
│   ├── ai-disrupting-pr/index.html         # By Strategyverse Content Team
│   ├── social-media-crisis/index.html      # By Strategyverse Content Team
│   ├── personal-branding-cxo/index.html    # By Strategyverse Content Team
│   ├── startup-pr-mistakes/index.html      # By Strategyverse Content Team
│   └── earned-vs-paid-media/index.html     # By Strategyverse Content Team
│
├── drafts/
│   ├── .gitkeep
│   └── TEMPLATE.html           # Blog post template for scheduled publishing
├── .github/
│   └── workflows/
│       └── publish-drafts.yml  # Daily GitHub Action to auto-publish scheduled posts
│
├── 404.html                    # Custom 404 page (absolute paths, served from any URL)
├── sitemap.xml                 # XML sitemap for search engines (19 URLs)
├── robots.txt                  # Crawler rules + sitemap reference
├── .gitignore
└── CLAUDE.md                   # This file
```

### Untracked folders (source material, not deployed)

- `Clients/` — Original client logos and coverage clips
- `Insights/` — Original blog article drafts
- `Testimonials/` — Original testimonial text and photos
- `Feedback on the draft website.docx` — Founder's feedback document
- `About-me.md.txt` — Founder bio source
- `New logo white.png`, `New logo_Blue.png` — Logo source files
- `Website I like.jpg`, `Website I like 2.jpg` — Design inspiration screenshots

---

## Pages

### Main Pages

| Page | File | Description |
| ---- | ---- | ----------- |
| Home | `index.html` | Hero with SVG graphic, services grid (8 services), clientele logos, "Why Strategyverse" section, testimonials carousel (5 people), CTA |
| About Us | `about/index.html` | Company story, mission/vision, values, founder section (Praveen Singh with photo and LinkedIn) |
| Services | `services/index.html` | 8 detailed service cards, clientele logos, rolling "Clients in Media" marquee, process section (Discover → Strategise → Execute → Measure) |
| Insights | `insights/index.html` | Grid of 12 blog article cards with Pexels images, category tags, read-time estimates |
| Contact | `contact/index.html` | Contact form (FormSubmit.co), email/location/hours info, "Book a Call" CTA |
| Privacy Policy | `privacy-policy/index.html` | 9-clause privacy policy |
| Terms of Service | `terms-of-service/index.html` | 12-clause terms of service |
| 404 | `404.html` | Custom error page — large 404 heading, redesign explanation, nav links to all main pages. Uses absolute paths (`/css/`, `/images/`) since GitHub Pages serves it from any URL |

### Blog Articles (12 total)

7 articles by **Praveen Singh** (founder's original writing) and 5 by **Strategyverse Content Team** (generated to fill out the insights section on trending PR topics).

Each blog page uses a consistent template: page-hero with category tag, article body with structured headings, a "related articles" or CTA section at the bottom, and the shared navbar/footer.

---

## Key Design Decisions

### Clean URLs

All pages use a folder/index.html pattern for clean URLs (no `.html` in the address bar):
- `/about/` instead of `about.html`
- `/blog/pitch-slapping/` instead of `blog-pitch-slapping.html`
- Asset paths use relative `../` (depth-1) or `../../` (depth-2) prefixes
- Navigation links are relative to each page's depth in the folder structure

### Layout

- **Container:** max-width 1200px, 24px side padding
- **Responsive breakpoints:** 992px (tablet), 640px (mobile)
- **Navbar:** Fixed position, background darkens on scroll (`.scrolled` class), logo uses overflow-hidden cropping (270px image in 60px container with negative margin-top to centre-crop)
- **Hero section:** Full-viewport gradient background with floating SVG illustration on the right side; SVG contains shield (reputation), microphone (voice/PR), LinkedIn & X icons, target (audience), hashtag (social), speech bubble (narrative), with animated elements
- **Sections alternate** between white and light gray (`--gray-100`) backgrounds
- **Scroll-reveal:** Elements with `.reveal` class fade/slide in via Intersection Observer

### Hero Graphic (SVG)

The hero contains an inline SVG (`viewBox="0 0 500 500"`) with these elements balanced left and right of a central shield:

- **Centre:** Shield with checkmark (reputation protection)
- **Top left:** Target circles (audience targeting)
- **Mid left:** Hashtag icon (social media)
- **Bottom left:** Speech bubble with text lines (narrative)
- **Top right:** LinkedIn icon
- **Mid right:** X / Twitter icon
- **Bottom right:** Microphone with sound waves (voice/PR)
- Connecting dotted lines between elements
- Floating accent dots with CSS animations
- Entire graphic floats with a 12s ease-in-out animation

### Contact Form

Uses **FormSubmit.co** (serverless form backend):
- Form action uses a **hashed email** (`37d6b91fe989fa1dc01c98311c49d16d`) instead of plain text to prevent email scraping
- **AJAX submission** via `fetch()` to the `/ajax/` endpoint — no page refresh
- On success: form is replaced in-place with a green checkmark and "Message Sent Successfully!" message
- On error: button resets and an alert is shown
- Hidden fields: subject line, captcha disabled, honeypot spam protection
- Fields: first name, last name, email, phone, company, service dropdown, message

### Testimonials Carousel

- Auto-advances every 5.5 seconds
- Pauses on hover, resumes on mouse leave
- Touch/swipe support (50px threshold)
- Responsive: 3 visible (desktop), 2 (tablet), 1 (mobile)
- Dot pagination

### Media Coverage Marquee

- CSS `@keyframes` infinite scroll animation
- JavaScript duplicates track items for seamless loop
- Located on the services page

---

## Services Offered (8)

1. Communications Strategy
2. Reputation Management
3. Crisis Communications
4. Branding
5. Social Media Management
6. Influencer Management
7. Media Relations
8. Content Management

Each uses an icon (Unicode symbols or inline SVG) in the brand blue colour. The Content Management icon specifically uses an inline SVG pen to avoid OS emoji colour rendering issues.

---

## Contact Information

- **Email:** info@strategyverse.in
- **Address:** First Floor, H-54, H Block, Sector 63, Noida - 201309
- **LinkedIn:** https://www.linkedin.com/company/strategyverse/?viewAsMember=true
- **Founder LinkedIn:** https://www.linkedin.com/in/prwin/
- **Hours:** Monday - Friday, 9:00 AM - 6:00 PM IST

Note: No Twitter or Instagram links are used anywhere on the site (by explicit instruction from the founder).

---

## JavaScript Features (js/main.js)

1. **Navbar scroll effect** — adds `.scrolled` class after 50px scroll
2. **Mobile hamburger toggle** — opens/closes `.nav-links`, prevents body scroll
3. **Scroll-reveal** — Intersection Observer on `.reveal` elements (threshold 0.15)
4. **Active nav highlighting** — matches current URL to nav links
5. **Contact form AJAX** — `fetch()` to FormSubmit `/ajax/` endpoint, in-place success message (no page refresh)
6. **Testimonials carousel** — auto-slide, pause-on-hover, swipe, dots, responsive
7. **Coverage marquee** — duplicates track items for seamless infinite scroll

---

## Shared Components (on every page)

### Navbar
```html
<nav class="navbar">
  Logo (logo-white.png) | Home | About Us | Services | Insights | Contact Us (CTA button)
  Mobile: hamburger toggle
</nav>
```

### Footer
```html
<footer class="footer">
  Brand column (logo with overflow-crop at 54px, tagline, LinkedIn)
  Quick Links column
  Services column
  Contact column
  Bottom bar: copyright 2026 | Privacy Policy | Terms of Service
</footer>
```

Footer logo uses the same overflow-crop technique as the navbar but at ~90% of header size (54px visible height, image scaled to 216px with negative margin-top).

---

## SEO

### Sitemap & Robots

- `sitemap.xml` — lists all 19 pages with `<lastmod>`, `<changefreq>`, and `<priority>` values (homepage 1.0, main pages 0.8, blog posts 0.6, legal pages 0.3)
- `robots.txt` — `User-agent: * Allow: /` with `Sitemap: https://strategyverse.in/sitemap.xml`

### Meta Tags (on every page)

- `<meta charset="UTF-8">` and `<meta name="viewport">`
- Unique `<title>` (under 60 chars, includes brand name)
- Unique `<meta name="description">` (under 155 chars)
- `<meta name="keywords">` — relevant terms per page
- `<link rel="canonical">` — full canonical URL per page

### Open Graph & Twitter Cards (on every page)

- `og:title`, `og:description`, `og:url`, `og:type`, `og:image`, `og:site_name`
- `twitter:card` (summary_large_image), `twitter:title`, `twitter:description`, `twitter:image`

### JSON-LD Structured Data

| Page | Schema Type | Details |
| ---- | ----------- | ------- |
| Homepage | `Organization` | Business name, URL, logo, address, email, LinkedIn |
| Services | `ProfessionalService` + `OfferCatalog` | All 8 services listed with descriptions |
| Contact | `LocalBusiness` | Address, email, opening hours, LinkedIn |
| Blog posts (12) | `Article` | Headline, author, publisher, dates, image |

### Heading & Image Audit

- Every page has exactly **one `<h1>`** tag with proper `<h2>` / `<h3>` hierarchy
- Every `<img>` tag has a descriptive `alt` attribute (zero missing or empty)

---

## Scheduled Blog Publishing System

### How It Works

A GitHub Actions workflow (`publish-drafts.yml`) runs daily at **5:30 AM IST** (midnight UTC). It checks the `drafts/` folder for blog posts whose date has arrived and automatically publishes them.

### How to Schedule a Blog Post

1. **Copy the template:** Duplicate `drafts/TEMPLATE.html`
2. **Name the file:** `YYYY-MM-DD-your-post-slug.html`
   - Example: `2026-04-15-why-pr-matters.html`
   - The date is the publish date; the slug becomes the URL (`/blog/why-pr-matters/`)
3. **Fill in the metadata** in the HTML comment at the top:
   ```
   TITLE: Why PR Matters in 2026
   DESCRIPTION: A compelling summary under 155 characters
   AUTHOR: Praveen Singh
   CATEGORY: Strategy
   READ_TIME: 6 min read
   IMAGE: https://images.pexels.com/photos/.../photo.jpeg?auto=compress&cs=tinysrgb&w=600
   IMAGE_ALT: Descriptive alt text
   KEYWORDS: PR, strategy, 2026
   ```
4. **Write the article** content inside the `<article>` section
5. **Commit and push** the file to the `drafts/` folder

### What the Workflow Does Automatically

- Moves the draft to `blog/<slug>/index.html`
- Adds a card to the top of the Insights page grid
- Adds the new URL to `sitemap.xml`
- Commits and pushes the changes

### Manual Trigger

You can also publish immediately by going to **GitHub → Actions → Publish Scheduled Blog Posts → Run workflow**.

### Files

- `drafts/TEMPLATE.html` — Blog post template with all metadata fields
- `.github/workflows/publish-drafts.yml` — The GitHub Actions workflow

---

## AEO Compliance Checklist

Every new blog post, page, or significant edit **must** satisfy all of the following before publishing.

### Schema (JSON-LD)

- [ ] **Article schema** on every blog post — `headline`, `author` (with `sameAs` LinkedIn URL), `url`, `image` (Pexels hero URL), `datePublished`, `dateModified`, `mainEntityOfPage`
- [ ] **FAQPage schema** on every blog post — minimum 3 Q&A pairs drawn from the article content
- [ ] **BreadcrumbList schema** on every blog post — Home → Insights → Article title
- [ ] **HowTo schema** on any article structured as numbered steps
- [ ] **Speakable schema** on every blog post — `cssSelector: [".page-hero h1", ".blog-content p:first-child"]`
- [ ] **og:image / twitter:image** must use the article's Pexels hero image, not the site logo

### Author Attribution

- Articles by **Praveen Singh**: `@type: Person`, `name`, `url: https://strategyverse.in/about/`, `sameAs: https://www.linkedin.com/in/prwin/`, `jobTitle: Founder & Chief Strategist`
- Articles by **Strategyverse Content Team**: `@type: Organization`, `name: Strategyverse Content Team`, `url: https://strategyverse.in`

### On-Page Content

- [ ] One `<h1>` per page, matching the Article schema `headline`
- [ ] Every `<img>` has a descriptive `alt` attribute
- [ ] Unique `<title>` (under 60 chars) and `<meta name="description">` (under 155 chars)
- [ ] `<link rel="canonical">` with the full URL

### Internal Linking

- [ ] Every blog post links to at least 2 other relevant blog posts using contextual anchor text

### Sitemap

- [ ] New blog URL added to `sitemap.xml` with appropriate `<lastmod>`, `<changefreq>` (monthly), and `<priority>` (0.6)

---

## Git Configuration

- **User:** StrategyVerse
- **Email:** info@strategyverse.in
- **Remote:** https://github.com/StrategyVerse/StrategyVerse-website.git
- **Branch:** main
- **Hosting:** GitHub Pages (source: main branch, root `/`)

---
> Source: [StrategyVerse/StrategyVerse-website](https://github.com/StrategyVerse/StrategyVerse-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-12 -->
