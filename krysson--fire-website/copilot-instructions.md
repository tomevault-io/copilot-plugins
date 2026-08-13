## fire-website

> This is the FIRE convention website - a static Next.js site for a rope bondage educational organization hosting three annual events in Orlando, FL.

# CLAUDE.md - Instructions for Claude Code

## Project Context

This is the FIRE convention website - a static Next.js site for a rope bondage educational organization hosting three annual events in Orlando, FL.

**Key constraint:** Keep it simple. This is a static marketing/information site with no auth, no database, no payments.

---

## Tech Stack

```
Framework:    Next.js 14+ (App Router)
Styling:      Tailwind CSS
Components:   shadcn/ui
Language:     TypeScript
Content:      Markdown + JSON files
Hosting:      Vercel
```

---

## Project Structure

```
fire-website/
├── app/
│   ├── layout.tsx              # Root layout with nav/footer
│   ├── page.tsx                # Homepage
│   ├── about/page.tsx          # About FIRE org
│   ├── faq/page.tsx            # FAQ & Policies
│   ├── contact/page.tsx        # Contact info
│   ├── sponsors/page.tsx       # Sponsors & vendors
│   ├── past-events/page.tsx    # Archive (placeholder)
│   └── events/
│       ├── blaze-2026/
│       │   ├── page.tsx        # BLAZE landing
│       │   ├── schedule/page.tsx
│       │   ├── presenters/page.tsx
│       │   ├── presenters/[slug]/page.tsx
│       │   ├── classes/page.tsx
│       │   └── venue/page.tsx
│       ├── flare-2026/
│       │   └── ...
│       └── fire-2027/
│           └── ...
├── components/
│   ├── ui/                     # shadcn components
│   ├── layout/
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── Navigation.tsx
│   ├── home/
│   │   ├── Hero.tsx
│   │   └── EventCards.tsx
│   └── events/
│       ├── ScheduleGrid.tsx
│       ├── PresenterCard.tsx
│       ├── ClassCard.tsx
│       └── VenueMap.tsx
├── content/
│   ├── blaze-2026/
│   │   ├── event.json
│   │   ├── schedule.json
│   │   ├── sponsors.json
│   │   ├── presenters/
│   │   └── classes/
│   ├── flare-2026/
│   ├── fire-2027/
│   └── organization/
│       ├── config.json         # SINGLE SOURCE OF TRUTH: contact emails, social links,
│       │                       # homepage event cards, class level colors
│       ├── about.md
│       ├── faq.md
│       └── policies.md
├── lib/
│   ├── content.ts              # Content loading utilities
│   └── utils.ts                # Helper functions
├── public/
│   ├── logos/
│   │   ├── blaze-2025.png      # Update year to 2026 when received
│   │   ├── fire-logo.png
│   │   └── flower-flat.png
│   └── images/
│       └── presenters/
└── styles/
    └── globals.css
```

---

## Coding Standards

### General
- Use TypeScript strict mode
- Prefer server components; use 'use client' only when needed
- Keep components small and focused
- Use descriptive variable names

### Styling
- Tailwind CSS for all styling
- Use CSS variables for brand colors (defined in globals.css)
- Mobile-first responsive design
- Consistent spacing using Tailwind scale

### Content
- Load markdown with gray-matter for frontmatter
- Parse markdown with remark/rehype
- Type all JSON content with interfaces

---

## Brand Colors (Tailwind Config)

```javascript
// tailwind.config.ts
colors: {
  fire: {
    black: '#0a0a0a',
    charcoal: '#1a1a1a',
    dark: '#2a2a2a',
    red: '#e63946',
    orange: '#f4a261',
    yellow: '#f9c74f',
  }
}
```

---

## Content File Formats

### config.json (Single Source of Truth)

`content/organization/config.json` controls site-wide settings. Edit this file — not component code — for:

- **Contact emails** (`contact.general`, `contact.presenters`, `contact.volunteers`, `contact.vendors`)
- **Social media links** (`social.fetlife`, `social.instagram`, `social.facebook`, `social.tiktok`)
- **Homepage event cards** (`homepage.events[]`) — dates, focus level, badge color, ticket slug, logo
- **Featured event** (`homepage.featuredEventId`) — which event shows the "Next Event" banner
- **Class level badge colors** (`classLevels[]`)

Never hard-code these values in component files. Always read from `getSiteConfig()`.

---

### event.json
```json
{
  "name": "BLAZE",
  "year": 2026,
  "tagline": "Ignite Your Rope Journey",
  "description": "A short paragraph about the event.\n\nUse \\n\\n to separate paragraphs. HTML tags like <strong> and <em> are supported.",
  "dates": {
    "display": "April 17-19, 2026",
    "start": "2026-04-17",
    "end": "2026-04-19"
  },
  "focus": "Beginner to Intermediate",
  "venue": {
    "name": "The Woodshed",
    "address": "6431 Milner Blvd Suite #4, Orlando, FL 32809"
  },
  "tickets": {
    "url": "https://forbiddentickets.com/events/...",
    "onSaleDate": "February 14, 2026"
  }
}
```

> **`description` formatting:** Use `\n\n` (double backslash-n) between paragraphs to create line breaks. HTML tags such as `<strong>`, `<em>`, and `<br>` are rendered. The `dates.start` ISO value drives the homepage countdown timer.

