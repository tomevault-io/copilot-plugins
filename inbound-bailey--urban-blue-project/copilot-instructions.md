## urban-blue-project

> Urban Blue Project is a development management firm focused on affordable missing-middle housing. They integrate approvals, servicing strategy, and delivery method to make small multi-family and low-density mixed-use projects viable — often pairing off-site construction (manufactured/modular) with decentralized water and wastewater (including septic) when it reduces risk and improves affordability.

# Urban Blue Project — Website Build Brief

## Project Overview

Urban Blue Project is a development management firm focused on affordable missing-middle housing. They integrate approvals, servicing strategy, and delivery method to make small multi-family and low-density mixed-use projects viable — often pairing off-site construction (manufactured/modular) with decentralized water and wastewater (including septic) when it reduces risk and improves affordability.

**Geography:** Ontario & Michigan
**Email:** hello@urbanblueproject.com
**Phone:** TBD

---

## Tech Stack

- **HTML / CSS / JS only** — no frameworks, no build step, no package manager
- **Hosted on Cloudflare Pages** via GitHub auto-deploy
- **Forms:** Formspark (embed via form `action` URL)
- **No CMS** for now (Sanity may be added later for a portfolio section)

### Why plain HTML/CSS/JS
This site will be maintained by the client using OpenAI Codex (an AI coding agent). Raw HTML files are the most reliable format for AI agents to read, edit, and push back to GitHub. No framework abstractions, no build tooling, no dependency management.

### Animation & UX Libraries (loaded via CDN `<script>` tags)

```html
<!-- Smooth scrolling -->
<script src="https://unpkg.com/lenis@latest/dist/lenis.min.js"></script>

<!-- Animation engine -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/gsap.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/gsap/3.12.5/ScrollTrigger.min.js"></script>
```

**GSAP** — handles all scroll-triggered animations: fade-ins, slide-ups, staggered reveals, parallax. This is what Apple.com uses.
**Lenis** — buttery smooth momentum scrolling. Creates the premium "weight" to scrolling that signals a high-end site.

These are script tags, not dependencies. They don't require npm, a build step, or any tooling. The client (or Codex) never needs to manage them.

### File Structure
```
urban-blue-project/
├── CLAUDE.md              ← this file (project brief for AI agents)
├── index.html             ← Home page
├── about.html             ← About page (Phase 2)
├── contact.html           ← Contact page (Phase 2)
├── css/
│   └── styles.css         ← All styles, single file
├── js/
│   └── main.js            ← Lenis init, GSAP animations, mobile nav, form
├── images/
│   ├── logo.svg           ← Urban Blue logo (compass "B" mark)
│   ├── hero.jpg           ← Hero image (modern modular housing)
│   └── ...
└── README.md              ← Setup/deploy instructions for client
```

---

## Design Direction

### The Brief
Think Apple product pages meets a high-end development firm. Ultra-clean, confident, lots of breathing room. The design should feel like it was made by a team that charges $200k+ for their services — because they do.

Two sources merged:
1. **Webflow template** — Use its hero section: dark navy background on the left, large hero image on the right, white text. Also use its nav style (logo + text links + CTA button).
2. **ChatGPT mockups** — Use their white/off-white, card-based layout for every section below the hero. Alternating light backgrounds. Rounded-corner cards with subtle borders.

### Design Principles
- **Restraint over decoration.** Every element earns its place. No gradients, no ornamentation, no visual noise.
- **Generous whitespace.** Sections should breathe. When in doubt, add more space, not more content.
- **Typography does the heavy lifting.** Large, confident headings. Clean body text. The type hierarchy alone should create visual interest.
- **Motion is subtle and purposeful.** Elements reveal on scroll with a slight upward drift and fade. Timing is ~0.6–0.8s with ease-out curves. Nothing bounces, nothing overshoots, nothing draws attention to itself.
- **Cards are information containers, not decorative elements.** Minimal styling — white background, thin border or very subtle shadow, rounded corners. Content-first.

---

## Design System

### Colors
```css
:root {
  --navy:       #0f2b3c;
  --navy-light: #1a3a4f;
  --white:      #ffffff;
  --off-white:  #f7f8fa;
  --gray-100:   #e8ecf0;
  --gray-300:   #c5cdd5;
  --gray-500:   #6b7a87;
  --gray-700:   #3d4852;
  --gray-900:   #1a1f24;
}
```
Navy is the only brand color. Everything else is grayscale. This constraint creates the premium feel.

### Typography
Use **DM Sans** from Google Fonts (or a similar geometric sans-serif with good weight range).

