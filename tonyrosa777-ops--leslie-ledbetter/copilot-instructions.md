## leslie-ledbetter

> Before writing ANY code — before touching a single file — enter Plan Mode.

# CLAUDE.md — Collaborative Insights Project Rules
#
# VARIABLES TO FILL (Task 1 of /prime will complete these):
#   Collaborative Insights     → e.g. "The Enchanted Collective"
#   CollaborativeInsights.com            → e.g. "enchantedmadison.com"
#   spiritual consulting and ascension guidance     → e.g. "luxury glamping and romantic experience property"
#   Texas (online delivery via Zoom)          → e.g. "Madison, Indiana"
#   Demo review: April 10, 2026 — launch date pending sale     → e.g. "June 2026"
#   people experiencing spiritual awakenings who feel confused or overwhelmed by emerging abilities  → e.g. "romantic couples aged 27–45 within 90 min drive"
#   60-minute spiritual consult via Zoom ($100/session)        → e.g. "glamping tents, cottage, day-use hot tub escapes, proposal packages"
#   launch a live, Google-indexed website that generates bookings for spiritual consult sessions          → e.g. "drive direct bookings and own the romantic getaway category in southern Indiana"
#   Calendly (embedded widget, max 8 hrs/day)    → e.g. "Lodgify (launch) / OwnerRez (scale)"
#   ProfessionalService       → e.g. "LodgingBusiness" or "LocalBusiness" or "ProfessionalService"

## Plan Mode Rule
Before writing ANY code — before touching a single file — enter Plan Mode.
Use EnterPlanMode and present a full build plan: what will be built, what files will
be created or modified, what design tokens will be used, what data will flow where.
Get alignment on the plan before the first keystroke. This is non-negotiable.

A wrong plan costs 5 minutes. A wrong build costs 5 hours.

## Subagent Delegation Rule
Any build phase with 3 or more discrete tasks MUST be broken into individual tasks
and delegated to subagents. One subagent per task. Run independent tasks in parallel.

Never execute a multi-task phase as a single monolithic session. This produces slower,
lower-quality output and exhausts context. The correct pattern:
1. Write a task list for the phase
2. Identify which tasks are independent (can run in parallel)
3. Spawn subagents for each task with complete context (file paths, design tokens, exact spec)
4. Collect outputs, verify, integrate

If a task is trivial (under 5 minutes, 1 file), do it inline. If it's substantive, delegate.

## Skill Creation Rule
When you solve a problem in a new way that works — a component pattern, an integration
flow, an animation approach, a build sequence — immediately:
1. Document the exact steps you took (not a summary — the actual implementation)
2. Create a skill file for it using /skill-creator
3. Reference the skill in future builds instead of reinventing

The goal: every non-trivial build decision becomes a reusable skill. Build once, reuse forever.
If a pattern is already in build-log.md, also create a skill so it can be invoked directly.

## Core Law: Research-Backed Decisions Only
Every design decision, copy choice, UX pattern, or technical recommendation
MUST be traceable to market-intelligence.md or initial-business-data.md.
If you cannot cite the research that backs a decision, do not make the
decision — surface it for review.

## Mandatory Pre-Read Protocol
At the start of EVERY session, read in order:
1. CLAUDE.md (this file)
2. progress.md
3. C:\Projects\Optimus Assets\knowledge\build-log.md  ← Cross-project errors + patterns. Check before starting any phase.
4. initial-business-data.md
5. market-intelligence.md
6. design-system.md
7. frontend-design.md
8. website-build-template.md

Never skip this sequence. Never rely on context from a previous session.
Treat each session as if it is your first.

EXCEPTION FOR SUBAGENTS: This protocol applies to the main orchestrator session only.
Subagents spawned via the Agent tool must NOT follow the full 8-file pre-read sequence —
they load only the files listed in their agent file's Required Reading section.
Loading all 8 files in every subagent wastes context before any work begins.

## Agent System Rules
These rules apply whenever the Subagent Delegation Rule triggers agent spawning.

**Agents never spawn agents.** Only orchestrators (workflow commands) spawn agents.
If a subagent needs help, it reports back to the orchestrator — it does not spawn
its own subagents. One level of hierarchy only. This is non-negotiable.

