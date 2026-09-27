## advit-global

> Next.js **pages router** (`pages/`), React, no CSS framework installed. Run `cat package.json` for the scripts; they are the source of truth for how to build, dev, and start.

# AGENTS.md — Advit Global Education

Next.js **pages router** (`pages/`), React, no CSS framework installed. Run `cat package.json` for the scripts; they are the source of truth for how to build, dev, and start.

## Brand brief

**Advit Global Education** teaches English test preparation — **IELTS, PTE, and the Duolingo English Test**. The name and mark point at study abroad, and abroad is why students come; the product we sell today is the class that gets them the band score. Write for that, not for placements.

The centre is at Shankhamul Tower, New Baneshwor, Kathmandu, and the market is Nepali students heading abroad. The visitor is a student, usually on a phone, usually also studying or working, comparing us against three other institutes within walking distance and a cousin's advice. They arrive skeptical — the sector is full of institutes that advertise guaranteed bands. And we are **new**, with no placement record to point at, which sets the whole strategy: we cannot win on volume claims, so we win on **transparency**. Say exactly what happens in the class, exactly who teaches it, and exactly what it costs, in more detail than a competitor is willing to. Specificity is the proof.

**Voice**: plain, specific, and quietly authoritative. Concrete over abstract — "unlimited mock tests, returned with feedback on your own answers" beats "quality coaching." Address the reader as *you*. Never use the vocabulary of a travel brochure ("dream destination", "your journey awaits"); never promise a band score, because no one can — describe the work that earns one.

Two hard limits on claims, both set by the client: **no batch-size figure** (no cap exists, so no number may be written) and **no trainer names or personal scores** ("trained and certified" is the ceiling). When a number is unavailable, describe the method rather than reaching for an adjective.

**What `logo_advit.png` commits us to.** Sampled from the file itself:

| Role | Values in the mark |
|---|---|
| Navy | `#001030` core → `#001A47` → `#002860` → `#003068` |
| Gold | `#A06000` deep → `#C88810` → `#D89818` → `#E8A828` → `#F8D848` highlight |

The gold is never flat — in the mark it is always a gradient running dark-bronze to bright, which is what makes it read as metal rather than yellow. The geometry is **angular and ascending**: a sharp-apex "A" cut through by a rising swoosh, three stars climbing off the peak, a meridian globe, an open book. The wordmark is an ultra-bold, wide, geometric uppercase sans; the tagline **"Rise Brilliant. Go Global."** sits below in a softer rounded sans.

Two commitments follow. **Navy is the ground and gold is the accent** — gold is the achievement, so spending it everywhere spends its meaning; it belongs on the one action we want and on moments of genuine proof. And the mark's diagonal is our one permitted flourish: **rising** motion, left-to-right and upward.

In one sentence: the mark should feel like an award you were handed, not an ad you were shown.

## The pipeline

Three **gates**. A gate is a full stop — present the work, then wait. The user's reply is the only thing that opens the next one; self-assessment opens nothing.

**Gate 1 — Content.** Write `design/CONTENT.md`: the page's job, its visitor, the single action it asks for, then every section in reading order with finished copy — headline, body, labels, button text, empty states. Ask the user for any fact you do not have rather than inventing it.
*Open when*: every section of the intended page has final copy in `design/CONTENT.md` and the user has said yes to it. Any placeholder text holds this gate shut.

**Gate 2 — System, then variations.** First search the repo for existing tokens; if they exist, document them in `design/SYSTEM.md` and build inside them. If they do not, present 4–5 whole directions — type pairing, colour roles, spacing rhythm, radius, elevation, mood — each defensible from the mark or the content, rendered side by side as a real page. Write the winner into `design/SYSTEM.md` as tokens. Then give every component in `design/CONTENT.md` 2–3 variations that differ in **structure** — what leads, what the eye hits first — drawing every value from `design/SYSTEM.md`, staged together on the review page in light and dark at 360px and 1440px.
*Open when*: `design/SYSTEM.md` holds the approved tokens and the user has named a winning variation for every component. One unpicked component holds this gate shut.

**Gate 3 — Integration.** Assemble the page from the picked variations only, pacing the sections and varying their density so the vertical rhythm lands on the spacing scale.
*Open when*: every item in *Definition of done* below is checked and reported.

## Design system

`design/SYSTEM.md` is the single source of truth for every token. Read it before writing any colour, space, radius, type size, or shadow — including the first one in a new file.

## Component conventions

- Components live in `components/`, one per file, `PascalCase.jsx`, default-exported, named for the content they carry (`AdmissionsTimeline`) rather than the shape they take (`ThreeColumnGrid`).
- Variations are staged in `components/variations/` as `<Component><Letter>.jsx` — `HeroA.jsx`, `HeroB.jsx`, `HeroC.jsx` — and rendered together on `pages/review.js`, the review page.
- A variation **graduates** when the user names it the winner: move it to `components/<Component>.jsx`, drop the letter, delete its siblings, and remove the set from the review page. A component the page imports never lives under `components/variations/`.
- Section copy comes from `design/CONTENT.md` verbatim. When copy needs to change, edit `design/CONTENT.md` first and take it back through the user.

## Definition of done for a page

Check each and report it:

- The page renders the approved copy from `design/CONTENT.md`, unchanged.
- Every colour, space, radius, type size, and shadow resolves to a token in `design/SYSTEM.md`.
- The page holds at 360px, 768px, and 1440px, and reads correctly in light and dark.
- Interactive elements have visible focus, hit targets clear 44px, and text contrast meets WCAG AA.
- The production build succeeds.
- You have run the page and looked at it — screenshot or live — before calling it done.

---
> Source: [E-n-N-D/advit-global](https://github.com/E-n-N-D/advit-global) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-24 -->
