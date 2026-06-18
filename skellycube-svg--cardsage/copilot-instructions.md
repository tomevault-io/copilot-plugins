## cardsage

> **FeeWorth** is an annual fee renewal decision engine for individuals and couples, built as a Progressive Web App (PWA).

# CLAUDE.md — FeeWorth

## Project Overview

**FeeWorth** is an annual fee renewal decision engine for individuals and couples, built as a Progressive Web App (PWA).

- **URL**: https://cardsage.co (domain pending migration to feeworth.co)
- **Tagline**: Is the fee worth it?
- **Audience**: Credit card holders wondering whether to keep, cancel, or downgrade cards at renewal time. Individuals and couples.
- **Contact**: cardsage.co@gmail.com
- **Revenue model**: Affiliate commissions via Apply Now links (CJ Affiliate / FlexOffers). FTC disclosure required — see Affiliate Links section below.

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| UI framework | React 18 (via CDN, `react.development.js`) |
| JSX transpilation | Babel Standalone (via CDN, `@babel/standalone`) |
| Build system | **None** — single-file HTML app, no npm, no bundler |
| Fonts | Google Fonts: Playfair Display (serif display / wordmark), Inter (body), Source Code Pro (mono) |
| Analytics | Plausible (`data-domain="YOUR_DOMAIN"` — replace with `cardsage.co`) |
| Hosting | Netlify (auto-deploy from GitHub) |
| PWA | manifest.json + sw.js service worker |

**Key constraint**: Everything runs in the browser. No server, no backend, no database. All state lives in `localStorage`.

---

## File Structure

```
FeeWorth/
├── index.html             # Slim HTML shell (~60 lines) — loads all other files
├── config.js              # Central configuration (single source of truth)
├── styles.css             # All CSS styles
├── firebase-auth.js       # Firebase initialization (module script)
├── cards-data.js          # All data: CARDS, STRATS, TIPS_DB, APPLY_URLS, etc.
├── components.js          # All React components (loaded via Babel Standalone)
├── sw-register.js         # Service worker registration + version check
├── sw.js                  # Service worker (network-first for code, cache-first for assets)
├── version.json           # Deployment version (must match CS_CONFIG.CACHE_VERSION)
├── manifest.json          # PWA manifest
├── icon-192.png           # PWA icon (192×192)
├── icon-512.png           # PWA icon (512×512)
├── privacy-policy.html    # Privacy policy page
├── terms.html             # Terms of service page
├── affiliate-disclosure.html  # FTC affiliate disclosure page
└── .gitignore
```

**Load order in index.html**:
1. `config.js` — central config, injects CSS custom properties into `:root`
2. `styles.css` — all styles (references CSS vars from config.js)
3. `firebase-auth.js` — module script, loads Firebase SDK async, dispatches `cs-firebase-ready`
4. React 18 + ReactDOM 18 + Babel Standalone (CDN)
5. `cards-data.js` — all card/tip/strategy data as globals
6. `components.js` — all React components (`<script type="text/babel">`, transpiled by Babel on DOMContentLoaded)
7. `sw-register.js` — registers service worker + checks version.json for updates

---

## Key Data Structures (cards-data.js)

### `CARDS` — array of ~100 card objects
```js
{
  id: "csr",                      // unique kebab-case ID
  name: "Chase Sapphire Reserve", // full name
  short: "Sapphire Reserve",      // short display name
  issuer: "Chase",
  isBiz: false,
  fee: 550,                       // annual fee in dollars
  network: "Visa",
  cur: "Chase Ultimate Rewards",  // points currency
  c1: "#1a1a2e", c2: "#4a3728",  // gradient colors for card art
  partners: ["Hyatt", "United"],  // transfer partners
  annual: [{n, v, d, cat}],       // annual benefits {name, value, desc, category}
  monthly: [{n, v, d, cat}],      // monthly benefits
  strat: ["chase-trifecta"],      // strategy IDs this card belongs to
  signup: "60,000 pts after $4k in 3 mo",
  earn: {d, g, gas, t, s, a, tr, p, o} // earn rates by category key
}
```

**BENEFITS DATA RULE**: Every recurring benefit must be stored as ONE entry with the correct `reset` field (`monthly` / `quarterly` / `semi-annual` / `annual`). NEVER split a recurring benefit into multiple entries (e.g. "1st half" and "2nd half"). The UI renders multiple checkboxes from a single entry based on the `reset` field. Duplicating entries causes the benefit to appear multiple times in the list.

**SKIP PERSISTENCE RULE**: Skipped benefit state (`cs_skipped`) does NOT reset with quarterly/annual resets — it persists until the user manually un-skips. Skipped benefits are excluded from progress bar calculations (both numerator and denominator) but remain visible in the list under a "SKIPPED" divider at the bottom of each card.