**Agents read files, not summaries.** Every agent gets context by reading known output
files directly (market-intelligence.md, design-system.md, /data/site.ts). The orchestrator
does NOT pass summaries or briefings — it passes file paths. The agent reads the file.

**Agents own their output files exclusively.** No two parallel agents may write to the
same file. Each agent owns exactly one output file or directory. If two tasks share an
output file, they run sequentially — not in parallel.

**Agents checkpoint progress.** After completing each discrete unit of work, the agent
writes a progress note to progress.md. If an agent fails mid-task, the orchestrator
can re-invoke it with "continue from [last checkpoint]" rather than starting over.

**Agent status lifecycle:** Every agent file has a status field: DRAFT → TESTED → VALIDATED.
Only VALIDATED agents run without human review of the output. DRAFT agents always get
output reviewed before proceeding.

**Orchestrators validate outputs.** Before unblocking the next task, the orchestrator
checks that the agent's output file exists, is non-empty, and passes the agent's
Validation criteria. Failing agents get re-run with a correction note — not silently passed.

**Variable injection via CLAUDE.md.** Agents read the project's CLAUDE.md directly to
get filled variables (Collaborative Insights, CollaborativeInsights.com, etc. — already substituted by /prime).
Orchestrators do NOT perform string substitution on agent file contents.

## Skill File Name Aliases
Some design skills reference files by generic names that differ from this
project's actual filenames. Resolve them:

| Skill references this name | Read this project file instead          |
|----------------------------|-----------------------------------------|
| FRONTEND_GUIDELINES.md     | frontend-design.md                      |
| APP_FLOW.md                | progress.md (site architecture section) |
| PRD.md                     | progress.md (phase overview + task list) |
| LESSONS.md                 | C:\Projects\Optimus Assets\knowledge\build-log.md |
| TECH_STACK.md              | website-build-template.md (Stack section) |
| progress.txt               | progress.md                             |

Never create duplicate files to satisfy a skill's expected filename.
Always resolve to the correct project file using this table.

## Frontend Design Rule
Before making ANY UI/UX decision, visual design change, component creation,
color selection, typography choice, layout decision, or CSS modification,
you MUST re-read frontend-design.md in full. No exceptions.
Reference the specific section of frontend-design.md that authorizes the
decision before implementing it.

## Build Template Rule
website-build-template.md is the build foundation — not the ceiling.
It defines the tech stack, directory structure, animation patterns, base
component architecture, and API route patterns that every Optimus project
starts from. Scaffold from the template first.

Then layer client-specific features on top, informed by initial-business-data.md
and market-intelligence.md. If a client need requires a component or pattern
not in the template, build it using the same stack, conventions, and patterns
the template establishes. Flag every custom addition in progress.md.

Do not ignore the template's patterns. Do not be constrained by its scope.

## Design System Rule
design-system.md is the brand constitution. It was synthesized directly
from market-intelligence.md and initial-business-data.md. You may not deviate
from the approved color palette, typography system, tone of voice, or brand
personality without explicit written approval and an update to design-system.md.
If a component requires a value not in the contract, flag it — do not improvise.

## Market Intelligence Rule
market-intelligence.md contains competitive research, audience psychology,
pricing benchmarks, and feature gap analysis. Every new feature, page, or
content block must be cross-referenced against this report.
Ask: "Does this serve the target audience? Is this validated by research?
Does this close a gap competitors have left open?"

## Progress Tracking Rule
After completing ANY subtask — not at the end of the session, AFTER EACH ONE —
immediately update progress.md with:
- What was completed
- What was discovered or decided
- What the next step is
- Any blockers or open questions

Do not batch updates. Do not defer to end of session. Context can exhaust mid-build
and a deferred update means that work is undocumented. Update after every subtask,
every time, without exception.

## Build Knowledge Rule
Before starting any phase, read the cross-project knowledge base:
`C:\Projects\Optimus Assets\knowledge\build-log.md`

This file contains every error solved and pattern discovered across all builds.
If a similar problem has been solved before, the solution is there.

