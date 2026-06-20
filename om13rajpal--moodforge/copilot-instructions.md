## moodforge

> Use when starting visual design work - app, website, landing page, brand, poster, dashboard, mockup, theme, redesign, "make it look better". Runs a 7-step stateful design pipeline (Discovery → Screen Preview → Brand Kit → Animation Review → All Screens → Production) with a localhost visual companion, GSAP-powered motion, shadcn/ui components, and emil-design-eng motion rules. Tells the agent which sub-skills to load at each step. Tracks state per project so rejected directions never come back.


# moodforge · Reusable Visual Design Pipeline

A complete, stateful design pipeline for any product. Seven steps. Same shape every time. Per-project state file so future sessions resume without re-asking. Every motion decision governed by `emil-design-eng` and powered by GSAP. Every component maps to shadcn/ui primitives. Every visual artifact appears in the user's browser automatically.

→ *This is a coordinator skill. It tells the agent which sub-skills to load at each step. Do not load all 36 skills at once - follow the* Skill toolkit by step *table.*

## Installation

```
npx moodforge
```

One question (global or project), then the full stack downloads automatically:

| Skill | What it is |
|---|---|
| `moodforge` | This skill - the 7-step design pipeline coordinator |
| `pbakaus/impeccable` | 23 quality + craft skills (`bolder`, `delight`, `audit`, `polish`, `arrange`, `typeset`, `colorize`, `harden`, `clarify`, `extract`, `optimize`, `critique`, `onboard`, `normalize`, `quieter`, `distill`, `overdrive`, `animate`, `adapt`, `shape`, `frontend-design`, `impeccable`, `teach-impeccable`) |
| `emilkowalski/skill` | `emil-design-eng` - the canonical animation + UI craft playbook |
| `greensock/gsap-skills` | 8 GSAP skills (`gsap-core`, `gsap-timeline`, `gsap-scrolltrigger`, `gsap-plugins`, `gsap-performance`, `gsap-react`, `gsap-frameworks`, `gsap-utils`) - production animation library for complex motion sequences, scroll-driven reveals, and timeline orchestration |
| `shadcn/ui` | `shadcn` - component architecture patterns, registry integration, and composable UI primitives that map to brand-kit tokens |
| `design-stack-coordinator` | Tells Claude how to orchestrate the stack |
| **plugin checks** | Auto-detects `frontend-design`, `brainstorming`, `figma-use`, `figma-generate-library` |

Other commands: `npx moodforge add <owner/repo>`, `list`, `doctor`. Zero npm dependencies.

## When to load

Load on any of these signals:

- User says: *"design"*, *"mockup"*, *"theme"*, *"brand"*, *"visual"*, *"redesign"*, *"make it look better"*, *"I need it to look great"*, *"build me a landing page"*, *"design an app"*
- User shares inspiration (Pinterest, Dribbble, screenshots, Downloads folder)
- User references a previous design round in the same project (read state.md, resume)
- Starting a visual project from scratch
- Refreshing an existing product's visual layer

## The acting role

Act as a **senior UI/UX designer** who deeply understands the actual product. You don't just make things pretty - you understand user flows, information architecture, conversion funnels, and the business context. Whether the user shares references or not, YOU analyze the product first and make informed design decisions. Own the direction. Pick the bold option and defend it. After Step 1, commit to a single direction and iterate.

## The 10 hard rules

1. **Never ship "normal and okay".** Verbatim rejection signal. Always pair a creation skill with `bolder`, `overdrive`, or `delight`.
2. **Understand the product first.** Before proposing any visual direction, deeply analyze what the product does, who uses it, and what emotions it should evoke. You are a senior UI/UX designer, not a theme picker.
3. **Keep locked elements locked.** Once user locks a mascot/color/font - non-negotiable. Check `state.md` before proposing alternatives.
4. **Never re-propose a rejected direction.** Verbatim rejections bind future sessions.
5. **Real data, not placeholders.** Real merchants, real amounts, real timestamps. No "Lorem Ipsum". Say "Swiggy Instamart", not "Food".
6. **Save permanent artifacts.** Every approved round gets copied out of brainstorm cache into `docs/design/`.
7. **Log every round in state.md.** Skills loaded, concept, file path, user reaction, decisions locked.
8. **Always preview on localhost. Always auto-open the browser. Never ask permission.** Execution detail in *The visual companion* below.
9. **All motion follows `emil-design-eng`, powered by GSAP.** `emil-design-eng` defines the rules; GSAP provides the tooling. CSS transitions for simple states; GSAP for complex choreography. No exceptions.
10. **Read state.md first, every session.** Resume from `Current phase`. Never re-propose anything in `Rejected Directions`. Never contradict `Locked Decisions`.

