## shippage

> |


# Shippage

A $50K landing page team in your terminal. One sentence in, production-ready page out.

This skill encodes the expertise of a CRO strategist ($10K/mo), conversion copywriter
($3K-5K/page), SaaS designer ($5K-15K/page), and full-stack developer — as data.
No AI slop. No templates. No subscriptions.

**Data moat**: 1,000+ lines of conversion copy rules from A/B tests · 200 real design
token sets from top SaaS sites · 7-step conversion psychology sequence · Social proof
cascade system · Risk reversal on every CTA · Pre-launch intelligence for zero-proof pages

**Stack**: React + Tailwind CSS + Framer Motion + shadcn/ui + Magic UI (default).
Adaptable to any framework — see `references/framework-adapters.md`.

---

## The Conversion Sequence

Every page follows a 7-step narrative arc. Each section has exactly one job:

1. **STOP THE SCROLL** (Hero) — 5 seconds to keep the visitor
2. **BUILD TRUST** (Social Proof) — logos and metrics establish credibility
3. **AGITATE THE PAIN** (Problem) — visitor feels understood
4. **SHOW THE SOLUTION** (Features) — how you solve their problem
5. **PROVE IT WORKS** (Testimonials) — real customers validate the claim
6. **REMOVE DOUBT** (Pricing + FAQ) — objections answered
7. **MAKE THE ASK** (Final CTA) — visitor converts

Do not reorder. Do not skip. Do not combine jobs.

---

## Step 1: Intake

### Quick Mode
User provides product name + one-line description. Skill infers the rest using these rules:

**Awareness level** — infer from product description:
- Mentions a known problem category (slow deploys, manual work, data chaos) → `problem-aware`
- Mentions a solution category (CI/CD, CRM, analytics) → `solution-aware`
- Mentions specific product capabilities or pricing → `product-aware`
- Default if unclear: `solution-aware`

**CTA** — infer from product stage signals:
- "coming soon", "beta", "waitlist", "early access" → `waitlist`
- "open source", "free tier", "try it" → `free-trial`
- Mentions pricing, plans, or enterprise → `purchase`
- B2B + complex product → `demo`
- Default: `free-trial`

**Vibe** — infer from product domain:
- Developer tools, CLI, API, infrastructure → `dark-premium`
- AI, ML, data science → `dark-premium`
- Marketing, sales, CRM, email → `bold-modern`
- Design, creative, content → `playful-creative`
- Finance, security, compliance, healthcare → `enterprise-trust`
- Default: `minimal-clean`

**Social proof** — default to `none` in Quick Mode (assume pre-launch, use Step 4c)

**Audience** — extract from one-liner keywords:
- "teams", "developers", "engineers" → dev teams at startups
- "marketers", "founders", "businesses" → business users / non-technical
- Default: "teams building [product category]"

### Guided Mode (8 questions)
1. **Product name + one-liner**
2. **Target audience** (job title, company size, industry)
3. **Awareness level**: problem-aware / solution-aware / product-aware
4. **Primary CTA**: free trial / demo / waitlist / purchase
5. **Available social proof**: logos / testimonials / metrics / none yet
6. **Vibe**: minimal-clean / bold-modern / dark-premium / playful-creative / enterprise-trust / "look like X.com"
7. **Brand assets** (optional): brand colors (hex), font name, logo file — or "none, pick for me"
8. **Voice calibration**: customer's own words describing their pain

### Minimum Input (3 fields)
If the user provides very little: require product name, one-liner, and primary audience.
Defaults: solution-aware, free-trial CTA, minimal-clean vibe, professional-casual voice.

---

## Step 2: Design System

### Four-Tier System

| Tier | Source | When |
|------|--------|------|
| **0: Custom** | User provides brand colors, fonts, or full token set | User says "my brand is #1E40AF / Poppins" |
| 1: Curated | `scripts/select-design-system.py` queries `design-tokens-db.json` — 200 systems across 8 industries | Default. Match by industry + vibe. |
| 2: Database | Same script with `--top 8` for broader search | When Tier 1 has no close match. |
| 3: Live | `scripts/extract-tokens.sh --site [url]` | When user says "make it look like X.com" |

```bash
python3 scripts/select-design-system.py --industry "Developer Tools" --vibe dark
python3 scripts/select-design-system.py --list-industries
```

### Tier 0: Custom Brand Tokens

When the user provides their own brand colors, fonts, or design values, use them directly.
Build the `globals.css` `:root` block by mapping user values to the required CSS custom properties:

