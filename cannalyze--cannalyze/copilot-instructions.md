## cannalyze

> > This file is the authoritative reference for any AI coding agent (Codex, Cursor, etc.) working on this codebase.

# AGENTS.md — Cannalyze Codebase Guide

> This file is the authoritative reference for any AI coding agent (Codex, Cursor, etc.) working on this codebase.
> Read it fully before making any changes.

---

## What This Project Is

**Cannalyze** is South Africa's cannabis intelligence platform. A single-page application (vanilla JS, no build step required) with:

- Strain discovery with terpene science
- Community reviews and posts
- Gamified learning (flashcards, quizzes, learning paths)
- AI Budtender powered by the Anthropic Claude API (claude-sonnet-4-20250514)
- User auth and persistent data via Supabase
- XP / badge / level progression system

**Stack**: Vanilla JS + CSS custom properties. No framework. No build step. Open `index.html` directly in a browser.

**Entry point**: `index.html` — imports all modules via `<script type="module">`.

---

## File Map

```
cannalyze/
├── AGENTS.md                   ← YOU ARE HERE
├── README.md                   ← Human-readable setup guide
├── index.html                  ← Entry point. All HTML structure lives here.
├── package.json                ← Dev tooling only (no runtime dependencies)
│
├── src/
│   ├── main.js                 ← App bootstrap: imports all modules, calls init()
│   │
│   ├── styles/
│   │   ├── variables.css       ← ALL CSS custom properties (colours, spacing)
│   │   ├── base.css            ← Reset, typography, scrollbar, background FX
│   │   ├── components.css      ← Reusable component styles (cards, buttons, badges)
│   │   └── pages.css           ← Page-specific styles (home, strain, community…)
│   │
│   ├── data/
│   │   ├── strains.js          ← STRAINS array — the 13 SA strains with full profiles
│   │   ├── terpenes.js         ← TERP_COLS map + TERP_DATA cards for academy
│   │   ├── community.js        ← COMM_POSTS, LEADERBOARD, CLUBS, FEED_ITEMS, BADGES_ALL
│   │   └── learn.js            ← PATHS, FLASHCARDS, QUIZ_Q, GUIDE_SCENARIOS
│   │
│   ├── lib/
│   │   ├── supabase.js         ← Supabase client init, auth helpers, all DB calls
│   │   ├── anthropic.js        ← Claude API call, persona system prompts, demo fallback
│   │   ├── xp.js               ← XP state, addXP(), level calculation, bar sync
│   │   └── router.js           ← go(page, tab) — page navigation and scroll
│   │
│   ├── components/
│   │   ├── nav.js              ← <nav> render, auth state switching
│   │   ├── strain-card.js      ← Strain grid card HTML builder
│   │   ├── terpene-wheel.js    ← SVG pie chart for terpene profile
│   │   ├── toasts.js           ← showToast(type, icon, msg) + XP pop animation
│   │   ├── modals.js           ← Auth modal, post modal — open/close/submit
│   │   └── particles.js        ← Floating particle background animation
│   │
│   └── pages/
│       ├── home.js             ← Discover page: filter bar, strain grid, sidebar
│       ├── strain-detail.js    ← Full strain page: stats, terpene wheel, reviews, write-review
│       ├── community.js        ← Posts feed, voting, leaderboard, new post
│       ├── learn.js            ← Academy tabs: paths, terps, flashcards, quiz, guide
│       ├── ai-budtender.js     ← Chat UI, send/receive, typing indicator, chip shortcuts
│       └── profile.js          ← XP bar, badge grid, privacy toggles, strain history
│
└── supabase/
    └── schema.sql              ← Full DB schema, RLS policies, triggers, seed data
```

---

## Key Conventions

### Navigation
```js
go('home')          // show #pg-home, hide others
go('strain')        // show strain detail (set currentStrain first)
go('community')     // etc.
```
`go()` is in `src/lib/router.js`. Never manipulate `.page` divs directly — always go through `go()`.

### State (global, in `src/main.js`)
```js
window.APP = {
  currentXP: 1240,
  currentFilter: 'all',
  currentStrain: null,
  reviewRating: 0,
  fcIndex: 0,
  quizIndex: 0,
  quizAnswered: false,
  likedStrains: new Set(),
  savedStrains: new Set(),
  chatHistory: [],       // Claude multi-turn messages array
  currentPersona: 'explorer',
  supabaseClient: null,
  currentUser: null,
  anthropicKey: '',
}
```
Access via `window.APP.x` from any module. Do NOT add new global variables — extend `window.APP`.

### XP System
```js
import { addXP } from './lib/xp.js'
addXP(50)   // awards XP, syncs all bars, spawns float animation
```
XP amounts: review +50, correct quiz +20, wrong quiz +5, flashcard flip +5, post +25, like strain +10, upvote post +5, onboarding +100.

### Supabase calls
All DB interaction goes through `src/lib/supabase.js`. Never call `window.APP.supabaseClient` directly from pages or components — import the helper functions instead.

```js
import { loadStrains, saveReview, savePost } from '../lib/supabase.js'
```