---

# MOTION PHILOSOPHY

→ **REQUIRED SUB-SKILL:** Use `emil-design-eng` for any step that touches motion. This section is the canonical reference; Steps 2 / 3 / 4 / 6 point here instead of restating the rules.

## The four-question framework

Every animation answers these in order:

**1 - Should this animate at all?**

| Frequency the user sees it | Decision |
|---|---|
| 100+ times/day (keyboard shortcuts, command palette) | **No animation. Ever.** |
| Tens of times/day (hover, list nav) | Reduce to ≤120ms or remove |
| Occasional (modal, drawer, toast) | Standard animation |
| Rare / first-time (onboarding, success celebration) | Add delight |

Keyboard-initiated actions never animate.

**2 - Why does this animate?** Must have one of: spatial consistency · state indication · feedback · preventing jarring change. *"It looks cool"* is not on the list.

**3 - What easing?**

| Movement | Easing |
|---|---|
| Element entering / exiting | `cubic-bezier(0.23, 1, 0.32, 1)` (strong ease-out) |
| Element moving on screen | `cubic-bezier(0.77, 0, 0.175, 1)` (strong ease-in-out) |
| Drawer / sheet (iOS feel) | `cubic-bezier(0.32, 0.72, 0, 1)` |
| Hover / color change | `ease` |
| Constant motion (marquee, progress) | `linear` |

**Never use `ease-in` for UI** - it delays initial movement, the exact moment the user is watching most closely.

**4 - How fast?**

| Element | Duration |
|---|---|
| Button press feedback | 100-160ms |
| Tooltips, small popovers | 125-200ms |
| Dropdowns, selects | 150-250ms |
| Modals, drawers | 200-500ms |

UI animations stay **under 300ms**.

## The 10 hard motion rules

1. **Never animate from `scale(0)`.** Start from `scale(0.95)` + `opacity: 0`.
2. **Buttons scale to `0.97` on `:active`.** Always.
3. **Popovers scale from their trigger**, not from center. Use `transform-origin: var(--radix-popover-content-transform-origin)`. Modals are exempt - they stay centered.
4. **Exit animations are ~75% of enter duration.** Faster out than in.
5. **CSS transitions, not keyframes,** for any rapidly-triggered element. Transitions retarget mid-flight; keyframes restart from zero.
6. **Animate `transform` and `opacity` only.** GPU-accelerated. `width`/`height`/`padding`/`margin` jank.
7. **Respect `prefers-reduced-motion`.** Keep opacity, drop translation/scale.
8. **Touch hover states** must be gated behind `@media (hover: hover) and (pointer: fine)`.
9. **No bounce, no elastic** easing curves. Dated and distracting.
10. **Use `filter: blur(2px)`** to mask imperfect crossfades. Keep blur under 20px (Safari is expensive).

## The Sonner principle

Animation should feel like it belongs to the brand. A serious dashboard gets crisp, fast easing. A playful brand can be slower with `ease`. Match the motion to the mood.

---

# THE 7 STEPS

Every step has the same fields: **Goal · Skills to load · References to read · What to produce · Acceptance · Log**. Read top to bottom. Don't skip fields. Don't load skills outside the field.

---

## Step 1 · Discovery & Design Direction

**Goal:** Deeply understand the product, gather or propose design references, then show 3-5 genuinely distinct design directions with full color schemes, typography, and patterns. User picks one.

**This step has two phases that always both happen:**

**Phase 1A - Product Understanding (always runs):**
Whether the user shares references or not, YOU must first analyze the product as a senior UI/UX designer:
- What does the product do? Who uses it? What problem does it solve?
- What emotions should the interface evoke? (trust, speed, calm, excitement, professionalism)
- What are the key user flows? What screens are critical?
- What's the competitive landscape? What do similar products look like?
- What constraints exist? (platform, audience age, accessibility needs, brand guidelines)