```css
/* Import */
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:ital,opsz,wght@0,9..40,300;0,9..40,400;0,9..40,500;0,9..40,600;0,9..40,700;0,9..40,800&display=swap');

/* Scale */
--font-family:    'DM Sans', sans-serif;
--text-xs:        0.75rem;    /* 12px — tags, fine print */
--text-sm:        0.875rem;   /* 14px — nav, captions */
--text-base:      1rem;       /* 16px — body */
--text-lg:        1.125rem;   /* 18px — lead paragraphs */
--text-xl:        1.25rem;    /* 20px — card titles */
--text-2xl:       1.5rem;     /* 24px — section subtitles */
--text-3xl:       2rem;       /* 32px — H2 mobile */
--text-4xl:       2.5rem;     /* 40px — H2 desktop */
--text-5xl:       3.25rem;    /* 52px — H1 */

/* Weights */
--font-regular:   400;
--font-medium:    500;
--font-semibold:  600;
--font-bold:      700;
--font-extrabold: 800;
```

**Heading style:** var(--font-extrabold), var(--navy), tight letter-spacing (-0.02em), line-height 1.1
**Body style:** var(--font-regular), var(--gray-700), line-height 1.6
**Bold inline text:** var(--font-semibold) with color var(--gray-900) — used in body copy for emphasis (e.g., "approvals", "servicing strategy")

### Spacing
```css
--space-xs:   0.5rem;   /* 8px */
--space-sm:   1rem;     /* 16px */
--space-md:   1.5rem;   /* 24px */
--space-lg:   2rem;     /* 32px */
--space-xl:   3rem;     /* 48px */
--space-2xl:  5rem;     /* 80px */
--space-3xl:  6.5rem;   /* 104px */

--container:  1200px;   /* max content width */
--container-narrow: 960px;  /* for text-heavy sections */
```
Sections use `padding: var(--space-3xl) 0` on desktop, `padding: var(--space-xl) 0` on mobile.

### Component Patterns

**Cards**
```css
.card {
  background: var(--white);
  border: 1px solid var(--gray-100);
  border-radius: 16px;
  padding: var(--space-lg);
}
```
No box-shadow by default. Optional very subtle shadow on hover: `box-shadow: 0 4px 24px rgba(0,0,0,0.06)`.

**Primary Button**
```css
.btn-primary {
  background: var(--navy);
  color: var(--white);
  border: none;
  border-radius: 100px;
  padding: 14px 32px;
  font-weight: var(--font-semibold);
  font-size: var(--text-sm);
  cursor: pointer;
  transition: background 0.3s ease;
}
.btn-primary:hover {
  background: var(--navy-light);
}
```

**Secondary Button**
```css
.btn-secondary {
  background: transparent;
  color: var(--navy);
  border: 1.5px solid var(--navy);
  border-radius: 100px;
  padding: 14px 32px;
  font-weight: var(--font-semibold);
  font-size: var(--text-sm);
  cursor: pointer;
  transition: all 0.3s ease;
}
.btn-secondary:hover {
  background: var(--navy);
  color: var(--white);
}
```

**Tag/Pill**
```css
.tag {
  display: inline-block;
  background: var(--off-white);
  border: 1px solid var(--gray-100);
  border-radius: 100px;
  padding: 4px 14px;
  font-size: var(--text-xs);
  font-weight: var(--font-medium);
  color: var(--gray-500);
}
```

**Section divider** — thin navy bar spanning full width between major sections:
```css
.section-divider {
  width: 100%;
  height: 3px;
  background: var(--navy);
}
```

---

## Animation System

All animations use GSAP + ScrollTrigger. Initialize Lenis for smooth scrolling.

### Lenis Setup (in main.js)
```javascript
const lenis = new Lenis({
  duration: 1.2,
  easing: (t) => Math.min(1, 1.001 - Math.pow(2, -10 * t)),
  smooth: true,
});

function raf(time) {
  lenis.raf(time);
  requestAnimationFrame(raf);
}
requestAnimationFrame(raf);

// Sync Lenis with GSAP ScrollTrigger
lenis.on('scroll', ScrollTrigger.update);
gsap.ticker.add((time) => lenis.raf(time * 1000));
gsap.ticker.lagSmoothing(0);
```

### Animation Patterns

**Fade-up reveal (default for most elements):**
```javascript
gsap.utils.toArray('.reveal').forEach(el => {
  gsap.from(el, {
    y: 40,
    opacity: 0,
    duration: 0.8,
    ease: 'power3.out',
    scrollTrigger: {
      trigger: el,
      start: 'top 85%',
      toggleActions: 'play none none none'
    }
  });
});
```

**Staggered card reveals (for card grids):**
```javascript
gsap.utils.toArray('.card-grid').forEach(grid => {
  const cards = grid.querySelectorAll('.card');
  gsap.from(cards, {
    y: 50,
    opacity: 0,
    duration: 0.7,
    stagger: 0.12,
    ease: 'power3.out',
    scrollTrigger: {
      trigger: grid,
      start: 'top 80%',
    }
  });
});
```