### Claude API
All AI calls go through `src/lib/anthropic.js`. The system prompt and persona variants live there — do not duplicate them elsewhere.

```js
import { sendMessage, getDemoResponse } from '../lib/anthropic.js'
// sendMessage(userText) → returns AI reply string
// getDemoResponse(userText) → [replyString, strainCardObject|null]
```

### Toasts
```js
import { showToast } from '../components/toasts.js'
showToast('xp', '🌿', 'Review posted! +50 XP')
showToast('badge', '⭐', 'Badge unlocked!')
showToast('err', '⚠', 'Something went wrong')
```

---

## CSS Architecture

All CSS custom properties are in `src/styles/variables.css`. **Never hardcode colours** — always use variables:

```css
/* Palette */
--ink, --deep, --forest, --panel, --card      /* backgrounds, dark to light */
--border, --border2                            /* border colours */
--em, --em2, --em3                             /* green accent (primary) */
--gold, --gold2, --gold3                       /* gold accent */
--teal, --cyan, --violet, --lav               /* cool accents */
--amber, --rose, --sky, --coral               /* warm/alert accents */
--white, --silver, --muted                    /* text hierarchy */
```

Gradient text utilities: `.org`, `.dat`, `.gld`, `.shim` — defined in `base.css`.

---

## Data Shape Reference

### Strain object (`src/data/strains.js`)
```js
{
  id: 'pink-cookies',           // slug, used for routing
  name: 'Pink Cookies',
  type: 'hybrid',               // 'indica' | 'sativa' | 'hybrid'
  emoji: '🍪',
  reviewCount: 13,              // staff review count
  commReviews: 31,              // community review count
  trending: false,
  effects: ['Relaxed', 'Euphoric', ...],
  aromas: ['Sweet', 'Floral', ...],
  benefits: ['Anxiety', 'Depression', ...],
  terpenes: {                   // percentages, should sum to ~100
    Limonene: 31,
    Caryophyllene: 28,
    Linalool: 25,
    Humulene: 16,
  },
  burn: 75,                     // 0–100 community burn quality score
  art: 'linear-gradient(...)', // CSS gradient for strain art panel
  comments: [                   // staff reviews
    { a: 'ReviewerHandle', r: 'Role/Level', t: 'Review text' }
  ]
}
```

### Post object (`src/data/community.js`)
```js
{
  id: 'optional-uuid',
  user: 'ZA_OG',
  color: '#FFD600',             // avatar background
  flair: 'OG Member',
  fc: 'rgba(255,214,0,.1)',    // flair chip background
  fb: 'rgba(255,214,0,.22)',   // flair chip border
  title: 'Post title',
  body: 'Post body text',
  strain: 'Durban Poison',     // optional strain tag
  up: 84,                      // upvote count
  cmts: 23,                    // comment count
  time: '14m ago',
}
```

---

## Common Tasks for Codex

### Add a new strain
1. Add entry to `STRAINS` array in `src/data/strains.js`
2. Add a row to `supabase/schema.sql` in the strains seed section
3. Add matching `strain_effects`, `strain_aromas`, `strain_benefits` rows

### Add a new page
1. Create `src/pages/my-page.js` with a `renderMyPage()` function
2. Add `<div id="pg-my-page" class="page">` in `index.html`
3. Add a nav tab in the `<nav>` section of `index.html`
4. Import and call `renderMyPage()` in `src/main.js`
5. Add page-specific CSS to `src/styles/pages.css`

### Change AI persona prompts
Edit `PERSONA_PROMPTS` object in `src/lib/anthropic.js`.

### Change XP amounts
Edit the XP constants at the top of `src/lib/xp.js`.

### Add a new badge
1. Add to `BADGES_ALL` in `src/data/community.js`
2. Add the badge definition to `supabase/schema.sql` badges seed
3. Trigger award via `supabaseClient.from('user_badges').insert(...)` in the relevant action

### Extend the Supabase schema
1. Add SQL to `supabase/schema.sql`
2. Add corresponding helper function to `src/lib/supabase.js`
3. Never call `supabaseClient` directly from page or component files

---

## Environment / Config

Keys are entered at runtime via the config bar in the UI and persisted to `localStorage`:

| Key | localStorage key | Purpose |
|-----|-----------------|---------|
| Anthropic API key | `cannalyze_api_key` | Live Claude AI |
| Supabase URL | `cannalyze_sb_url` | Database + auth |
| Supabase anon key | `cannalyze_sb_anon` | Database + auth |

In production, move the Anthropic key to a Supabase Edge Function proxy (see `README.md`).

---

## Do Not

- Do not introduce a build step or framework unless explicitly instructed
- Do not hardcode colour values — use CSS variables from `variables.css`
- Do not add global variables — extend `window.APP`
- Do not call Supabase directly from pages — use `src/lib/supabase.js`
- Do not call the Anthropic API directly from pages — use `src/lib/anthropic.js`
- Do not break the single-file deployability — `index.html` must work standalone if needed

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Cannalyze)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Cannalyze)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