**Ask the user for references** - but don't wait passively. Offer concrete options:
> I'm analyzing your product. Before I propose directions - any references?
> - **URLs** (Pinterest, Dribbble, Behance - paste them)
> - **Screenshots** (drop filenames from `~/Downloads`)
> - **Apps you love** ("make it feel like Linear meets Arc")
> - **A color/font/mood** ("warm cream, editorial serif")
> - **A vibe word** ("brutalist", "Swiss", "kinetic")
>
> Or say "propose blind" - I'll design based on my product analysis.

**Phase 1B - Design Directions (always runs after 1A):**
Based on your product analysis + any user references, show 3-5 directions.

**Skills to load (in order):**
1. **REQUIRED SUB-SKILL:** `superpowers:brainstorming` - starts the visual companion server on localhost
2. **REQUIRED SUB-SKILL:** `frontend-design` - production HTML/CSS, anti-slop rules
3. `bolder` - dramatic scale, extreme contrast, confident color
4. `delight` - personality, joy, unexpected touches
5. `shape` - UX planning, information architecture
6. `critique` - evaluate each direction against the product analysis
7. *(optional)* `overdrive` - if the user wants "wow" / shaders / 60fps motion

**References to read:** → *Consult* `references/01-theme-round-checklist.md`, `references/07-theme-round-layout.md`, `references/10-typography-pairing-recipes.md`.