```css
:root {
  --background: [user bg, or derive: white for light, #0a0a0a for dark vibe];
  --foreground: [user text, or derive: contrast against background];
  --primary: [user brand color — the main accent];
  --primary-foreground: [derive: white or black, whichever contrasts 4.5:1];
  --muted: [derive: desaturated, lighter shade of background];
  --muted-foreground: [derive: mid-contrast text for secondary content];
  --accent: [user secondary color, or derive: lighter shade of primary];
  --accent-foreground: [derive: contrast text against accent];
  --border: [derive: subtle gray matching the palette];
  --card: [derive: slight offset from background];
  --card-foreground: [same as foreground];
  --destructive: [default: #ef4444 unless user specifies];
  --destructive-foreground: [derive: contrast text];
  --ring: [same as primary];
  --radius: [user value, or 0.5rem default];
}
.dark { /* derive dark mode variants: invert lightness, keep hue/saturation */ }
```

**Minimum user input needed**: one brand color (hex). Everything else can be derived.
**Font**: If user specifies a font name, use it for both heading and body (or heading only if they provide two).
Import via `@fontsource/[font]` (Vite/generic) or `next/font/google` (Next.js).

### Vibe Defaults (fallback when no match found and no custom tokens)

| Vibe | Font | Palette | Radius |
|------|------|---------|--------|
| minimal-clean | Inter | #000/#fff | 4px |
| bold-modern | Plus Jakarta Sans | saturated #6366f1 | 8px |
| dark-premium | Geist | #0a0a0a/#fafafa | 8px |
| playful-creative | DM Sans | warm #f97316 | 12px+ |
| enterprise-trust | Inter | navy/slate | 6px |

Tokens become CSS custom properties in `globals.css` → referenced via Tailwind semantic classes.

---

## Step 3: Section Selection

### Mandatory Sections
- `navbar` — `references/sections/navbar.md`
- Hero: `hero-centered` or `hero-split`
- `cta-footer` — `references/sections/cta-footer.md`

### Optional Sections

| Section | When to Include |
|---------|----------------|
| `social-proof-logos` | Always if logos available |
| `pain-points` | Problem-aware and solution-aware |
| `features-bento` or `features-alternating` | Always (core conversion) |
| `how-it-works` | Complex products, enterprise |
| `testimonials` | When testimonials with results exist |
| `pricing` | Self-serve with public pricing |
| `comparison-table` | Solution-aware comparing alternatives |
| `faq` | Always recommended |
| `waitlist` | When CTA is waitlist — replaces hero + cta-footer with viral waitlist system |
| `exit-intent-popup` | Always recommended — recovers 2-5% of abandoning visitors. Auto-includes for lead-gen CTAs (free trial, waitlist, demo, newsletter) |
| `sticky-cta-bar` | Always recommended — keeps CTA accessible after hero scrolls out of view. 8-15% conversion improvement |
| `contact-form` | When lead capture or direct inquiries are needed — most landing pages benefit from this |
| `cookie-consent` | Always recommended — GDPR/CCPA-compliant cookie consent banner with preference panel, script blocking, regional auto-detection |

### Programmatic Selection

```bash
python3 scripts/select-sections.py \
  --awareness problem-aware --cta free-trial \
  --proof logos,testimonials --sections 9

python3 scripts/select-sections.py \
  --awareness solution-aware --cta waitlist \
  --proof logos --hero centered --sections 8
```

### Hero Variant Heuristic
- **hero-split** (default): products with strong UI/screenshot
- **hero-centered**: waitlist (no screenshot), product-aware + purchase/demo, or explicit `--hero centered`

Target: **6–9 sections**. Fewer feels thin. More causes scroll fatigue.

---

## Step 4: Copy

Read `references/conversion-copy.md` — the authoritative source for every copy decision.

**Key rules** (detail in conversion-copy.md):
- Headline formulas by awareness level (problem/solution/product-aware)
- Subheadline: 15–25 words, must contain a specific number or timeframe
- Every feature gets a "so that [outcome]" benefit clause
- Zero banned AI words (leverage, unlock, revolutionize, seamlessly — full list in Section 5)
- "You/your" outnumbers "we/our" 3:1
- CTA uses approved text only + trust hint below every CTA
- Voice calibration: weave customer language throughout

After generating all copy, run a final anti-AI word sweep.

---

## Step 4b: Icons and Images

### Icons
All templates use **Lucide React** icons. When generating, select contextually relevant icons:
- Pain points: use negative-emotion icons (AlertTriangle, Clock, Frown, Ban)
- Features: use function icons (BarChart3, Zap, Shield, Users, Globe, Lock, Cpu)
- How-it-works: use process icons (Settings, Rocket, CheckCircle, ArrowRight)
- Pick icons that match the specific feature/pain described — never reuse the same icon twice