### `STRATS` — object keyed by strategy ID
```js
{
  "chase-trifecta": {
    id, name, emoji, req: ["csr","csp","cfu"],  // required card IDs
    alt: [["csr","cff","cfu"]],                 // alternative card combos
    req_names, desc, forBeginners, analogy,
    firstStep, value, play: [...], learn
  }
}
```
**6 strategies**: `chase-trifecta`, `amex-trifecta`, `c1-duo`, `citi-duo`, `ink-trio`, `atmos-strategy`

### `TIPS_DB` — array of 25 tip objects
```js
{
  id: "t1",
  cat: "sweetspot",     // sweetspot | routing | stacking | timing | arbitrage | application
  title: "...",
  cards: ["csr","hyatt"],  // card IDs relevant to this tip
  body: "...",
  beginnerTip: "...",   // plain-English explanation for beginners
  startHere: true,      // (optional) marks tip as a "Start Here" foundational tip
  value: 3,             // 1–3 rating
  difficulty: "beginner" // beginner | intermediate | advanced
}
```

### `BCAT` — benefit category metadata
Keys: `travel`, `dining`, `entertainment`, `status`, `statement`, `awards`, `protection`
Each: `{label, icon, color, bg}`

### `BASIC_CATS` — standard spending categories
9 categories: `d` (dining), `g` (groceries), `gas`, `t` (travel), `s` (streaming), `a` (Amazon), `tr` (rideshare), `p` (pharmacy), `o` (everything else)

### `SPECIAL_CATS` — brand-specific categories
11 categories: Hyatt, Delta, Southwest, United, Hilton, Marriott, Alaska/Atmos, American Airlines, Amazon, Rent, IHG

### `EARN_PRIORITY` — object mapping category key → ordered array of card IDs (best earner first)
```js
{ d: ["amex-gold", "csr", ...], g: ["amex-gold", "amex-bcp", ...], ... }
```

### `ROTATING_Q1` — array of current quarter rotating category cards
```js
{ card, id, q, cats, rate, note, verified }
```

### `APPLY_URLS` — object mapping card ID → affiliate application URL
```js
{ "csr": "https://creditcards.chase.com/...", ... }
```
Cards without approved affiliate links use a `#apply-{cardId}` placeholder (falls back gracefully).

### `daysUntil(dateString)` — helper function
Returns integer days until a given date string. Used for benefit reset countdowns.

---

## React Components

| Component | Description |
|-----------|-------------|
| `App` | Root component — owns `myCards`, `checkedArr` state; renders TopNav + active tab |
| `TopNav` | Sticky frosted-glass nav with centered FeeWorth wordmark and tab bar |
| `HomeTab` | Dashboard with stats grid, strategy cards, rotating categories, email capture |
| `BenefitsTab` | Filterable benefit tracker — check off redeemed benefits, monthly/annual split |
| `TipsTab` | Tips browser with Beginner/Advanced mode toggle, Start Here section, category-grouped layout (Earning/Redeeming/Managing/Travel), and "Ready for you" badge per tip |
| `UsecardTab` | Category guide — which card to use for each spending category |
| `OffersTab` | Merchant offers browser — personalized to user's wallet with toggle |
| `QuizTab` | 5-question card finder quiz with animated transitions and localStorage persistence |
| `QuizResults` | Quiz output — Top 3 card recommendations + strategy suggestion + Apply Now |
| `WalletTab` | Card browser — add/remove cards from wallet, view card details |
| `StratModal` | Bottom sheet modal for strategy details (description, playbook, required cards) |
| `CardArt` | Visual credit card rendering with gradient from card's `c1`/`c2` colors |
| `ValueMeter` | 1–3 dot visual indicator for tip/strategy value rating |
| `EmailCapture` | Optional email signup component (Home + Benefits tabs); opens mailto on submit |
| `MerchantOfferCard` | Single merchant row in Offers tab — dims if user has no matching card |

---

## localStorage Keys

| Key | Type | Description |
|-----|------|-------------|
| `cs_cards` | `string[]` | Array of card IDs in user's wallet |
| `cs_checked` | `string[]` | Array of benefit check keys (`"{cardId}-{benefitName}"`) |
| `cs_email` | `string` | User's email address (optional, from EmailCapture) |
| `cs_quiz` | `object \| null` | Saved quiz answers object |
| `cs_tips_mode` | `string` | Tips tab mode preference (`'beginner'` or `'advanced'`) |
| `cs_benefit_check_dates` | `object` | Map of benefit key → ISO date string when it was last checked |
| `cs_skipped` | `string[]` | Array of benefit keys the user has skipped (excluded from progress) |

All keys are managed via the `useLS(key, defaultValue)` hook, which wraps `useState` + `localStorage`.

---

## Affiliate Links

**Network**: Currently using direct issuer URLs (CJ Affiliate / FlexOffers approval pending).
**Placeholder format**: Cards not yet in an affiliate program use `#apply-{cardId}` as the href, which renders as a non-functional anchor until replaced with a real URL.

