## the-spot-catering

> Official website for **The Spot Catering** - Denver's Corporate Catering Queen, owned by Mandy Smith.

# The Spot Catering - Website & Power Hub

## Project Overview
Official website for **The Spot Catering** - Denver's Corporate Catering Queen, owned by Mandy Smith.

- **Live Site**: https://the-spot-catering.vercel.app
- **GitHub**: https://github.com/BrettLechtenbrerg/the-spot-catering
- **Vercel Project**: the-spot-catering (bretts-projects-3e254e58)
- **Future Home**: Will move to Mandy's GitHub (CrockSpotCatering) when complete

## Tech Stack
- **Framework**: Next.js 16 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS 3.4
- **Animations**: Framer Motion
- **Icons**: Lucide React
- **Deployment**: Vercel

## Brand Colors
```
Primary Orange:    #FAAA44  (spot-orange - main accent, CTAs)
Orange Hover:      #E89A35  (spot-orange-hover)
Deep Navy:         #262262  (spot-navy - primary text, headers)
Navy Light:        #3A3A8C  (spot-navy-light)
Purple/Magenta:    #9E1F63  (spot-purple - secondary accent)
Purple Hover:      #B82573  (spot-purple-hover)
Teal:              #8CBEC0  (spot-teal - light accents)
Blue-Gray:         #879FAF  (spot-blue-gray - subtle backgrounds)
Orange-Red:        #D34F1D  (spot-orange-red - warm highlights)
```

## Brand Typography
- **Display Font**: Oswald (headings, titles)
- **Body Font**: Inter (body text, paragraphs)

## Taglines & Puns (Mandy loves these!)
- "We know how to hit the spot"
- "Spot on catering for any occasion"
- "You've found your spot"
- "Helping your corporate events stay on focus and hit the spot"
- "More espresso, less depresso" (breakfast events)

## Business Info
- **Owner**: Mandy Smith - The Corporate Catering Queen
- **Phone**: 925-699-6629
- **Email**: spotcafes@gmail.com
- **Location**: Denver, Colorado
- **Social**:
  - Facebook: https://www.facebook.com/thespotcafes
  - Instagram: https://www.instagram.com/spotcafes/

## Certifications (Important selling points!)
- **MWBE**: Minority Women's Business Enterprise
- **DBE**: Disadvantaged Business Enterprise
- **EBE**: Emerging Business Enterprise
- **SBEC**: Small Business Enterprise Concessionaire
- **LLC**: MNS Worldwide - Majority Woman Owned Business

## Website Pages

| Page | Route | Status | Features |
|------|-------|--------|----------|
| Home | `/` | ✅ Complete | Hero with featured image, services, stats, menu preview, CTAs |
| Corporate | `/corporate` | ✅ Complete | Services grid with greyscale images, benefits, certifications |
| Themed Events | `/themes` | ✅ Complete | 8 theme cards with greyscale images, holiday list, 3-step process |
| Pop-Up Events | `/pop-up` | ✅ Complete | 4 pop-up types with greyscale images, benefits, social CTAs |
| Menus | `/menus` | ✅ Complete | 4 categories with gradient cards + images (no prices), extras |
| About | `/about` | ✅ Complete | Personal story, timeline, Mandy's photo, values, certifications |
| Privacy | `/privacy` | ✅ Complete | Privacy policy with Colorado Privacy Act compliance |
| Terms | `/terms` | ✅ Complete | Terms of service with catering-specific clauses |
| Contact | `/contact` | ✅ Complete | Form, contact info, social links |

## Visual Design Features

### Greyscale Image Cards (Used across site)
- Cards have greyscale background images that colorize on hover
- Dark gradient overlays for text readability
- Smooth scale-up animation on hover
- Used on: Corporate, Themes, Pop-Up pages

### Menus Page (Spicy Design!)
- Full-bleed gradient cards with greyscale background images
- Vibrant gradient overlays: amber (Breakfast), emerald (Lunch), purple (Happy Hour), red (Themed)
- Animated icons that rotate on hover
- Flame icon accents next to taglines
- White menu item cards with orange left border

## Service Categories

### Corporate Events
- Team Training Events/Workshops
- Team Appreciation/Retreat Events
- Board Meetings
- Client Meetings

