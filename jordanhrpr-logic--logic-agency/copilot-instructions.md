## logic-agency

> Marketing website for Logic Agency Inc., a 2-person packaging and supply chain operations agency. The site has one job: convert brands searching for packaging help into monthly retainer clients.

# CLAUDE.md — Logic Agency Inc.

## What This Project Is

Marketing website for Logic Agency Inc., a 2-person packaging and supply chain operations agency. The site has one job: convert brands searching for packaging help into monthly retainer clients.

## Company Context

Logic Agency is an outsourced packaging and supply chain ops team on a monthly retainer. Founded by Jordan Harper, 20+ years in China/Asia supply chain operations. Based in Orange County, CA and Salt Lake City, UT.

**What they do:** Packaging sourcing, supply chain management, retail execution, inventory management, vendor coordination, global sourcing across 15+ countries.

**Who they serve:** Brands in beauty/wellness, CPG, and tech wearables doing $5-20M+ revenue, shipping 50K-500K+ units. Clients include Adidas, Vans, Target, Disney, Puma, Paramount+, Spotify, Epicutis, A24.

**How they charge:** Monthly retainers, not projects.
- Starter: $2,500-3,000/month — Advisory and sourcing
- Growth: $5,000-7,000/month — Packaging program management (most popular)
- Enterprise: $10,000+/month — Embedded operations

**Contact:**
- jordan@logicagencyinc.com
- 385.368.6837
- Domain: logicagencyinc.com

## Site Architecture

```
/                                    → Homepage (conversion page with tiers)
/guides/retail-ready-packaging       → Guide: Getting Your Packaging Retail-Ready
/guides/packaging-cost-reduction     → Guide: Packaging Cost Reduction Without Sacrificing Brand
/guides/packaging-system-that-scales → Guide: Building a Packaging System That Scales
/guides/packaging-sourcing           → Guide: How to Source Packaging Without Getting Burned
```

Future pages (not yet built):
- `/guides/us-market-entry` — US Market Entry for International Brands (build when Haldirams case study has results)

## Design System

### Colors
```css
--o: #FF600A;    /* Tiger Orange — primary brand color, CTAs, accents */
--od: #E55509;   /* Orange dark — hover states */
--bl: #0055FF;   /* Accent Blue — secondary accent, rarely used */
--bk: #212121;   /* Near black — body text */
--gr: #3E3E3E;   /* Gray — secondary text */
--dk: #2A2A2A;   /* Dark — dark section backgrounds */
--lt: #DFDFDF;   /* Light gray — borders, dividers */
--ow: #F3F3F3;   /* Off-white — light section backgrounds */
--w: #FFFFFF;    /* White — cards, content areas */
```

### Typography
- Font: **Poppins** (Google Fonts) — weights 400, 500, 600, 700, 800, 900
- Headlines: 800 weight, tight letter-spacing (-1.5px to -2.5px), tight line-height (1.05-1.15)
- Body: 400 weight, 16.5px, line-height 1.8, color `var(--gr)`
- Labels: 11-13px, 700 weight, uppercase, letter-spacing 1.5-2.5px

### Grid Texture
Light and dark sections use a subtle grid background:
```css
/* Light grid */
.gl { background-color: var(--ow); background-image: linear-gradient(rgba(0,0,0,.03) 1px, transparent 1px), linear-gradient(90deg, rgba(0,0,0,.03) 1px, transparent 1px); background-size: 28px 28px; }

/* Dark grid */
.gd { background-color: var(--dk); background-image: linear-gradient(rgba(255,255,255,.04) 1px, transparent 1px), linear-gradient(90deg, rgba(255,255,255,.04) 1px, transparent 1px); background-size: 28px 28px; }
```

**CRITICAL:** Every section must have an explicit `background-color`, not just `background-image`. Without it, mobile browsers show transparent sections and text becomes unreadable against inherited dark backgrounds.

### Logo
SVG — two overlapping circles in Tiger Orange:
```html
<svg viewBox="0 0 40 40" fill="none">
  <circle cx="15" cy="20" r="12" stroke="#FF600A" stroke-width="2.5" fill="none"/>
  <circle cx="25" cy="20" r="12" stroke="#FF600A" stroke-width="2.5" fill="none"/>
</svg>
```

### Buttons
- Primary: `.bo` — Tiger Orange background, white text, 50px border-radius
- Ghost light: `.bg` — transparent, dark text, light border
- Ghost dark: `.bw` — transparent, white text, white border at 40% opacity
- All buttons: 16px 36px padding, 700 weight, 15px font size, hover lifts with translateY(-2px)

### Section Pattern
Sections alternate between light (`.gl`) and dark (`.gd .dks`) backgrounds. Dark sections use white text with opacity for hierarchy (100% headlines, 85% body, 75% secondary, 60% tertiary, 50% labels).