**FTC disclosure is required in 3 locations** (already implemented):
1. Below every "Apply Now" button — `.apply-disclose` class: `"Affiliate link — we may earn a commission at no cost to you."`
2. Above Quiz results card list — inline notice
3. App footer — `"FeeWorth may earn a commission from card applications. This does not influence our recommendations."`

Full disclosure page: `affiliate-disclosure.html`

---

## PWA Setup

**manifest.json**
```json
{
  "name": "FeeWorth",
  "short_name": "FeeWorth",
  "start_url": "./index.html",
  "display": "standalone",
  "theme_color": "#03071d",
  "background_color": "#ffffff"
}
```

**sw.js** — reads `CACHE_VERSION` from `config.js` via `importScripts()`
- `LOCAL_ASSETS` pre-cached on install: index.html, config.js, styles.css, firebase-auth.js, cards-data.js, components.js, sw-register.js, version.json, manifest.json, icons, all legal pages
- CDN origins cached on first fetch: unpkg.com, fonts.googleapis.com, fonts.gstatic.com
- Network-first for all `.html`, `.css`, `.js` files (always fresh when online)
- Cache-first for everything else (icons, manifest, CDN assets)
- Old caches deleted on activate

**version.json** — `{"version":"v34"}` — must match `CS_CONFIG.CACHE_VERSION` in config.js

---

## Hosting & DNS

| Layer | Details |
|-------|---------|
| Source | GitHub: `https://github.com/skellycube-svg/cardsage` |
| Deploy | Netlify — auto-deploys on every push to `main` (~30 seconds) |
| Domain | `cardsage.co` |
| DNS A record | `75.2.60.5` |
| DNS CNAME (www) | `apex-loadbalancer.netlify.com` |

---

## Design System

**CSS variables** — defined in `config.js` → `CS_CONFIG.CSS_VARS`, injected into `:root` at load time:
- Background: `--bg: #ffffff`, `--s3: #f8f8f6` (subtle), `--s4: #f0f0ee`
- Accent/gold: `--acc: #b8860b`, `--gold: #b8860b`, `--gld2: #d4a840`, `--gld3: #fef9ec`
- Text: `--tx: #1a1a2e`, `--tx2: #6b7280`, `--tx3: #9ca3af`
- Issuer brand colors: `--chase: #112e51`, `--amex: #006fcf`, `--citi: #004c97`, `--capone: #003a70`
- Green: `--grn: #166534`, `--grn2: #16a34a` (success states)

**Fonts**:
- Display/brand: Playfair Display (italic, gold gradient shimmer on `.grad-text`)
- Body: Inter
- Financial values: Source Code Pro (`.mono`, `.stat-val`)

---

## Deployment Rules (follow these after every significant change)

After completing any significant change to FeeWorth, always do the following automatically without being asked:

1. **BUMP SERVICE WORKER**: If any of these changed — branding, colors, fonts, CSS, HTML structure, or new files added — increment the CACHE_VERSION number in sw.js
2. **BUMP VERSION.JSON**: Set the `"version"` field in version.json to match the new CACHE_VERSION in sw.js (e.g. `{"version":"v17"}`). These two must always stay in sync.
3. **UPDATE MANIFEST**: If the app name, theme color, or branding changed, update manifest.json to match.
4. **DEPLOY TO GITHUB**: `git add . && git commit -m "[description]" && git push`
5. **CONFIRM**: After pushing, tell me "Deployed to GitHub — Netlify will update in ~30 seconds"
6. **VERIFY BENEFITS**: When updating any card's data, always web search for "[card name] current benefits 2026" to verify all credits, perks, protections, and reset schedules are complete and accurate before committing. Never remove a benefit unless confirmed discontinued. Never add a benefit without a source.

---

## Rules

### Pre-Change Commit Rule
Before starting any major refactor or feature, commit the current working state first. This creates a restore point. Format: `"checkpoint: before [description of upcoming change]"`

### End-of-Session Rule
At the end of every session, before closing:
1. Update the Session History table with what was done
2. Update any other outdated sections of CLAUDE.md
3. Commit CLAUDE.md: `"docs: update CLAUDE.md with session [#] progress"`

### Context Warning Rule
If context usage reaches 70%, pause current work and:
1. Update CLAUDE.md with everything done so far
2. Commit it
3. Tell the user context is getting full and recommend starting a new session

---

## Session History

| # | Date | What Was Built / Changed |
|---|------|------------------------|
| 1 | Pre-March 2026 | Initial FeeWorth build: React SPA, cards data, tips, Firebase auth, PWA setup |
| 2 | March 2026 | Stitch design overhaul: Playfair Display fonts, gold color scheme, decorative credit card, SVG nav icons |
| 3 | March 2026 | Newsletter signup, Bilt 2.0 cards, tips restructure (flights/hotels/stacking/other), emoji elimination |
| 4 | March 21, 2026 | Modular refactor: config.js, split index.html into separate files, plain English comments |

---
> Source: [skellycube-svg/cardsage](https://github.com/skellycube-svg/cardsage) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