### Meal Types
- Breakfast Boards
- Breakfast Burrito Platter
- In Room Coffee Service
- Individually Packaged Lunches
- Buffet Style Lunch

*Note: Prices are not displayed on the website — contact for custom quotes.*

### Themed Events
- Holiday Parties (all major holidays)
- Taco Bars, Chili Bars
- BBQ Events
- Luau
- Oktoberfest
- And more!

### Happy Hour/Special
- Charcuterie Boards
- Grazing Boards
- After-hours meetings

## Project Status

- [x] GitHub repo created
- [x] Project folder set up
- [x] CLAUDE.md created
- [x] Tech stack configured
- [x] Vercel project connected
- [x] Homepage built (with featured image hero)
- [x] All 9 pages built (including Privacy & Terms)
- [x] Images uploaded (13 images including Mandy's photo)
- [x] SEO optimized (metadata in layout.tsx)
- [x] Mobile responsive (Tailwind responsive classes)
- [x] Prices removed from all pages (contact for quotes)
- [x] Mandy's photo added to About page
- [x] Greyscale image cards on Corporate, Themes, Pop-Up pages
- [x] Spicy gradient design on Menus page
- [x] Footer logo fixed (no white square)
- [ ] Contact form backend (currently frontend-only)
- [ ] Analytics integration
- [ ] Custom domain
- [ ] Verify pandemic/National Guard reference with Mandy
- [ ] Ready for Mandy's GitHub transfer

## Components

| Component | File | Purpose |
|-----------|------|---------|
| Header | `components/Header.tsx` | Navigation with top bar, mobile menu |
| Footer | `components/Footer.tsx` | 4-column footer with links, contact, certs (logo rounded-full) |
| HeroSection | `components/HeroSection.tsx` | Reusable hero with customizable props |

## Key Files
```
/app
  layout.tsx              # Root layout with SEO metadata
  globals.css             # Tailwind + custom styles
  page.tsx                # Homepage
  /corporate/page.tsx     # Corporate catering (greyscale image cards)
  /themes/page.tsx        # Themed events (8 greyscale image cards)
  /pop-up/page.tsx        # Pop-up events (4 greyscale image cards)
  /menus/page.tsx         # Menu options (gradient cards, no prices)
  /about/page.tsx         # About Mandy & story
  /contact/page.tsx       # Contact form
  /privacy/page.tsx       # Privacy policy
  /terms/page.tsx         # Terms of service
/components
  Header.tsx              # Site header
  Footer.tsx              # Site footer
  HeroSection.tsx         # Reusable hero
/public/images            # 13 photos (including meet-mandy.jpg)
```

## Deployment Commands

```bash
# Navigate to project
cd /Users/brettlechtenberg/Documents/agent-girl/chat-dd71e03e/the-spot-catering

# Development
npm install
npm run dev              # localhost:3000

# Deploy to production
vercel --prod --yes

# Git commit
git add -A && git commit -m "Description" && git push origin main
```

## Assets Location
- **Project (current)**: `/Users/brettlechtenberg/Documents/agent-girl/chat-dd71e03e/the-spot-catering/`
- **Branding Source**: `/Users/brettlechtenberg/Desktop/Mandy Smith/`

## Mandy's Story (Reference)
Mandy started in sales and marketing for Crock Spot. She saw an empty cafe at a catering job and took the spot - turning one location into three restaurants. Her love for catering led her to downsize to one restaurant plus dedicated catering operations, focusing on making people feel special through creative catering experiences.

This pivot happened one month before COVID hit. As the dining landscape changed, Mandy saw opportunity and rolled with it. A woman-owned business that bet on herself and created a thriving corporate catering empire.

**Mission**: We make food fun, social, and special for everyone.

## Session Notes

### April 3, 2026 Session
- Fixed broken Oktoberfest and Luau images on Themes page
- Added spicy gradient design to Menus page with animated icons
- Added greyscale image cards to Pop-Up Events page (4 cards)
- Fixed Community Events and Flash Sales images
- All external images use Unsplash URLs

### Pending Verification
- **Pandemic Reference**: The Lunch menu mentions "serving the National Guard meals for 36 days straight during the Pandemic" - verify this is accurate with Mandy before launch

---

**Last Updated**: April 3, 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/BrettLechtenbrerg)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/BrettLechtenbrerg)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
