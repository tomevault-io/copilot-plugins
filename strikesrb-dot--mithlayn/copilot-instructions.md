## mithlayn

> You are building **Mithlayn** (mithlayn.com), a scholarly web application for studying the *mutashābihāt* (similar/repeated verses) of the Qurʾān, backed by classical Islamic scholarship.

# CLAUDE.md — Mithlayn Project System Prompt

You are building **Mithlayn** (mithlayn.com), a scholarly web application for studying the *mutashābihāt* (similar/repeated verses) of the Qurʾān, backed by classical Islamic scholarship.

Read this file completely before every session. No exceptions.

---

## Project Identity

**Name:** Mithlayn (مثلين) — Arabic for "two similars"  
**Mission:** Make Quranic mutashābihāt study accessible and beautiful. Scholarly depth, not superficial decoration.  
**Users:** Muslims serious about Qurān study — ḥuffāẓ (memorizers), students of tafsīr, Arabic learners.

---

## Design Mandate — The Anti-AI-Look Rule

Every frontend decision must pass this test: **Does this look like it was designed by a human who cares deeply about this subject, or does it look like AI output?**

### What to avoid (generic AI output):
- Hero sections with gradient blobs and floating cards
- Everything centered with too much padding
- Generic sans-serif at 16px with `#6b7280` subtext
- Rounded pill buttons with shadow hover effects everywhere
- Stock icon libraries used decorously
- Section headers like "Why Choose Mithlayn?" or "Features"
- Animations that serve no purpose beyond looking "dynamic"

### What to aim for (Apple/Linear/Vercel standard):
- One idea per screen section — ruthless whitespace
- Typography does the visual work — size contrast, weight, tracking
- Color used sparingly and with purpose — not as decoration
- Interaction that feels inevitable, not surprising
- Arabic text treated with the same care as English — it is the primary text
- The interface should feel like it was printed — precise, considered, not fluid

### Reference sites (spirit, not literal clone):
- apple.com — restraint, hierarchy, one thing at a time
- linear.app — dark product aesthetic, surgical grid
- vercel.com — monochrome confidence
- quran.com — Arabic text handling as a model (not the design)

---

## Always invoke the frontend-design skill first

Before writing **any** frontend code in a session:

```
invoke the frontend-design skill
```

No exceptions. This applies to: new pages, component additions, CSS changes, layout work.

---

## Screenshot workflow

After building or modifying any visual:
1. Start local server: `npx serve . -l 3000` (or `python3 -m http.server 3000`)
2. Take a full-page screenshot of the affected page
3. Compare against the design mandate above
4. Fix issues silently — do not ask permission for obvious improvements
5. For animated/dynamic elements, skip screenshot comparison and note this

Store screenshots in `temp-screenshots/`. Clear this folder at start of each session.

Do NOT take excessive screenshots. One full-page pass per major change is enough.

---

## File structure — never violate this

```
mithlayn/
├── CLAUDE.md               ← this file
├── index.html              ← hub portal only (gate + profiles + portal cards)
├── quran.html              ← Quran reader, pairs, Ḥifẓ, Browse, Stats, Priorities
├── arabic.html             ← Arabic learning (Bayna Yadayk + morphology)
├── commentary.html         ← Scholar source material (Kirmānī, Sakhāwī, Iskāfī)
├── practice.html           ← All drills and SRS (replaces revision.html)
├── memorize.html           ← Recitation rhythm / ḥifẓ pacing tool
├── citations.html          ← Source list
├── assets/
│   ├── css/
│   │   ├── tokens.css      ← CSS custom properties ONLY — do not add layout here
│   │   ├── shell.css       ← Nav, header, gate, shared components
│   │   ├── quran.css       ← Quran page only
│   │   ├── arabic.css      ← Arabic page only
│   │   ├── commentary.css  ← Commentary page only
│   │   ├── practice.css    ← Practice page only
│   │   └── memorize.css    ← Memorize page only
│   └── js/
│       ├── shared.js       ← Shared utilities (do not bloat)
│       ├── shell.js        ← Gate, profiles, nav, theme
│       ├── quran.js        ← Quran page logic
│       ├── arabic.js       ← Arabic page logic
│       ├── practice.js     ← Drills and SRS logic
│       └── memorize.js     ← Recitation rhythm logic
├── data/                   ← SACRED. Never modify. Never move. Never rename.
│   ├── quran.json
│   ├── conflicts/          ← Per-surah mutashābihāt pairs (c-1.json … c-114.json)
│   ├── kirmani/
│   ├── iskafi/
│   ├── sakhawi/
│   ├── external/           ← aswaatulqurraa.json
│   ├── arabic/             ← Per-surah morphology files
│   ├── morphology/
│   ├── grammar/
│   └── [other metadata]
└── sw.js                   ← Service worker — update cache string when files change
```

