## leftclick-agency

> Multi-page marketing website for LeftClick, an AI automation agency. Static HTML files with embedded CSS and JavaScript—no build step required.

# LeftClick Agency Website

## Project Overview

Multi-page marketing website for LeftClick, an AI automation agency. Static HTML files with embedded CSS and JavaScript—no build step required.

## Related Sites

- **Live Site**: https://leftclick-agency.netlify.app
- **LeftClick.ai**: https://www.leftclick.ai

## Tech Stack

- **Framework**: Static HTML (no build step)
- **Styling**: Embedded CSS
- **JavaScript**: Vanilla JS, embedded
- **Hosting**: Netlify
- **Fonts**: Inter (Google Fonts CDN)

## File Structure

```
/Users/nicksaraev/leftclick-agency/
├── index.html          # Main landing page (hero, case studies, process, services overview)
├── services.html       # Detailed services page (Lead Gen, PM, Hiring, Sales Admin systems)
├── about.html          # About page (founders, story, timeline, values, press)
├── contact.html        # Contact page (form, founder section)
├── netlify.toml        # Netlify configuration
├── .gitignore          # Git ignore rules
├── CLAUDE.md           # This file
└── .netlify/           # Netlify local state (gitignored)
```

## Git Workflow

### Branch Naming Convention

Feature branches use the `feature/` prefix:
- `feature/services-page`
- `feature/about-page`
- `feature/contact-page`

### Merging Feature Branches

When merging multiple feature branches into main:

```bash
# 1. Check current status and list branches
git status
git branch -a

# 2. Merge each feature branch
git merge feature/services-page -m "Merge feature/services-page: description"
git merge feature/about-page -m "Merge feature/about-page: description"
git merge feature/contact-page -m "Merge feature/contact-page: description"

# 3. Push to remote
git push origin main
```

### IMPORTANT: Handling index.html Conflicts

Feature branches often use `index.html` as their main file during development. When merging, this can **overwrite the main landing page**.

**After merging, always verify:**
1. `index.html` contains the **main landing page** (title: "LeftClick | AI Automation Agency")
2. Feature content is in its **dedicated file** (e.g., `services.html`, not `index.html`)

**If a feature branch overwrote index.html:**
```bash
# 1. Save the feature content to its proper file
cp index.html services.html  # or about.html, contact.html

# 2. Restore the original landing page from before the merge
git show <commit-before-merge>:index.html > index.html

# 3. Commit the fix
git add index.html services.html
git commit -m "fix: Restore landing page, move services to services.html"
```

**To find the original index.html:**
```bash
# View commit history
git log --oneline

# Show index.html from a specific commit
git show f87222a:index.html | head -20
```

## Design System

### Colors
| Token | Hex | Usage |
|-------|-----|-------|
| Black base | `#000000` | Background |
| Dark gray | `#0a0a0a`, `#111111` | Cards, sections |
| Emerald primary | `#10b981` | Accents, CTAs |
| Emerald light | `#34d399` | Hover states |
| Emerald dark | `#059669` | Active states |

### Typography
- **Font**: Inter
- **Weights**: 300-800
- **Letter spacing**: -0.03em (tight)

### Corner Radii
Squared/luxe aesthetic — avoid rounded pills:
- Small: `4px`
- Medium: `6px`
- Large: `8px`

### Logo
Plain text wordmark: "LeftClick" where "Click" is rendered in emerald green (`#10b981`).

## Page Structure

### index.html (Main Landing Page)
1. **Hero** — Headline, subtitle, CTA buttons, floating tech icons
2. **Social Proof** — Scrolling logo ticker (Make, n8n, Zapier, etc.)
3. **Case Studies** — 3 cards with stats (SaaS, E-Commerce, Agency)
4. **How It Works** — 3-step process (Growth Mapping, Scope & Architecture, Build & Deploy)
5. **Services Overview** — 6 service cards (Workflow, AI, Data, Chatbots, Reporting, Custom)
6. **CTA** — Final call-to-action with Calendly link
7. **Footer** — Logo, nav links, copyright

### services.html (Detailed Services)
1. **Hero** — Stats-focused hero with key metrics
2. **Lead Generation Systems** — AI Cold Email, Application Systems, Content Systems
3. **Project Management Systems** — Automated Fulfillment, Onboarding, PM Workflows
4. **Hiring Systems** — Intake Systems, AI Scoring, Trial Processes
5. **Sales Administration** — Custom CRMs, AI Asset Generators, Nurture Systems
6. **Who It's For** — Target personas (Agencies, SaaS, Professional Services, E-Commerce)
7. **Results/Testimonials** — Client success stories
8. **Process** — 4-step timeline (Discovery, Design, Build, Launch)
9. **Pricing** — Three tiers (Single System, Growth Package, Retainer)
10. **CTA** — Final call-to-action

### about.html (About)
1. **Hero** — Company origin story headline with gradient orbs
2. **Story Section** — Company history and early AI adoption narrative
3. **Founders Section** — Profiles for Nick Saraev (CEO) and Noah Edis (COO)
4. **Timeline Section** — Key milestones from 2020-2025
5. **Values Section** — 6-card grid of company principles
6. **Press Section** — Featured media logos (Popular Mechanics, Apple News, Bloomberg, Indie Hackers)
7. **CTA** — Call-to-action linking to Calendly

### contact.html (Contact)
1. **Hero** — Contact headline
2. **Contact Form** — Name, email, company, message fields
3. **Founder Section** — Direct contact info
4. **CTA** — Calendly booking link

## Interactive Features

- Scroll-triggered reveal animations via Intersection Observer
- Animated counters with easeOutQuart easing
- Mouse-following cursor glow effect (desktop only)
- Scroll progress indicator in header
- Smooth scroll navigation
- Fully responsive down to mobile

## Deployment

### Deploy to Production
```bash
netlify deploy --prod
```

### Preview Deploy
```bash
netlify deploy
```

### Requirements
- Netlify CLI installed (`npm install -g netlify-cli`)
- Authenticated with Netlify (`netlify login`)

## External Dependencies

- **Calendly**: Meeting booking at `calendly.com/leftclick-meeting-30`
- **Google Fonts**: Inter font family via CDN
- **No npm packages** — Zero build dependencies

## Common Tasks

### Update Content
Edit the relevant HTML file directly. All content, styles, and scripts are embedded.

### Change Colors
Search for hex codes:
- Primary green: `#10b981`
- Light green: `#34d399`
- Dark green: `#059669`

### Update Calendly Link
Search for `calendly.com/leftclick-meeting-30` and replace across all files.

### Update Pricing (services.html)
Find the `.pricing-section`. Each tier has:
- `.pricing-name` — tier name
- `.pricing-amount` — price
- `.pricing-features` — feature list

### Add/Remove Services (services.html)
Each service category has a `.services-detail-grid` containing `.service-detail-card` elements.

## Notes

- Keep the single-file architecture — no bundlers or build steps
- Maintain the squared corner aesthetic (no pills)
- Test scroll animations after content changes
- Counter animations trigger on scroll into view
- **index.html is the main landing page** — don't overwrite with feature content

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nickjwells) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