### schedule.json
```json
{
  "days": [
    {
      "date": "2026-04-17",
      "label": "Friday",
      "slots": [
        {
          "time": "7:00 PM",
          "title": "Registration Opens",
          "type": "general"
        },
        {
          "time": "8:00 PM",
          "title": "Intro to Floor Work",
          "presenter": "desmond-ellise",
          "room": "Main Room",
          "type": "class"
        }
      ]
    }
  ]
}
```

### Presenter Markdown
```markdown
---
name: Desmond & Ellise
slug: desmond-ellise
pronouns: he/him & she/her
photo: /images/presenters/desmond-ellise.jpg
social:
  fetlife: https://fetlife.com/users/8036439
---

Des and Ellise have both been practicing rope since 2018...
```

### sponsors.json
```json
{
  "sponsors": [
    {
      "tier": "gold",
      "name": "Sponsor Name",
      "logo": "/images/sponsors/sponsor-name.png",
      "url": "https://example.com",
      "tagline": "Short tagline — shown on Gold/Silver/Bronze cards"
    },
    {
      "tier": "silver",
      "name": "Another Sponsor",
      "logo": "/images/sponsors/another-sponsor.png",
      "url": "https://example.com"
    },
    {
      "tier": "community",
      "name": "Community Friend",
      "logo": "/images/sponsors/community-friend.png",
      "url": "https://example.com"
    }
  ]
}
```

**Tiers** (controls display size and position):
- `gold` — largest cards, 2-column grid
- `silver` — medium cards, 3-column grid
- `bronze` — smaller cards, 4-column grid
- `community` — smallest cards, 5-column grid

**Fields:**
- `tier` — required. One of `gold`, `silver`, `bronze`, `community`
- `name` — required. Display name shown below the logo
- `logo` — required. Path to logo image in `/public/images/sponsors/`
- `url` — required. Link to sponsor's website (opens in new tab)
- `tagline` — optional. Short phrase shown in italics below the name. Use for Gold/Silver/Bronze; omit for Community tier

If `sponsors.json` is empty or the `sponsors` array has no entries, the page shows "Sponsors will be announced soon."

Logo images should be placed in `public/images/sponsors/`. Transparent PNG works best; aim for consistent aspect ratios within a tier.

---

### Class Markdown
```markdown
---
title: Introduction to Floor Work
slug: intro-floor-work
presenter: desmond-ellise
level: Beginner
duration: 90 minutes
---

This class covers the fundamentals of floor-based rope bondage...
```

---

## Key Components

### Navigation
- Sticky header with logo
- Desktop: horizontal nav with event dropdown
- Mobile: hamburger menu with slide-out drawer
- Highlight current event based on route

### Event Landing Pages
- Hero with event logo and key info
- Quick stats (dates, focus, venue)
- Prominent ticket CTA button
- Links to schedule, presenters, classes, venue

### Schedule Display
- Tab or button group for day selection
- Time-based list or grid
- Click to view class details
- Mobile-friendly layout

### Presenter Cards
- Photo with fallback placeholder
- Name and pronouns
- Brief excerpt of bio
- Link to full profile

---

## External Links

`ticketEventSlug` in `content/organization/config.json` is the **event folder name** (e.g. `"flare-2026"`), not a Forbidden Tickets path. The code calls `getEventData(ticketEventSlug)` to load that event's `event.json` and reads `tickets.url` from it. The actual Forbidden Tickets URL lives in `content/[event]/event.json` under `tickets.url`.

Leave `ticketEventSlug` blank (`""`) to hide the ticket button for that event on the homepage and show "Learn More" instead.

---

## Environment Variables

```env
# .env.local (if needed)
NEXT_PUBLIC_SITE_URL=https://fireorlando.com
```

Contact emails and social links are stored in `content/organization/config.json`, not environment variables.

---

## Deployment

1. Push to GitHub main branch
2. Vercel auto-deploys from GitHub
3. Custom domain: fireorlando.com configured in Vercel

---

## Content Updates

To add/update content:

1. **Contact emails / social links / homepage cards:** Edit `content/organization/config.json`
2. **New presenter:** Add markdown file to `/content/[event]/presenters/`
3. **New class:** Add markdown file to `/content/[event]/classes/`
4. **Update schedule:** Edit `/content/[event]/schedule.json`
5. **Event details (dates, venue, tagline, tickets):** Edit `/content/[event]/event.json`
6. **New images:** Add to `/public/images/`
7. Commit and push - Vercel rebuilds automatically

---

## Common Tasks

### Add a new presenter
```bash
# Create file: content/blaze-2026/presenters/new-presenter.md
# Add photo: public/images/presenters/new-presenter.jpg
# Update schedule.json if they're teaching
```

### Add or update a sponsor
```bash
# Edit: content/[event]/sponsors.json
# Add logo image: public/images/sponsors/[sponsor-name].png
# Tiers: gold, silver, bronze, community
# tagline field is optional (Gold/Silver/Bronze only)
```

### Roll to the next event (e.g. BLAZE is over, FLARE is next)
```bash
# 1. Edit content/organization/config.json:
#    - Set featuredEventId to the new event's id (e.g. "flare")
#    - Set the old event's ticketEventSlug to "" (removes its ticket button)
#    - Set the new event's ticketEventSlug to its folder name (e.g. "flare-2026")
#
# 2. Edit content/[new-event]/event.json:
#    - Confirm tickets.url is the correct Forbidden Tickets URL
#    - Confirm dates.start is correct (drives the homepage countdown timer)
```

### Update contact email or social links
```bash
# Edit: content/organization/config.json
# Update the relevant field under "contact" or "social"
```

---
> Source: [Krysson/fire-website](https://github.com/Krysson/fire-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