---

## CSS rules

- All values use `var()` from `tokens.css` — no hardcoded colors or sizes in component CSS
- Two themes: `html[data-theme="dark"]` (Cosmic Cliffs palette) and `html[data-theme="light"]` (Mushaf Cream)
- Theme is set on `<html>` element, persisted to `localStorage['mithlayn.theme']`
- Settings values from `localStorage['mithlayn.settings.v2']` override tokens at runtime
- Mobile-first breakpoints: base = mobile, `@media (min-width: 768px)` = tablet+
- Arabic text: always `font-family: var(--font-arabic)` (Amiri), `direction: rtl`
- Never inline styles except for truly one-off dynamic values

---

## JavaScript rules

- No frameworks — vanilla JS only
- Each page's JS is self-contained in its own file
- Data loading: fetch from `/data/` lazily, cache in module-level variables
- localStorage namespace: `mithlayn.*` (e.g. `mithlayn.theme`, `mithlayn.settings.v2`)
- Quran text: always NFC-normalize Arabic before any string comparison
- Never guess at verse text — always load from `data/quran.json`

---

## Qurʾān content — highest caution

- Arabic Qurʾān text is **always** rendered in Arabic script. Never transliterate.
- Canonical text source: Tanzil Uthmani (`data/quran.json`)
- NFC-normalize all Arabic before substring matching
- If uncertain about any verse, pair, or scholarly attribution — **stop and flag it**, do not guess
- Scholar names: al-Kirmānī (الكرماني), al-Sakhāwī (السخاوي), al-Iskāfī (الإسكافي), Aṣwāt al-Qurrāʾ (اصوات القراء)

---

## Deployment pipeline

1. Work and test locally on `localhost:3000`
2. When a change is approved: `git add . && git commit -m "description"` 
3. Push to GitHub → Vercel auto-deploys to mithlayn.com
4. Never push directly when changes are untested

**Never push to GitHub unless the user explicitly says "push" or "deploy".**

---

## What each page does

### index.html — Hub
Gate (passcode) → Profile select → Portal cards (6 sections). Lightweight. No data loading.

### quran.html — Quran Reader
- Browse view: Muṣḥaf-style verse display with mutashābihāt pairs highlighted
- Ḥifẓ view: Memorization tracking and session tools
- Priorities, Progress, Stats views: Study management
- Data: `data/quran.json`, `data/conflicts/c-{n}.json`, scholar JSONs for pair details

### arabic.html — Arabic Learning
- Word-by-word morphology reader (per-surah `data/arabic/ar-{n}.json`)
- Bayna Yadayk curriculum content (to be integrated)
- Grammar reference (`data/grammar/nahw.json`)
- Roots explorer (`data/arabic/roots.json`)

### commentary.html — Scholar Sources
- Full text display for Kirmānī, Sakhāwī, Iskāfī
- Entry-by-entry navigation with verse anchoring
- Existing file — redesign shell, keep logic

### practice.html — Drills
- Replaces `revision.html`
- Pair Match, Pair Recite, Missing Word, Scramble, Location
- Leitner SRS intervals
- Arabic vocabulary drills
- Data: `data/conflicts/`, morphology files

### memorize.html — Recitation Rhythm
- Rest-credit pool timing model
- Continuous session timer, auto-flip recite/rest
- Debt mode
- Storage key: `mithlayn.recitation.v3`

---

## When you finish a task

- Confirm what files were changed
- State what still needs to be done on this page
- If ready to test: give the localhost URL and what to look for
- Never auto-push to GitHub

---

*Last updated: scaffold build — May 2026*

---
> Source: [strikesrb-dot/mithlayn](https://github.com/strikesrb-dot/mithlayn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-18 -->