### Navigation
Fixed dark nav bar, 72px height, backdrop blur. Logo left, links right. "Let's Talk" CTA button in orange. Nav links hide on mobile (max-width: 640px).

### Cards
White background, 16-20px border-radius, 1px border at 4% black opacity, hover lifts with translateY(-4px to -6px) and box-shadow.

## Page Templates

### Homepage
Full marketing page with sections: Hero → Situations (problem cards) → How It Works → Pricing Tiers → Metrics → Case Studies → Industries → FAQ (accordion) → CTA → Footer.

### Guide Pages
Article layout: Nav → Hero (breadcrumb, h1, lede paragraph, meta bar) → Article body (800px max-width, prose with embedded components) → CTA Band → Related Guides → Footer.

Guide-specific components (vary by page):
- Numbered section cards
- Comparison tables (dark header, alternating rows)
- Callout boxes (white, orange left border)
- Timeline layouts
- Checklist grids
- Inline case study cards (dark background)
- Red flag cards (red X icon)
- Landed cost calculator boxes
- Stage cards (colored left border progression)
- Sign/warning cards
- Math comparison boxes

## Content Strategy

### Voice
Direct, operational, expert. No marketing fluff. Writes like someone who's done the work, not someone selling a service. Uses specific numbers, real scenarios, and pattern-matched insights from 20 years of sourcing experience.

### SEO
- Each guide targets a specific search intent cluster
- JSON-LD Article schema on every guide page
- JSON-LD FAQ schema with 4-5 questions per guide (tuned for LLM discoverability)
- Meta descriptions target primary search queries
- Canonical URLs set on all pages
- Internal links: every guide links to homepage pricing section and to other guides via Related Guides section

### Search Intent Mapping
- Retail-Ready → "retail packaging requirements", "case pack requirements", "packaging for Target/Walmart"
- Cost Reduction → "reduce packaging costs", "packaging too expensive", "DIM weight optimization"
- System That Scales → "packaging for startup", "scaling packaging", "when to hire packaging operations"
- Packaging Sourcing → "where to source packaging", "packaging suppliers", "domestic vs overseas packaging"

## Case Studies (Reference Data)

### Epicutis (Premium Skincare — Enterprise Tier)
- Scaled from 3 to 21+ SKUs
- Full packaging program ownership
- Managed inventory system reducing order-to-cash time
- 15% cost savings
- 90-day inventory plan

### Audio Enhancement (B2B Hardware — Growth Tier)
- Packaging system functioning as daily-use product hub for classroom microphones
- Premium overseas production at $0 additional landed cost
- 20% shipping savings through right-sizing
- 4+ SKU expansion

### Gesine (Pre-Seed Startup — Growth Tier)
- Packaging system built for launch and engineered for commercial scale
- Retail-scalable from day one
- Full program management

### Artilect (Referenced in cost reduction content)
- 3-phase cost reduction roadmap
- 95% material reduction

### Haldirams (Enterprise — In Progress)
- US mainstream market entry
- Packaging adaptation for US retail
- Expo West launch planning
- Do NOT present as completed case study — still in progress

## Development Notes

### Tech Stack
- Next.js (App Router recommended)
- Tailwind CSS or keep existing vanilla CSS (both work)
- Deploy to Vercel
- Static generation — no server-side rendering needed
- No CMS — content lives in code

### Shared Components to Extract
- Nav (fixed dark bar with logo + links + CTA)
- Footer (dark, simplified on guide pages vs. 4-column on homepage)
- CTA Band (dark section with headline + subtext + two buttons)
- Related Guides (grid of linked cards)
- Callout Box (white card with orange left border)

### Mobile Breakpoints
- 1024px: Collapse multi-column grids to single column
- 640px: Hide nav links, reduce padding to 24px, stack flex layouts

### Performance
- No JavaScript frameworks beyond Next.js
- Minimal JS: intersection observer for scroll animations, accordion toggle for FAQ
- Google Fonts loaded via link tag (Poppins)
- No images currently — placeholder zones exist on homepage for future photos

### Accessibility
- All text meets WCAG AA contrast on both light and dark backgrounds
- Interactive elements have hover states
- Semantic HTML (sections, nav, footer, headings in order)

## Key Principles

1. **Pricing is the centerpiece.** The tiers should be visible and clear on the homepage. Every guide page funnels back to pricing.
2. **Search intent first.** Guide pages exist to capture organic search traffic. Write for the person typing the query, not for the brand.
3. **No blog.** Guides are evergreen reference content at `/guides/`, not dated blog posts.
4. **Show expertise through specificity.** Real numbers, real timelines, real scenarios. Never generic.
5. **Mobile-first testing.** Jordan checks everything on phone first. Every section must render correctly on mobile with explicit background colors and single-column layouts.

---
> Source: [jordanhrpr-logic/logic-agency](https://github.com/jordanhrpr-logic/logic-agency) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