Templates with `icon: React.ElementType` props (how-it-works, pain-points, features-bento,
features-alternating) accept any Lucide icon. Choose from https://lucide.dev/icons.

If user provides custom SVG icons or brand icon files, render them as inline `<svg>` or
`<img>` components passed to the icon prop slot.

### Images and Screenshots

| Template | Needs | If user has none |
|----------|-------|------------------|
| Hero (split/centered) | Product screenshot | Use a gradient placeholder with "Your Product Here" or skip screenshot and use hero-centered with text-only layout |
| Features alternating | Screenshot per feature | Use abstract geometric illustrations via CSS (gradient boxes, code snippet mockups) |
| Social proof logos | Company logo files | Use text-only logo names in gray: `<span className="text-lg font-semibold text-muted-foreground">Acme Corp</span>` |
| Testimonials | Avatar photos | Use initials avatar: colored circle with first letter of name |

Always ask: "Do you have product screenshots, company logos, or testimonial photos ready?
If not, I'll generate placeholder visuals you can swap later."

---

## Step 4c: Pre-Launch Optimization

When user has no social proof (logos, testimonials, metrics), replace standard sections
with high-converting pre-launch alternatives. Most indie hackers and solo founders hit
this case — handle it as a first-class workflow, not a fallback.

### Replace Social Proof Logos with Traction Signals
Instead of a logo strip, use one of:
- **Waitlist counter**: "Join 847 founders on the waitlist" (use a realistic number)
- **GitHub stars**: "★ 2.4k stars on GitHub" (if open source)
- **Beta badge**: "Currently in private beta — 50 spots left"
- **Founder credibility**: "Built by a [Company] engineer" or "From the team behind [Previous Product]"

### Replace Testimonials with Founder Story
Instead of customer quotes, use:
- **Builder credibility section**: "I built [Product] because..." — 3-sentence founder story
  with photo, linking the pain (Section 3) to personal experience
- **Early traction proof**: beta user count, GitHub activity, Product Hunt launch results
- **"Why now" section**: market timing argument (regulation change, technology shift, cost reduction)

### Replace Product Screenshots with
- **Problem visualization**: Show the painful "before" state (messy spreadsheet, error messages,
  slow dashboard mockup) to make the pain tangible
- **Architecture diagram**: For dev tools — show the system diagram of how the product works
- **Waitlist CTA with value preview**: "See how [Product] works — join the waitlist for early access"

### Pre-Launch Section Order
Replace the standard 7-step sequence with:
1. **Waitlist hero** (`references/sections/waitlist.md`) — viral waitlist with email capture,
   referral link, queue position, tiered rewards, leaderboard, and social sharing.
   Replaces $15-69/mo tools (GetWaitlist, Viral Loops, Prefinery). Includes serverless
   API route template (`references/waitlist-api.md`) with 4 ranking algorithms, fraud
   prevention, welcome emails, and analytics tracking.
2. Traction signals (waitlist count or founder credibility)
3. Pain points (agitate harder — this carries more weight without proof)
4. Solution preview (features with CSS mockups instead of screenshots)
5. Founder story (replaces testimonials)
6. FAQ (address "when does this launch?", "how do I get access?", "is this free?")
7. Final CTA (waitlist or early access)

---

## Step 5: Effects

Read `references/effects-catalog.md` for the complete library with install instructions.

**Budget**: max 3 premium effects per page:
1. ONE hero background (Aurora, Particles, DotPattern, etc.)
2. ONE headline text effect (optional)
3. ONE CTA/section effect (optional)

Universal motion (scroll reveals, hover states, stagger) is always included — free.

If an effect fails to install: fall back to CSS-only alternative. The page must look good with zero JS effects.

---

## Step 6: Assembly

1. Read each section template from `references/sections/`
2. Read shared rules from `references/section-defaults.md` — **especially the Visual Polish section**
3. Apply design tokens in `globals.css` (`:root` + `.dark`)
4. Insert generated copy into section templates
5. Wire effects: hero background (REQUIRED), text effect (recommended), CTA effect (recommended)
6. **Apply visual polish (REQUIRED):**
   - Grain texture overlay on `body` (see section-defaults.md §Visual Polish)
   - Glow effects behind hero content and CTA buttons
   - Gradient transitions between sections — never hard edges
   - Frosted glass on cards for dark vibes
   - Colored shadows using brand primary
   - Scroll-linked parallax on hero section
   - Animated number counters on social proof stats
   - Mix 3-4 different entrance animation variants across sections (see §Scroll Animation map)