When any error is resolved:
1. Add a row to the Error Encyclopedia table in `build-log.md` immediately
2. Create a detailed entry file in `C:\Projects\Optimus Assets\knowledge\errors\`
3. Do not continue work until the entry is written

When any phase completes with a non-obvious finding or pattern:
1. Add a row to the Build Patterns table in `build-log.md`

At project close:
1. Add a row to the Project Retrospectives table in `build-log.md`

## Content Standards
- Testimonials must read like a real human typed them on a phone. Never use the em dash (—).
  Humans use commas, periods, and ellipses. Em dashes are a copywriter/AI tell.
- All copy in `/data/site.ts` — zero hard-coded strings in components.
- Blog article CTAs close with an action, never a soft suggestion.
- Hungarian translations must be in third-person formal register — this is culturally
  mandatory for any client in a formal/governmental role (see Sylvia Rich retrospective).

## Act as Business Owner Rule
If initial-business-data.md lacks information needed for any section — the about story,
the founder background, a service detail, the company history — do NOT leave it blank,
do NOT write [MISSING:], and do NOT ask the business owner. Write it yourself in the
voice of the business owner: compelling, specific, plausible, and grounded in what you
do know about their industry and market.

Mark every invented section with a comment: // [DEMO COPY — pending client review]

The demo must look complete and impressive. A half-empty site loses the sale.
Corrections and personalizations happen after payment, not before the pitch.
This is a cardinal rule: we never hassle the business owner for details we can
reasonably write ourselves.

## Code Standards
- Next.js (App Router) + Tailwind CSS 4 + PostCSS — see website-build-template.md Stack section
- Animations: Framer Motion + react-intersection-observer — all scroll-triggered
- Design tokens defined as CSS custom properties in globals.css — not in tailwind.config
- TypeScript — strict mode on
- Mobile-first breakpoints: always design for 390px width before desktop
- Atomic git commits after every subtask — format: type(scope): description
- All copy lives in /data/site.ts — zero hard-coded strings in components
- Performance budget: Lighthouse score ≥ 90 on all pages
- Icons: use high-quality emoji, not SVG icon libraries. No Lucide, no Heroicons, no react-icons.
  Emoji renders natively, loads at zero cost, and looks clean at display size.
  Reference style: tonyrosa777-ops/placed-right-fence service page.

  Emoji is required in every one of these locations — never a plain text label alone:
  - Quiz answer options: every option has a leading emoji before the label text
  - Service cards: each service has an emoji (sourced from site.ts services[].emoji)
  - Pain point cards: each pain point has an emoji (site.ts painPoints[].emoji)
  - How It Works / Process steps: each step has an emoji (site.ts processSteps[].emoji)
  - Stats bar: each stat has a leading emoji (site.ts stats[].emoji)
  - About section belief/value bullets: each belief has an emoji
  - Pricing page feature lists: ✅ for included, ✗ for not included (never plain text)
  - Trust checklist bullets on service area pages: ✅ or context-specific emoji per bullet
  - FAQ: optional — emoji per question category group, not per question

  Choosing the right emoji: match the semantic meaning, not decoration.
  A plumbing service gets 🔧. A lawn care service gets 🌿. Speed stat gets ⚡.
  Wrong: generic ✨ on everything. Right: specific, meaningful, instantly readable.

## Hero Architecture Rule
Every hero section ships with exactly 3 layers. No exceptions. No photos. No static backgrounds.

**There is NO photo in the hero. Ever.** The client photo / brand image belongs in the About section,
not the hero. A photo placeholder in the hero is a build failure — flag it and replace it with the
3-layer animation stack before the phase is marked complete.

**Layer 1 — Canvas Particle System (HeroParticles.tsx)**
Choose particle type from the animation-specialist Selection Matrix based on brand axes.
Renders behind all content (z-0). Always present.

**Layer 2 — [BrandName]Canvas.tsx (Brand Canvas — brand-specific)**
A canvas-based animation that visually represents this specific business. NOT an SVG. NOT a generic
shape. A custom `<canvas>` component named after the brand (e.g. `HealthShieldCanvas.tsx`,
`ForgeCanvas.tsx`). Lives in the right panel of the two-column hero split.

Every brand canvas follows the same 5-phase lifecycle:
1. **STREAM** — N particles spawn at canvas edges and flow along quadratic bezier curves toward a
   center target. Each frame: `t += speed`. When all particles reach `t >= 0.94` → fire phase 2.
2. **RISE** — Particles cleared. Brand shape extrudes using `springOut(t)`:
   `1 - 2^(-9t) * cos(t * 10π * 0.68)` — gives physical spring overshoot. Duration: ~500ms.
3. **COOL** — Shape color animates through heat palette: white-hot → brand accent → brand primary.
   `heatRGB(t)` interpolates between stops. The shape literally "becomes" the brand as it cools.
4. **ARC** — Secondary element draws progressively (rail across fence pickets, arc around shield).
   Drawn via `ctx.arc(x, y, r, start, end * progress)`.
5. **IDLE** — `breathe = sin(elapsed * 0.00088)`. Oscillates coolingT + arc alpha — ambient pulse.

What changes per brand: the shape drawn in RISE (drawPicket, drawCross, drawFlame, etc.),
the heat palette endpoint (cools to brand primary), and the secondary element in ARC.
The 5-phase sequence and springOut function are identical on every build.

Canvas container: `position: relative`, explicit height `clamp(340px, 50vw, 540px)`.
Canvas fills container with `position: absolute; inset: 0`.
Always cast: `canvas.getContext("2d") as CanvasRenderingContext2D` — never leave nullable,
nested draw functions will fail TypeScript strict mode.

**Layer 3 — Framer Motion Stagger Text**
H1 first, subheadline at 0.15s delay, CTAs at 0.3s delay.
Renders above canvas (z-10). Always present.

**H1 = siteConfig.tagline always.** The tagline IS the H1 — it gets the shimmer class because
it is the brand identity statement. Emotional hook copy ("Stop paying twice your mortgage") goes
in the subheadline below the H1. Never put ad-hook copy in the H1.
Two shimmer classes — pick based on dominant brand token:
- `.hero-shimmer` — amber/gold sweep (for brands with gold/warm primary)
- `.hero-shimmer-sage` — sage/white sweep (for brands with cool/green/neutral primary)

**Tagline shimmer is mandatory.** The H1 (siteConfig.tagline) ALWAYS receives a shimmer class.
"Where healthcare finally makes sense." is the H1 with shimmer. Verify it renders in the browser
before phase sign-off. Wrong: any copy other than the brand tagline in the H1.

**Hero text must always be readable.** Hero headings and body copy always use `color: var(--text-primary)`
(which is #f5f5f5 on dark builds). If the background is dark and the text is dark, this is a build
failure. Do a visual check: can you read every word without highlighting? If not, fix the color token.

**Second CTA is always the quiz.** The hero's secondary button always links to `/quiz` with label
from `hero.ctaSecondary`. It is never a webinar, info session, events page, or external link.
The secondary CTA slot belongs to the quiz on every build, without exception.

This 3-layer stack is non-negotiable. The animation-specialist agent selects the specific
variants for layers 1 and 2. The text stagger is the same on every build.

## Always-Built Features Rule
Every project ships with ALL of the following, no exceptions, no client-by-client decisions:

**Pricing Page (sales tool — deleted before launch)**
Built in Phase 1. If the sweep completes without a /pricing page, that is a build failure.
In the nav bar throughout the entire build and demo process.
Deleted as part of the pre-launch audit — it is an Optimus sales tool, not a client deliverable.
The pre-launch-auditor agent flags /pricing still existing as a hard FAIL.

Nav display: the "Pricing" link renders in amber with a ⬥ marker (e.g. `⬥ Pricing`) so it is
instantly visually distinct from client-facing nav items. This signals to anyone viewing the demo
that it is an internal tool, not a page the client owns.

Fixed Optimus pricing structure — same on every build, never customized per client:
- Starter: $1,500 — core pages + canvas+SVG animated hero
- Pro: $3,000 — Starter + blog architecture, quiz lead capture, booking calendar (MOST POPULAR — this is the sell)
- Premium: $5,500 — Pro + shop (anchors Pro as reasonable, never gets a badge)

Pro gets the "Most Popular" badge. Starter and Premium are anchors.
Premium never gets a badge — its job is to make $3,000 feel reasonable.

The pricing page always contains:
1. Three tier cards (Starter / Pro / Premium) with feature lists and deposit breakdown (50% upfront)
2. ROI Calculator — two sliders (average job/project value + clients per month) + package selector
   → outputs: monthly revenue, break-even timeline, 12-month ROI per tier
3. Full comparison chart — feature rows grouped by category, checkmarks per tier
   Categories: Foundation / Conversion / Content & SEO / Commerce / Support
4. CTA on each tier that opens the booking calendar inline (never a link away)

**Interactive Quiz**
A scored lead funnel with typed output — not a contact form with extra steps. The quiz
computes a result archetype and delivers personalized results to the user via email. The client
gets a qualified lead notification. Both emails are sent via Resend in parallel.

**Architecture — two layers, fully decoupled:**

Data layer (`src/data/quiz.ts` — all quiz logic, zero UI dependency):
- `QuizType` — 4 result archetypes named for THIS brand's actual audience segments. Read `initial-business-data.md` and `market-intelligence.md` before writing these. Never copy archetypes from a prior build.
- `QUIZ_QUESTIONS` — 8 questions, each with exactly 4 answers, every answer tagged with a `QuizType`
- `QUIZ_RESULTS` — keyed by `QuizType`: name, tagline, body[] paragraphs, recommendedProgram { name, href, reason }
- `scoreQuiz(answers: QuizType[]): QuizType` — counts type occurrences, returns the highest; deterministic, pure, testable

UI layer (`src/app/quiz/QuizClient.tsx` — 3 phases via single `phase` state):
1. **intro** — hook headline + "Start the quiz" CTA
2. **question** — 8 questions rendered one at a time via `questionIndex` (0–7)
   - Each answer click → sets `pendingAnswer` for 400ms: selected answer glows brand primary, others dim to 30% opacity → auto-advances
   - On Q8 (last question): same 400ms timeout → `scoreQuiz(newAnswers)` → sets `resultType` → advances directly to results. No interstitial.
   - Back navigation → slices `answers` array to discard future answers, re-highlights the saved answer for that question
   - `direction` (1 or -1) → AnimatePresence x-offset: forward slides right-to-left, back slides left-to-right
3. **results** — renders `QUIZ_RESULTS[resultType]`: name, tagline, body paragraphs, recommended program card with link, then `<BookingCalendar />` inline directly below — never a link to /booking. The user typed themselves, saw their result, and the calendar is right there. One decision, one click.

There is no email gate phase. No name/email is collected by the quiz. Calendly's booking
form collects name and email as part of its own flow — nothing is lost. The friction wall
of an email gate at peak motivation is the worst possible place to ask for anything.

No `/api/quiz` email route. The client is notified of bookings through Calendly's own
booking confirmation notifications — not through a separate quiz API call.

Quiz CTA placement — two mandatory locations:
1. Site header: "Take the Quiz" button always visible in nav, always routes to /quiz
2. Homepage CTA block: full section that launches the quiz (links to /quiz page — not inline)
Never omit the header CTA. It is the highest-visibility quiz entry point.

Question count: 8 is the ceiling, not the floor. Use the minimum number of questions needed
to reliably assign an archetype. If 5 questions type the user cleanly, use 5. Do not pad
to 8 because Gray Method used 8. The auto-advance glow buys back some engagement time,
but drop-off still compounds with every extra question.

Reference implementation: tonyrosa777-ops/gray-method-training quiz.

**Inline Booking Calendar**
Custom-built calendar UI — a date picker that looks completely native to the site.
Uses the site's brand colors, typography, and design tokens. Not a Calendly iframe.
Under the hood, it calls the Calendly API to fetch available slots and submit bookings.

Architecture:
- `/api/calendly/slots` — GET, calls Calendly API for available times on a given date
- `/api/calendly/book` — POST, submits a booking via Calendly API
- `CALENDLY_API_KEY` — server-side env var (never NEXT_PUBLIC — key must stay server-only)
- `NEXT_PUBLIC_CALENDLY_EVENT_TYPE_URI` — the Calendly event type URI (public, safe to expose)
- Custom `<BookingCalendar />` component: month grid → date selection → time slot picker → confirm form

The component is 100% branded: brand-color selected states, brand font, brand button style.
A user looking at it should not be able to tell it uses Calendly under the hood.

Lives on a dedicated /booking page AND as a homepage teaser section.
NEVER implemented as an href link or Calendly iframe redirect.

Demo mode: if `CALENDLY_API_KEY` is not set, render seeded fake availability (deterministic
hash of date → available times) so the calendar is fully interactive during demo.
A blank or broken calendar kills the demo. A working calendar closes the sale.

**Testimonials Page**
Always built as a core page at /testimonials. Always ships with 36 testimonials.
Never conditional. Never "use what the client has." Write all 36.

Testimonial writing rules (non-negotiable):
- Written in the voice of the target audience from initial-business-data.md/design-system.md
- 36 total. Written by the content-writer agent from scratch, grounded in audience psychology.
- Any real testimonials the client provides are included and count toward the 36.
  Remaining slots are written to match the same voice and specificity.
- Paginated 9 per page on the /testimonials page (4 pages total = 4 × 9 = 36)
- Grid is always 3 columns × 3 rows — this makes every page a complete, consistent square.
  NEVER use 8 per page: 8 in a 3-col grid = 2 full rows + 2 orphans = broken layout on all 4 pages.
  9 per page is the only number that fills 3 columns perfectly. This is non-negotiable.
- ZERO em dashes (—) in any testimonial. Use commas, periods, ellipses only.
- Human tone: short sentences, specific details, sounds like a phone review, not a press quote.
- Vary by: service type, outcome, persona, and length (2 sentences to 4 sentences)

Homepage testimonials section: shows 3-4 featured quotes + "See All Testimonials" → /testimonials.
Page layout: featured quote full-width → paginated grid (9 per page, 3-col × 3-row) → filter by service → booking CTA.

**Blog**
9-10 articles minimum. SEO and AEO foundation. Always built. See Phase 7 in build-checklist.md.

**Shop**
Always scaffolded on every project. The scaffold is built whether or not the client bought Premium.
The decision gate runs AFTER scaffold — not before.

Reference implementation: C:\Projects\andrea-abella-marie\src\

Required files (scaffold on every build):
- src/lib/cart.tsx — CartProvider + useCart (localStorage-persisted cart state)
- src/components/CartDrawer.tsx — slide-in drawer, quantity controls, subtotal, checkout CTA
- src/lib/printful-seeded-products.json — 10-15 seeded products (name, price, category, preview image)
- src/lib/printful.ts — Printful API client (reads PRINTFUL_API_KEY from env)
- src/app/api/printful/products/route.ts
- src/app/api/printful/variants/[id]/route.ts
- src/app/api/stripe/checkout/route.ts
- src/app/api/stripe/webhook/route.ts
- src/components/ShopContent.tsx — product grid, category filter, variant picker, seeded fallback
- src/app/shop/page.tsx
CartProvider and CartDrawer wired in layout.tsx.

Seeded fallback rule (non-negotiable): ShopContent fetches /api/printful/products and falls back
to printful-seeded-products.json on any error. The shop must render a real-looking product grid
during demo with zero Printful credentials. An empty grid kills the demo.

Decision gate (after scaffold):
- Client bought Premium → wire PRINTFUL_API_KEY + STRIPE_SECRET_KEY + STRIPE_WEBHOOK_SECRET
- Client did not buy Premium → delete all shop files from the list above, remove from nav + sitemap

These are built in every Phase 1 agent sweep. They are never optional, never deferred,
never listed as "if applicable." If a phase sign-off doesn't include all of them: it is not done.

## Page Animation Rule
Every page ships with a brand-appropriate animation. The hero has the full 3-layer stack.
Every other page gets ambient effects only — never the full canvas+SVG assembly.

**The full 3-layer stack (HeroParticles + BrandCanvas + stagger text) is homepage hero only.**
Interior pages use lightweight ambient effects that match the brand's mood without the weight
of a full canvas animation. The hierarchy is:

- Homepage hero: full 3-layer stack (particles + brand canvas + stagger)
- Interior page headers: ambient only — rising ash particles, subtle twinkles, shimmer text, or breathing orbs

**Non-negotiable per-page minimums:**
- `/services` and individual service pages: rising ash particles or subtle twinkle canvas behind
  the page header (small canvas, low particle count). Never a static plain gradient.
- `/testimonials`: shimmer text effect on the featured quote header. Subtle twinkle or ash particles
  behind it. Booking CTA teaser at the bottom gets a breathing orb or gradient animation.
- `/blog` index: shimmer overlay or subtle animated gradient on the featured post hero header.
- `/about`: SlideIn animations for stats and photo. FadeUp on section headers.
- `/contact` and `/booking`: breathing orb behind the CTA header. Never flat.
- `/quiz`: slide left/right transition between each step.

**What "ambient effects" means in practice:**
- Rising ash: small canvas, ~20–30 particles that drift upward and fade. Not the 145-particle full system.
- Subtle twinkles: occasional 4-point glimmer flashes. Low density.
- Shimmer text: `.hero-shimmer` or `.hero-shimmer-sage` on the page H1. Always.
- Breathing orb: 1–2 radial gradient blobs, CSS-only, 12s cycle.

**Practical rule:** Before marking any page complete, scroll through it at full speed.
If it feels static or flat compared to the homepage, add ambient effects. The user should
feel the luxury quality on every page, not just the hero.

A website with one animated page and seven static pages is not a luxury product. It is a demo
that loses the sale the moment the client clicks away from the hero.

## Section Alternation Rule
The full homepage must alternate background tones so that no two adjacent sections
share the same background. Every transition shifts tone. This is non-negotiable.

**Before building any homepage section**, write the full section order with a dark/light
assignment as a comment block at the top of `app/page.tsx`. Plan the rhythm first,
then build. Fixing alternation after the fact costs 3–5 refactor commits.

Two background tones:
- **Dark:** `background: var(--primary)` — cards use `rgba(255,255,255,0.04)` bg,
  `rgba(255,255,255,0.08)` border. Text uses `var(--text-primary)`.
- **Light:** `background: var(--bg-base)` or `var(--bg-elevated)` — standard card styling.

Rules:
- Zero adjacent sections may share the same background tone. Not "avoid 3 in a row" — zero.
- Gallery/photo sections should almost always be light — dark backgrounds compete with images.
- The rhythm map must be committed in the same commit as the first homepage section.
- If a section is added or reordered later, update the rhythm map and verify no adjacency clash.

Reference pattern: `knowledge/patterns/homepage-dark-light-section-rhythm.md`

## Conversion Flow Rule
Never embed third-party redirects that take users off the CollaborativeInsights.com domain.
All conversion flows (booking, scheduling, purchase, inquiry) must be embedded
inline or iframed with seamless visual integration. Approved conversion tool:
Calendly (embedded widget, max 8 hrs/day). Every extra click costs conversions. Every domain redirect
costs trust.

## SEO Rule
Every page must include: semantic HTML5 structure, unique title tag, meta
description, Open Graph tags, ProfessionalService schema markup, crawlable text
(zero content locked in images or iframes), and proper heading hierarchy
(one H1 per page).

## Page Wiring Rule
Any new route or page created must be added to navigation and sitemap.ts in the
same commit. Never create a page without connecting it. New page = nav + sitemap
in the same commit, no exceptions.

## Placeholder CTA Rule
"Coming soon" or static CTA boxes are not acceptable phase sign-offs. Every primary
conversion flow must have a demo-mode interactive component before the phase is marked
complete — embedded calendar, mock booking widget, or form with success state.
Flag any static placeholder as a blocker and propose the interactive component before closing.

## Generated Assets Rule
Any script that outputs files into /public must commit those files as part of the
same task commit. Generated images, videos, and data files are never a separate
follow-up step. Generated assets are part of the task that created them.

## Communication Rule
Be opinionated. Flag tradeoffs. Cite research. When there is a clearly better
architectural choice, recommend it with justification. When something will break
at scale, say so. Do not pad responses. Do not assume obvious tasks are complete
without verifying.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tonyrosa777-ops) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