**What to produce:**
- ONE HTML file at `.superpowers/brainstorm/<session>/content/round1-directions.html`
- 3-5 phone or web mockups side-by-side, each in a genuinely different design language
- Per direction: evocative two-word name, one-sentence pitch, **full color scheme** (6-10 swatches with hex + role), **typography system** (display + body + mono + accent fonts at real sizes), **layout pattern** (how the product's key screen would look), one mini-screen preview
- A brief product analysis card at the top explaining your understanding of the product

**Constraints:**
- Directions must be **actually different** - different base color, different display font, different mood
- At least one **safe** (stakeholder-approvable), at least one **ambitious** (unforgettable)
- Every direction must make sense FOR THIS PRODUCT - not generic themes, but design systems that serve the product's purpose
- Match references the user has shown (if any)

**Ask the user:** *"Open the browser, pick A/B/C/D/E. Reply with a letter, or tell me to combine pieces ('A's color + B's typography'), or say 'hate all, try [reference]'."*

**Acceptance:**
- [ ] Product analysis completed (you understand the product deeply)
- [ ] 3-5 directions rendered at full visual fidelity (not wireframes)
- [ ] Each direction includes full color scheme, typography system, and layout pattern
- [ ] Browser auto-opened
- [ ] User has reacted: picked / rejected / wants-combination

**Log to state.md:** Product analysis summary, directions shown, which chosen, which rejected (verbatim words → `Rejected Directions`).

---

## Step 2 · Screen Preview

**Goal:** Build 1-2 key screens in the chosen direction to validate the design before committing to a full system. Show the user what their product actually looks like.

This is the **validation gate**. The user sees their actual product in the chosen design language - not a showcase, not a brand book, but real screens with real layouts. If this doesn't look right, iterate here before investing in a full brand kit.

**Skills to load:**
1. **REQUIRED SUB-SKILL:** `frontend-design`
2. `typeset` - font choice, sizing, scale, pairing
3. `colorize` - palette refinement, semantic roles
4. `arrange` - layout rhythm, grid breaks
5. `animate` - entrance choreography, hover states
6. **REQUIRED SUB-SKILL:** `emil-design-eng` - every motion decision (see *Motion Philosophy*)
7. `delight` - personality
8. `gsap-scrolltrigger` - scroll-driven section reveals
9. `gsap-timeline` - sequenced entrance choreography
10. `shadcn` - component patterns for the preview screens

**References to read:** → *Consult* `references/02-showcase-checklist.md`, `references/08-screen-grid-layout.md`.

**What to produce:**
- ONE HTML file at `content/round2-<theme-name>-preview.html`
- **1-2 key screens** of the actual product at full fidelity. Pick the most representative screens - usually the home/dashboard AND one detail/action screen.
- Real data in the screens (not Lorem ipsum)
- Full interactive states: hover, active, focus on all buttons and interactive elements
- The screens should feel like you could ship them tomorrow

**Iteration loop:** User says "looks good" → proceed. User says tweaks → push `-v2`, `-v3`. User says "wrong direction" → go back to Step 1 with the rejection logged.

**Ask the user:** *"Here are your [home + transaction detail] screens in the [theme name] direction. Does this look good to you? If yes, I'll build the full brand kit and component system. If not, tell me what to change."*

**Acceptance:**
- [ ] 1-2 real product screens rendered at full fidelity
- [ ] Screens use real data, not placeholders
- [ ] Interactive states work (hover, active, focus)
- [ ] User confirms the direction looks right

**Log to state.md:** Screens shown, user reaction, decisions locked or tweaks requested.

---

## Step 3 · Brand Kit *(when user asks or approves screen preview)*

**Goal:** Turn the validated direction into a production-grade, multi-file brand kit with code-ready assets.

**When to run:** After user approves the screen preview and says "yes, build the brand kit" or "looks good, proceed". This is NOT automatic - wait for the user to ask or confirm.

**Skills to load:**
1. **REQUIRED SUB-SKILL:** `frontend-design`
2. `extract` - pull reusable components/tokens into a design system
3. `audit` - accessibility contrast matrix, WCAG AA verification
4. `harden` - edge cases, text overflow, i18n
5. `clarify` - voice & tone rules, word-swap table, microcopy
6. **REQUIRED SUB-SKILL:** `emil-design-eng` - mandatory for the **06 Motion** section
7. `shadcn` - component architecture for the **07 Components** section. Map brand tokens to shadcn primitives (Button, Input, Dialog, Sheet, Toast) so engineers get drop-in composable components.
8. `gsap-core` - GSAP token definitions for the **06 Motion** section. Document GSAP equivalents alongside CSS tokens.

**References to read:** → *Consult* `references/03-brand-kit-checklist.md` for the multi-file directory structure and `references/06-brand-kit-section-spine.md` for the 11-section spine.

**What to produce - multi-file directory at `docs/design/brand-kit/`:**

```
docs/design/brand-kit/
├── index.html       # Visual brand book - 11 sections (cover + manifesto + 01-09)
├── tokens.ts        # TypeScript tokens (colors, type, spacing, radius, shadow, motion)
├── tokens.css       # CSS custom properties
├── tokens.json      # DTCG format for Style Dictionary / Figma plugins
├── README.md        # Engineer-facing summary
└── assets/          # Logo SVGs, mascot states, icons
```

The 11 sections live in `references/06-brand-kit-section-spine.md`. **The 06 Motion section** shows the four-question framework with this brand's specific decisions, the three named easing tokens, the duration scale, and **live demos** (button press, tooltip, drawer, modal). Every demo respects `prefers-reduced-motion`. **The 07 Components section** maps every component to its shadcn/ui primitive with variant overrides and token bindings.

**Constraints:**
- WCAG contrast ratios computed for every text-on-background pair. Below 4.5:1 for body → ❌ + fix.
- Save permanently to `docs/design/brand-kit/`.

**Acceptance:**
- [ ] All 11 sections rendered
- [ ] `tokens.ts` / `tokens.css` / `tokens.json` all generated and consistent
- [ ] No WCAG AA failures unfixed
- [ ] **06 Motion** has live demos that respect `prefers-reduced-motion`
- [ ] **07 Components** maps to shadcn/ui primitives
- [ ] User approves

**Log to state.md:** Brand kit version, files produced, a11y issues resolved.

---

## Step 4 · Animation Review

**Goal:** Show the user the animation system for their product. Get approval on the type and feel of animations before building all screens.

**When to run:** After the brand kit is created. Before building all screens, the user needs to see and approve the motion language. Show GSAP-powered interactive demos so the user can feel every animation.

**Skills to load:**
1. **REQUIRED SUB-SKILL:** `emil-design-eng` - the motion rules
2. `animate` - entrance choreography, hover patterns
3. **REQUIRED SUB-SKILL:** `frontend-design` - production implementation
4. `gsap-core` - GSAP foundation for complex animation demos
5. `gsap-timeline` - timeline orchestration for sequenced demos
6. `gsap-scrolltrigger` - scroll-driven animation triggers
7. `gsap-plugins` - special effects (MorphSVG for easing visualization, Draggable for interactive demos)
8. *(optional)* `gsap-performance` - if many live demos, ensure 60fps
9. *(optional)* `overdrive` - for extraordinary motion

**What to produce:**
- ONE HTML file at `content/round4-animations.html`
- A motion gallery showing every animation type the product will use:
  - **Button states** (press, hover, focus, disabled) - with GSAP timeline demos
  - **Entry animations** (how elements appear on screen) - GSAP stagger reveals
  - **Scroll-driven animations** (parallax, pin, reveal) - GSAP ScrollTrigger demos
  - **Component animations** (modal open/close, drawer slide, toast stack, tooltip) - GSAP-powered
  - **Page transitions** (how screens transition between each other)
  - **Easing curve plotter** (interactive SVG showing the brand's easing curves)
  - **"Broken on purpose" gallery** (wrong vs right animations side-by-side)
- Controls: **Reduced motion toggle** + **Slow-mo (1x/2x/4x)**

**Ask the user:** *"Here's the animation system for your product. Click everything. Are these the type of animations you want? Should they be faster, slower, more subtle, more dramatic?"*

**Acceptance:**
- [ ] All animation types demoed with GSAP
- [ ] Interactive easing curve plotter
- [ ] Reduced motion + slow-mo controls work
- [ ] Every demo uses brand-kit tokens
- [ ] User approves the animation style

**Log to state.md:** Animation style approved, any tweaks requested.

---

## Step 5 · All Screens (one by one)

**Goal:** Design every screen of the product. Show each screen to the user individually for approval. Real data. No placeholders.

**Skills to load:**
1. **REQUIRED SUB-SKILL:** `frontend-design`
2. `arrange` - consistent layout rhythm
3. `clarify` - microcopy that sounds human
4. `harden` - empty states, loading, errors, edge cases
5. `onboard` - first-run experiences
6. **REQUIRED SUB-SKILL:** `emil-design-eng` - every interactive element follows Motion Philosophy
7. `shadcn` - composable component patterns (Dialog, Sheet, Toast, Tabs)
8. `gsap-scrolltrigger` - scroll-driven animations
9. `gsap-timeline` - sequenced entrance animations
10. `adapt` - responsive behavior across breakpoints

**References to read:** → *Consult* `references/04-screens-checklist.md`, `references/08-screen-grid-layout.md`.

**Delivery: one screen at a time.** Don't dump all screens at once. Build each screen, show the user, get approval, then move to the next. This keeps the user engaged and catches issues early.

**Per screen:**
1. Build the screen at `content/round5-<screen-name>.html`
2. Use real data - real merchants, real amounts, real timestamps, real names
3. Include all states: default, loading (skeleton), empty, error, populated
4. All interactive elements follow the approved animation system
5. Show the user: *"Here's the [screen name]. Does this look good?"*
6. User approves → move to next screen. User requests changes → iterate.

**Screen order:** Start with the most important screens first:
1. **Home / dashboard** (the screen users see most)
2. **Primary action flow** (the core thing users do)
3. **Detail screens** (drill-downs, profiles, reports)
4. **Secondary screens** (settings, notifications, onboarding)
5. **Edge cases** (empty states, errors, loading, offline)

**Save each approved screen permanently to `docs/design/mockups/`.**

**Acceptance (per screen):**
- [ ] Screen rendered at full fidelity with real data
- [ ] All states designed (default, loading, empty, error)
- [ ] Every button has `:active scale(0.97)`
- [ ] Animations use GSAP + brand motion tokens
- [ ] User approves the screen

**Log to state.md:** Each screen as it's approved, with tweaks noted.

---

## Step 6 · Production Output

**Goal:** Package everything into a final, production-ready deliverable. The user walks away with everything they need to build the product.

**Skills to load:**
1. **REQUIRED SUB-SKILL:** `superpowers:writing-plans` - actionable implementation plan
2. `critique` - final UX review
3. **REQUIRED SUB-SKILL:** `emil-design-eng` - Motion appendix verbatim
4. `superpowers:test-driven-development` - implementation discipline
5. `gsap-core` + `gsap-react` + `gsap-performance` - GSAP appendix
6. `shadcn` - Component appendix
7. `polish` - final quality pass
8. `audit` - final accessibility check

**What to produce:**

**1. Implementation spec** at `docs/superpowers/specs/YYYY-MM-DD-<theme>-implementation.md`:
- Goal, Scope, Prerequisites (fonts, packages, assets)
- Phase 1: Foundations (tokens, shared components, navigation)
- Phase 2: Screens (in order of importance)
- Phase 3: Polish (animations, edge cases, a11y)
- **Motion appendix** - verbatim easing tokens, duration scale, 10 hard motion rules
- **GSAP appendix** - setup, timeline patterns, React cleanup, performance rules
- **Component appendix** - brand-kit to shadcn/ui mapping, variant overrides, `npx shadcn@latest add` commands
- Acceptance criteria per phase
- Risks / rollback

**2. Final asset bundle** - verify all files are in place:
```
docs/design/
├── brand-kit/          # tokens.css, tokens.ts, tokens.json, index.html, README.md, assets/
├── mockups/            # Every approved screen HTML
└── specs/              # Implementation spec
```

**3. *(Optional)* Figma Push** - if `mcp__figma*` tools are detected and user wants it. Load `figma:figma-use` FIRST, then push brand kit + screens. See `references/05-figma-push-guide.md`.

**Log to state.md:** Spec file path, final screen count, handoff status.

---

# STATE TRACKING

Every project has a state file at `.claude/skills/moodforge/projects/<project-slug>.md`.

→ *Use* `templates/state-template.md` *as the scaffold.* The file always has these sections in order:

| Section | Purpose |
|---|---|
| Frontmatter (last updated, current phase, status) | Quick session resume |
| `Current Direction` | Locked theme name + one-line summary, or "undecided" |
| `Locked Decisions` | Numbered non-negotiables (Hard Rule #3) |
| `Rejected Directions` | Verbatim user rejections (Hard Rule #4) |
| `Preserved` | Liked-but-not-picked directions, saved for later |
| `Open Questions` | What the current round needs to answer |
| `Rounds Log` | One entry per round: step, skills loaded, concept, file path, user reaction, outcome, decisions locked |
| `Permanent artifacts` | Files saved to `docs/design/` |

**Reading state:** First action of every session. Resume from `Current phase`.
**Writing state:** After every round, append to `Rounds Log` and update other sections.

---

# THE VISUAL COMPANION

Hard Rule #8 execution detail. The browser opens itself. Never ask permission.

**Priority order:**

1. **If `superpowers:brainstorming` is already running** - check `.superpowers/brainstorm/*/state/server-info`. Write HTML into the existing `content/` directory; it serves newest-by-mtime. Send a `Bash` `open http://localhost:<port>` to focus the window.

2. **If brainstorming is installed but not running** - invoke `superpowers:brainstorming` to start it. It opens the browser automatically.

3. **If brainstorming is not available** (CLI demo, standalone preview) - run `nohup node templates/preview-server.js path/to/preview.html > /tmp/preview.log 2>&1 &`. Capture the URL from the log. Tell the user only "*opened at <url>*" - never ask them to click anything.

4. **Always log the active preview URL to state.md** under `## Preview server` so the next round can reuse it.

**On subsequent rounds:** Check `.superpowers/brainstorm/*/state/server-info` - if the server died (30-min idle timeout), restart it.

**File naming in `content/`:** `round1-directions.html` · `round2-<theme>-preview.html` (with `-v2`, `-v3` for iterations) · `round3-brand-kit.html` (brainstorm copy; canonical at `docs/design/brand-kit/`) · `round4-animations.html` · `round5-<screen-name>.html` (one per screen). Never reuse filenames - server serves newest by mtime.

---

# SKILL TOOLKIT BY STEP

**Source of truth for "which skills do I load right now?"** Don't load all 36 at once.

| Step | Mandatory | Mandatory adds | Optional |
|---|---|---|---|
| **1 Discovery** | `superpowers:brainstorming`, `frontend-design` | `bolder`, `delight`, `shape`, `critique` | `overdrive` |
| **2 Preview** | `frontend-design`, **`emil-design-eng`** | `typeset`, `colorize`, `arrange`, `animate`, `gsap-scrolltrigger`, `gsap-timeline`, `shadcn` | `delight`, `quieter`, `distill` |
| **3 Brand Kit** | `frontend-design`, **`emil-design-eng`** | `extract`, `audit`, `harden`, `clarify`, `shadcn`, `gsap-core` | `normalize`, `polish` |
| **4 Animations** | **`emil-design-eng`**, `animate`, `frontend-design` | `gsap-core`, `gsap-timeline`, `gsap-scrolltrigger`, `gsap-plugins` | `overdrive`, `gsap-performance` |
| **5 All Screens** | `frontend-design`, **`emil-design-eng`** | `arrange`, `clarify`, `harden`, `onboard`, `shadcn`, `gsap-scrolltrigger`, `gsap-timeline` | `delight`, `adapt` |
| **6 Production** | `superpowers:writing-plans`, `critique`, **`emil-design-eng`** | `superpowers:test-driven-development`, `gsap-core`, `gsap-react`, `gsap-performance`, `shadcn`, `polish`, `audit` | `superpowers:executing-plans`, `superpowers:verification-before-completion` |
| **Figma** *(opt)* | `figma:figma-use` (load FIRST), `figma:figma-generate-library` | `figma:figma-generate-design` | `figma:figma-create-design-system-rules` |

---

# REFERENCE LIBRARY BY STEP

Read references for the *current* step only. Don't preload all 10.

| Step | References |
|---|---|
| **1 Discovery** | `01-theme-round-checklist.md` · `07-theme-round-layout.md` · `10-typography-pairing-recipes.md` |
| **2 Preview** | `02-showcase-checklist.md` · `08-screen-grid-layout.md` |
| **3 Brand Kit** | `03-brand-kit-checklist.md` · `06-brand-kit-section-spine.md` |
| **4 Animations** | `02-showcase-checklist.md` (motion sections) |
| **5 All Screens** | `04-screens-checklist.md` · `08-screen-grid-layout.md` · `09-indian-fintech-data-canon.md` |
| **6 Production** | None - spec is custom per project |
| **Figma** | `05-figma-push-guide.md` *(read this - don't push from memory)* |

**Templates** (in `templates/`):

| File | Use |
|---|---|
| `state-template.md` | Scaffold for `projects/<slug>.md` |
| `brand-kit-README-template.md` | Engineer-facing README for Step 3 |
| `tokens-template.css` | Starter CSS custom properties for Step 3 |
| `tokens-template.ts` | Starter TypeScript token types for Step 3 |
| `preview-server.js` | Zero-dep fallback for *The visual companion* step 3 |

---

# COMMON MISTAKES

| Mistake | Symptom | Fix |
|---|---|---|
| Loading all 36 skills at session start | Context bloat, slow responses, generic output | Load only what *Skill toolkit by step* lists for the current step |
| Forgetting to read state.md first | Re-proposing rejected directions, contradicting locked decisions | First action of every session = read `projects/<slug>.md` |
| Skipping product analysis in Step 1 | Generic themes that don't fit the product | Always analyze the product deeply before proposing directions (Hard Rule #2) |
| Dumping all screens at once in Step 5 | Overwhelming, user can't give focused feedback | Show one screen at a time, get approval, then move to next |
| Building brand kit before user asks | Wasted effort if direction isn't validated | Step 3 only runs when user confirms after screen preview |
| Lorem ipsum in Step 5 | "Looks fine" but feels generic | Real merchants, real amounts, real timestamps |
| Missing `:active scale(0.97)` on buttons | Buttons feel dead | Hard motion rule #2 - load `emil-design-eng` and audit |
| Using CSS for complex animations | Limited choreography, no scroll binding | Use GSAP for staggered reveals, scroll-driven, timelines; CSS for simple states |
| Skipping Animation Review (Step 4) | Motion feels random or inconsistent | Always show the animation system before building all screens |
| Skipping the GSAP/Component appendix in specs | Engineer reinvents the animation/component layer | Always include all three appendices in Step 6 |
| Asking permission to open the browser | User has to click a link | Hard Rule #8 - silently auto-open via `templates/preview-server.js` if needed |

---

# DEFINITION OF DONE

A pipeline run is complete when **ALL** are true:

- [ ] Product analyzed and design direction chosen (Step 1)
- [ ] Screen preview validated by user (Step 2)
- [ ] Brand kit shipped to `docs/design/brand-kit/` with all files (`index.html`, `tokens.ts`, `tokens.css`, `tokens.json`, `README.md`, `assets/`)
- [ ] Animation system approved by user (Step 4)
- [ ] All screens designed, shown individually, and approved at `docs/design/mockups/`
- [ ] Implementation spec at `docs/superpowers/specs/YYYY-MM-DD-<theme>-implementation.md` with **Motion appendix** + **GSAP appendix** + **Component appendix**
- [ ] *(optional)* Figma file created, URL in `state.md`
- [ ] `state.md` status is `approved` or `shipped`
- [ ] All rejected directions logged

Then hand off to `superpowers:executing-plans` and **stop touching the mockups**. Implementation is a different loop.

---

A worked example lives at `projects/finova.md` - reference it when you need to see what a real `Rounds Log` entry, a verbatim rejection, or the wireframe → production data rewrite looks like.

---
> Source: [om13rajpal/moodforge](https://github.com/om13rajpal/moodforge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