**Hero entrance (on page load, not scroll-triggered):**
```javascript
const heroTl = gsap.timeline({ delay: 0.2 });
heroTl
  .from('.hero-tag', { y: 20, opacity: 0, duration: 0.5, ease: 'power2.out' })
  .from('.hero-heading', { y: 30, opacity: 0, duration: 0.7, ease: 'power3.out' }, '-=0.2')
  .from('.hero-body', { y: 20, opacity: 0, duration: 0.6, ease: 'power2.out' }, '-=0.3')
  .from('.hero-buttons', { y: 20, opacity: 0, duration: 0.5, ease: 'power2.out' }, '-=0.2')
  .from('.hero-image', { scale: 1.05, opacity: 0, duration: 1, ease: 'power2.out' }, '-=0.6');
```

### Animation Rules
- **Never use bounce or elastic easing.** Always `power2.out` or `power3.out`.
- **Duration range:** 0.5s–1.0s. Nothing faster, nothing slower.
- **Stagger:** 0.1–0.15s between items. Enough to feel sequential, not enough to feel slow.
- **Direction:** Almost always `y: 30–50` (upward drift). Never from the sides.
- **Start trigger:** `top 80–85%` — elements reveal when ~15–20% visible.
- **One-shot only.** `toggleActions: 'play none none none'` — elements animate in once. No reverse on scroll-back.

### CSS for animation-ready elements
```css
/* Smooth transitions for interactive elements */
a, button, .card {
  transition: all 0.3s ease;
}

/* Smooth scroll behavior fallback */
html {
  scroll-behavior: smooth;
}

/* Lenis overrides */
html.lenis, html.lenis body {
  height: auto;
}
.lenis.lenis-smooth {
  scroll-behavior: auto !important;
}
```

---

## Responsive Breakpoints

```css
/* Mobile first — default styles = mobile (< 768px) */

@media (min-width: 768px)  { /* Tablet  */ }
@media (min-width: 1024px) { /* Desktop */ }
@media (min-width: 1280px) { /* Large   */ }
```

- Nav: hamburger on mobile, full links on desktop
- Hero: stacked on mobile (text above image), split 50/50 on desktop
- Card grids: 1 column mobile → 2 columns tablet → 2-3 columns desktop
- Section padding reduces on mobile
- Heading sizes scale down ~20-30% on mobile

---

## Home Page Sections (index.html)

Build this page first. Get client sign-off before proceeding to About and Contact.

### Section 1: Navigation
- **Layout:** Logo (compass "B" mark + "Urban Blue Project" + "Ontario • Michigan" subtitle) left. Three text links: Home, About, Contact. CTA button right: "Book a feasibility call →"
- **Style:** White background. Links in var(--gray-700), active in var(--navy). CTA is btn-primary.
- **Behavior:** Sticky on scroll with subtle background blur. Hamburger on mobile with slide-down menu.
- **Animation:** None on nav itself. Mobile menu slides down with 0.3s ease.
- **Reference:** Webflow template nav (simplified to 3 pages)

### Section 2: Hero
- **Layout:** Split — left ~50% dark navy background, right ~50% full-height image. Full viewport height or close to it.
- **Left side content:**
  - Tag pill: "Affordable missing-middle delivery • Ontario & Michigan"
  - H1: "Affordable missing-middle housing—delivered faster on constrained sites."
  - Body: "Urban Blue Project is a development management firm. We integrate **approvals**, **servicing strategy**, and **delivery method** to make small multi-family and low-density mixed-use projects viable—often pairing **off-site construction (manufactured/modular)** with **decentralized water and wastewater (including septic)** when it reduces risk and improves affordability."
  - Buttons: "Book a feasibility call" (primary, white bg on dark) + "See the model" (outline, white border on dark)
- **Right side:** hero.jpg, cover-fit, full height. Subtle scale-in on load.
- **Below hero (still part of hero section, white bg):** Three mini feature cards:
  - "Off-site Delivery Strategy" — Manufacturer fit, procurement, logistics, install sequencing.
  - "Decentralized Services" — Water + wastewater/septic matched to constraints and approvals.
  - "Affordability Discipline" — Phasing, unit mix, risk controls, lender-ready decisions.
- **Animation:** Staggered entrance timeline on page load (see hero animation pattern above).

### Section 3: Promise Bar
- Full-width, light background
- "Promise: feasibility → approvals clarity → servicing strategy → off-site delivery plan → execution support."
- Small text, centered, arrows between items
- Class: `.reveal`

### Section 4: Why Us — Problem Statement
- **Background:** var(--off-white)
- **Layout:** Left = heading + body. Right = 2×2 card grid.
- **Left:**
  - H2: "Servicing constraints and timelines are killing affordable missing-middle deals."
  - Body: "Most projects don't fail because the architecture is wrong. They fail because the **approvals pathway is unclear**, **servicing constraints surface too late**, and traditional construction adds cost variability that breaks affordability."