7. Add conversion tools: exit-intent popup (always), sticky CTA bar (always). Add `data-hero-cta` to hero CTA button and `data-cta-footer` to final CTA section.
8. Add cookie consent banner (`references/sections/cookie-consent.md`). Add `data-cookie-settings` to footer "Cookie Settings" link.
9. Generate legal pages (`references/legal-pages.md`): Privacy Policy, Terms of Service, Cookie Policy, Acceptable Use Policy. Fill templates with user's business details from intake.
10. Add SEO assets: JSON-LD structured data (Organization + SoftwareApplication + FAQPage schemas), OG image (component or Next.js dynamic), sitemap.xml, robots.txt. See `references/framework-adapters.md` for templates per framework.
11. See `references/framework-adapters.md` for framework-specific setup

### Dark Mode
- **Dark-first only**: dev tools, AI/ML, dark-premium vibe
- **Light + toggle**: enterprise, fintech, general SaaS
- **Light-only**: conservative enterprise, healthcare

---

## Step 7: QA

Run automated checks: `python3 scripts/qa-check.py --dir [project]`

### Critical Manual Checks
1. CTA visible without scrolling on mobile, all touch targets ≥ 44px
2. Zero banned AI words, every feature has "so that" benefit clause
3. Hero content is static HTML (no JS-dependent first paint), LCP < 2.5s
4. Color contrast ≥ 4.5:1, semantic heading hierarchy, skip-to-content link
5. Title ≤ 60 chars, meta description ≤ 160 chars, OG tags present, JSON-LD schemas valid

Full checklists: Mobile (10 items), Speed (9), Conversion (12), Accessibility (9), SEO (9), Dark Mode (5).

---

## Audit Mode

When user asks to "audit", "roast", or "review" a landing page URL:

1. **Extract** tokens via `extract-tokens.sh --site [url]`, read page content
2. **Score** against all QA checklists (Mobile, Speed, Conversion, A11y, SEO)
3. **Recommend** specific fixes ranked by conversion impact
4. **Rewrite** headline, subheadline, primary CTA, weakest feature, first testimonial
5. **Compare** design tokens against industry best-in-class

Format each fix as: BEFORE → AFTER → WHY.

---

## File Reference

| Category | Files |
|----------|-------|
| **Copy rules** | `references/conversion-copy.md` |
| **Effects** | `references/effects-catalog.md` |
| **Shared defaults** | `references/section-defaults.md` |
| **Framework setup** | `references/framework-adapters.md` |
| **Design tokens** | `references/design-tokens-db.json` (200 sites, 8 industries) |
| **Section templates** | `references/sections/` — navbar, hero-centered, hero-split, social-proof-logos, pain-points, features-bento, features-alternating, how-it-works, testimonials, comparison-table, pricing, faq, cta-footer, contact-form, waitlist, exit-intent-popup, sticky-cta-bar, cookie-consent |
| **Legal pages** | `references/legal-pages.md` (Privacy Policy, Terms of Service, Cookie Policy, Acceptable Use Policy — replaces Termly/Iubenda/GetTerms at $3-35/mo) |
| **Waitlist system** | `references/sections/waitlist.md` (frontend) + `references/waitlist-api.md` (serverless API, email templates, storage adapters) |
| **Conversion tools** | `references/sections/exit-intent-popup.md` (exit-intent detection, A/B testing, frequency capping, mobile teaser) + `references/sections/sticky-cta-bar.md` (scroll-aware bar, 4 variants, countdown timer) |
| **Scripts** | `scripts/select-design-system.py`, `scripts/select-sections.py`, `scripts/extract-tokens.sh`, `scripts/parse-tokens.py`, `scripts/qa-check.py` |

---

## Execution Summary

1. **Intake** — product info (quick, guided, or minimal mode). Ask about brand assets.
2. **Design System** — custom tokens (Tier 0) > curated (Tier 1/2) > live extract (Tier 3) > vibe defaults
3. **Sections** — pick 6–9 using decision matrix or `select-sections.py`
4. **Copy** — write per conversion-copy.md rules, sweep for banned words
4b. **Icons & Images** — select contextual Lucide icons, handle missing screenshots with fallbacks
5. **Effects** — pick ≤ 3 premium effects by vibe, with CSS fallbacks
6. **Assembly** — generate project with dark mode support, cookie consent banner, conversion tools
7. **Legal Pages** — generate Privacy Policy, Terms of Service, Cookie Policy from `references/legal-pages.md`
8. **SEO Assets** — JSON-LD schemas, OG image, sitemap.xml, robots.txt from `references/framework-adapters.md`
9. **QA** — run `qa-check.py` + manual critical checks, fix all failures

Output: `npm install && npm run dev` — a single deployable project.

---
> Source: [imjahanzaib/shippage](https://github.com/imjahanzaib/shippage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