- **Right — 4 cards:**
  - "Municipal servicing timing" — Capacity and upgrades don't align with your pro forma timeline.
  - "Late-stage surprises" — Water/wastewater requirements derail approvals and budgets when found too late.
  - "Construction volatility" — Labour, sequencing delays, and change orders crush affordability targets.
  - "Stakeholder misalignment" — Without governance, "simple" projects become multi-year stalls.
- **Below cards:** "Our approach: treat **servicing** and **delivery method** as first-order feasibility drivers—before you overspend on design."

### Section 5: How We Make It Bankable
- **Background:** var(--white)
- **Layout:** Left = heading + body. Right = card grid (2×2 + 1).
- **Left:**
  - H2: "How we make off-site + decentralized systems bankable"
  - Body: "These methods can reduce risk—or introduce it—depending on how they're structured. We manage the details that typically sink projects."
- **Right — 5 cards:**
  - "Stage-gated decisions" — No major commitments before feasibility gates clear.
  - "Approvals clarity" — Who approves what, when, and with what dependencies.
  - "O&M model" — Operator/maintenance plan for decentralized systems.
  - "Lifecycle costing" — O&M + replacement reserves inside the pro forma.
  - "Procurement discipline" — Lead times, scope lock, interfaces, logistics, inspections—planned upfront.
- **CTA bar:** "Outcome: fewer surprises, faster decisions, tighter affordability control." + "Start with feasibility"

### Section 6: Delivery Model
- **Background:** var(--off-white)
- **H2:** "The Urban Blue delivery model"
- **Subtitle:** "We unlock constrained sites and protect affordability by integrating three elements into one accountable plan."
- **3 cards:** Off-site construction (tag: Manufactured/Modular), Decentralized servicing (tag: Water/Wastewater/Septic), Affordability viability (tag: Deal discipline) — each with "Why it matters" + bullet list.
- **CTA bar:** "Not every site needs off-site or decentralized systems..." + "Book a feasibility call"

### Section 7: Services
- **Background:** var(--white)
- **H2:** "Services built around off-site + decentralized servicing"
- **4 cards (2×2):** Constraint-First Feasibility Sprint (tag: 2–4 weeks), Approvals + Servicing Pathway, Off-site Delivery Strategy + Procurement, Execution Support / Owner's Rep — each with subtitle + bullets.
- **Buttons:** "Book a feasibility call" + "Request capability statement"

### Section 8: Resources / Proof
- **Background:** var(--off-white)
- **H2:** "Proof you can review today"
- **3 resource cards** + NDA case study CTA bar

### Section 9: Insights
- **Background:** var(--white)
- **H2:** "Practical insight for missing-middle delivery"
- **3 article cards** (placeholder content, link to #)

### Section 10: Form CTA
- **Background:** var(--white), wrapped in large rounded container
- **Left:** H2 + body + info cards + buttons + fine print
- **Right:** Formspark form (Name, Email, Geography dropdown, Project type dropdown, Constraints textarea, Submit)

### Section 11: Footer
- 3-column: brand info, site links, external links
- "© 2026 Urban Blue Project. All rights reserved."

---

## Phase 2 Pages

### About Page (about.html)
Nav → Page header → Sectors (4 cards) → Team bios → Collaborators → Form CTA → Footer

### Contact Page (contact.html)
Nav → Full-width form → Direct contact details → Footer

---

## Form Integration (Formspark)

```html
<form action="https://submit-form.com/FORM_ID_HERE" method="POST">
  <input type="hidden" name="_redirect" value="https://urbanblueproject.com/thank-you">
  <!-- fields -->
</form>
```

Replace `FORM_ID_HERE` with actual Formspark form ID.

---

## Assets Needed from Client

- [ ] Logo files (SVG preferred)
- [ ] Hero image (high-res modern housing)
- [ ] Team headshots + bios (About page)
- [ ] Downloadable PDFs (Resources section)
- [ ] "Book a feasibility call" destination (Calendly? Scroll to form?)
- [ ] LinkedIn URL
- [ ] Confirmed phone number
- [ ] Privacy / terms text

---

## Notes for Future Maintainer (Codex / AI Agent)

This site uses zero build dependencies. Every page is a standalone HTML file with shared CSS and JS via relative paths.

- **Text:** Edit HTML directly in `.html` files.
- **Styles:** Edit `css/styles.css`. All design tokens are CSS custom properties in `:root`.
- **Animations:** Edit `js/main.js`. GSAP animations grouped by section.
- **New page:** Copy existing page, update `<main>`, add nav link to all pages.
- **Form:** Edit `<form>` HTML for fields, update `action` URL for destination.

**Do not** introduce frameworks, build tools, or package managers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/inbound-bailey)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/inbound-bailey)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
