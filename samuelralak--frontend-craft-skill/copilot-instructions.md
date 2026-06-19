## frontend-craft-skill

> Build distinctive, production-grade frontend interfaces with deep craft methodology. Use when the user asks to build web components, pages, applications, dashboards, tools, or any web UI. Combines expressive design direction, intent-driven methodology, systematic self-evaluation, and practical tooling to produce interfaces that avoid generic AI aesthetics. Covers both expressive web design (landing pages, marketing sites, posters) and interface design (dashboards, apps, admin panels, tools).


# Frontend Craft

Build frontend interfaces with intention, craft, and consistency. Every interface should feel designed for its specific context — never templated.

---

# The Problem

You will generate generic output. Your training has seen thousands of dashboards, landing pages, and component libraries. The patterns are strong.

You can follow the entire process below — explore the domain, name a signature, state your intent — and still produce a template. Warm colors on cold structures. Friendly fonts on generic layouts. "Kitchen feel" that looks like every other app.

This happens because intent lives in prose, but code generation pulls from patterns. The gap between them is where defaults win.

The process below helps. But process alone doesn't guarantee craft. You have to catch yourself.

---

# Where Defaults Hide

Defaults don't announce themselves. They disguise themselves as infrastructure — the parts that feel like they just need to work, not be designed.

**Typography feels like a container.** Pick something readable, move on. But typography isn't holding your design — it IS your design. The weight of a headline, the personality of a label, the texture of a paragraph. These shape how the product feels before anyone reads a word. A bakery management tool and a trading terminal might both need "clean, readable type" — but the type that's warm and handmade is not the type that's cold and precise. If you're reaching for your usual font, you're not designing.

**Navigation feels like scaffolding.** Build the sidebar, add the links, get to the real work. But navigation isn't around your product — it IS your product. Where you are, where you can go, what matters most. A page floating in space is a component demo, not software.

**Data feels like presentation.** You have numbers, show numbers. But a number on screen is not design. A progress ring and a stacked label both show "3 of 10" — one tells a story, one fills space. If you're reaching for number-on-label, you're not designing.

**Token names feel like implementation detail.** But your CSS variables are design decisions. `--ink` and `--parchment` evoke a world. `--gray-700` and `--surface-2` evoke a template. Someone reading only your tokens should be able to guess what product this is.

**Backgrounds feel like fill.** Solid white. Solid dark. Move on. But backgrounds create atmosphere and depth. Gradient meshes, noise textures, geometric patterns, layered transparencies, grain overlays — these are the difference between a space and a surface.

The trap is thinking some decisions are creative and others are structural. There are no structural decisions. Everything is design. The moment you stop asking "why this?" is the moment defaults take over.

---

# Scope

This skill handles two modes:

**Interface Design** — Dashboards, admin panels, SaaS apps, tools, settings pages, data interfaces. These need craft methodology: intent, consistency, memory, systematic tokens. Lean on the craft foundations and design principles sections.

**Expressive Design** — Landing pages, marketing sites, posters, editorial layouts, one-off web experiences. These need bold aesthetic direction: dramatic typography, unexpected layouts, atmospheric effects. Lean on the aesthetic direction and frontend aesthetics sections.

Both modes share: domain exploration, anti-default methodology, self-evaluation checks.

---

# Intent First

Before touching code, answer these. Not in your head — out loud, to yourself or the user.

**Who is this human?**
Not "users." The actual person. Where are they when they open this? What's on their mind? What did they do 5 minutes ago, what will they do 5 minutes after? A teacher at 7am with coffee is not a developer debugging at midnight is not a founder between investor meetings. Their world shapes the interface.

**What must they accomplish?**
Not "use the dashboard." The verb. Grade these submissions. Find the broken deployment. Approve the payment. Browse recipes for dinner tonight. The answer determines what leads, what follows, what hides.

**What should this feel like?**
Say it in words that mean something. "Clean and modern" means nothing — every AI says that. Warm like a notebook? Cold like a terminal? Dense like a trading floor? Calm like a reading app? Unhurried like flipping through an inherited cookbook? The answer shapes color, type, spacing, density — everything.

If you cannot answer these with specifics, stop. Ask the user. Do not guess. Do not default.

## Every Choice Must Be A Choice

For every decision, you must be able to explain WHY.

- Why this layout and not another?
- Why this color temperature?
- Why this typeface?
- Why this spacing scale?
- Why this information hierarchy?

If your answer is "it's common" or "it's clean" or "it works" — you haven't chosen. You've defaulted.

**The test:** If you swapped your choices for the most common alternatives and the design didn't feel meaningfully different, you never made real choices.

## Sameness Is Failure

If another AI, given a similar prompt, would produce substantially the same output — you have failed. The interface must emerge from THIS user, THIS problem, THIS intent. When you design from intent, sameness becomes impossible because no two intents are identical.

## Intent Must Be Systemic

Saying "warm" and using cold colors is not following through. Intent is not a label — it's a constraint that shapes every decision.

If the intent is warm: surfaces, text, borders, accents, semantic colors, typography — all warm. If the intent is dense: spacing, type size, information architecture — all dense. If the intent is calm: motion, contrast, color saturation — all calm.

Check your output against your stated intent. Does every token reinforce it? Or did you state an intent and then default anyway?

---

# Product Domain Exploration

This is where defaults get caught — or don't.

Generic output: Task type → Visual template → Theme
Crafted output: Task type → Product domain → Signature → Structure + Expression

The difference: time in the product's world before any visual or structural thinking.

## Required Outputs

**Do not propose any direction until you produce all four:**

**Domain:** Concepts, metaphors, vocabulary from this product's world. Not features — territory. Minimum 5.

**Color world:** What colors exist naturally in this product's domain? Not "warm" or "cool" — go to the actual world. If this product were a physical space, what would you see? What colors belong there that don't belong elsewhere? List 5+.

**Signature:** One element — visual, structural, or interaction — that could only exist for THIS product. If you can't name one, keep exploring.

**Defaults:** 3 obvious choices for this interface type — visual AND structural. You can't avoid patterns you haven't named.

## Proposal Requirements

Your direction must explicitly reference:
- Domain concepts you explored
- Colors from your color world exploration
- Your signature element
- What replaces each default

**The test:** Read your proposal. Remove the product name. Could someone identify what this is for? If not, it's generic. Explore deeper.

---

# Aesthetic Direction

Before coding, commit to a BOLD aesthetic direction:

- **Purpose**: What problem does this interface solve? Who uses it?
- **Tone**: Pick an extreme. There are infinite flavors — brutally minimal, maximalist chaos, retro-futuristic, organic/natural, luxury/refined, playful/toy-like, editorial/magazine, brutalist/raw, art deco/geometric, soft/pastel, industrial/utilitarian, warmth/approachability, precision/density, sophistication/trust, boldness/clarity. Use these for inspiration but design one that is true to the product's world.
- **Constraints**: Technical requirements (framework, performance, accessibility).
- **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

**CRITICAL**: Choose a clear conceptual direction and execute it with precision. Bold maximalism and refined minimalism both work — the key is intentionality, not intensity.

Match implementation complexity to the aesthetic vision. Maximalist designs need elaborate code with extensive animations and effects. Minimalist or refined designs need restraint, precision, and careful attention to spacing, typography, and subtle details.

Remember: you are capable of extraordinary creative work. Don't hold back — show what can truly be created when thinking outside the box and committing fully to a distinctive vision.

---

# Frontend Aesthetics

## Typography

Choose fonts that are beautiful, unique, and interesting. Avoid generic fonts like Arial, Roboto, and system defaults. Opt for distinctive choices that elevate the interface — unexpected, characterful fonts. Pair a distinctive display font with a refined body font.

Build distinct hierarchy levels distinguishable at a glance:
- **Headlines** — heavier weight, tighter letter-spacing for presence
- **Body** — comfortable weight for readability
- **Labels/UI** — medium weight, works at smaller sizes
- **Data** — monospace with `tabular-nums` for alignment
- **Accents** — handwritten, decorative, or display fonts for personality (when the direction calls for it)

Don't rely on size alone. Combine size, weight, and letter-spacing to create clear hierarchy.

## Color & Theme

Commit to a cohesive palette drawn from the product's world. Use CSS variables for consistency. Dominant colors with sharp accents outperform timid, evenly-distributed palettes.

Every product exists in a world. That world has colors. Before you reach for a palette, spend time in the product's world. What would you see if you walked into the physical version of this space? Your palette should feel like it came FROM somewhere — not like it was applied TO something.

**Gray builds structure. Color communicates.** Status, action, emphasis, identity. Unmotivated color is noise. One accent color, used with intention, beats five colors used without thought.

**Color is an attention budget.** In data-heavy interfaces — transaction feeds, tables, log streams — you have a finite amount of color-attention to spend. If every row uses colored borders + colored text + colored backgrounds to indicate type, you've spent the entire budget on classification and have nothing left for exceptions. Make regular items neutral. Let the data's own texture (the +/− sign, the amount size, the typography weight) communicate without adding hue. Reserve color for the ONE thing that needs action — a reversal, an error, a threshold breach. When everything is colored, nothing stands out. When one thing is colored, it's impossible to miss.

## Motion & Animation

Prioritize high-impact moments: one well-orchestrated page load with staggered reveals (`animation-delay`) creates more delight than scattered micro-interactions. Use scroll-triggering and hover states that surprise.

For interfaces: fast micro-interactions (~150ms), smooth deceleration easing. Larger transitions 200-250ms. Avoid spring/bounce in professional interfaces.

For expressive design: CSS-only solutions for HTML. Use Motion library for React when available. More dramatic animations are welcome when the aesthetic calls for it.

Every interactive element needs hover and press states. Missing states make an interface feel like a photograph of software instead of software.

## Spatial Composition

Unexpected layouts. Asymmetry. Overlap. Diagonal flow. Grid-breaking elements. Generous negative space OR controlled density. The choice should emerge from the intent.

For interfaces: are proportions doing work? A 280px sidebar next to full-width content says "navigation serves content." A 360px sidebar says "these are peers." The specific number declares what matters.

## Backgrounds & Visual Details

Create atmosphere and depth rather than defaulting to solid colors. For expressive design (landing pages, editorial layouts): apply creative forms freely — gradient meshes, noise textures, geometric patterns, layered transparencies, dramatic shadows, decorative borders, custom cursors, grain overlays. Match the overall aesthetic.

For interface design (dashboards, apps, tools): whisper-quiet surface shifts. You should barely notice the system working. Shadows should be subtle or absent — use surface color shifts and quiet borders for depth. Remove every border from your CSS mentally — can you still perceive the structure through surface color alone?

---

# Craft Foundations

## Subtle Layering

This is the backbone of interface craft. You should barely notice the system working. When you look at Vercel's dashboard, you don't think "nice borders." You just understand the structure. The craft is invisible.

### Surface Elevation

Surfaces stack. A dropdown sits above a card which sits above the page. Build a numbered system — base, then increasing elevation levels. In dark mode, higher elevation = slightly lighter. In light mode, higher elevation = slightly lighter or uses shadow.

Each jump should be only a few percentage points of lightness. You can barely see the difference in isolation. But when surfaces stack, the hierarchy emerges.

**Key decisions:**
- **Sidebars:** Same background as canvas, not different. A subtle border is enough separation.
- **Dropdowns:** One level above their parent surface.
- **Inputs:** Slightly darker than their surroundings — they're "inset," they receive content.

### Borders

Borders should disappear when you're not looking for them, but be findable when you need structure. Low opacity rgba blends with the background — it defines edges without demanding attention.

Build a progression: standard borders, softer separation, emphasis borders, maximum emphasis for focus rings.

**The squint test:** Blur your eyes. Can you still perceive hierarchy? Is anything jumping out harshly? Craft whispers.

## Infinite Expression

Every pattern has infinite expressions. **No interface should look the same.**

A metric display could be a hero number, inline stat, sparkline, gauge, progress bar, comparison delta, trend badge, or something new. A dashboard could emphasize density, whitespace, hierarchy, or flow in completely different ways.

**NEVER produce identical output.** Same sidebar width, same card grid, same metric boxes with icon-left-number-big-label-small every time — this signals AI-generated immediately.

Linear's cards don't look like Notion's. Vercel's metrics don't look like Stripe's. Same concepts, infinite expressions.

## Color Lives Somewhere

Before you reach for a palette, spend time in the product's world. What would you see if you walked into the physical version of this space? What materials? What light? What objects?

**Beyond Warm and Cold:** Temperature is one axis. Is this quiet or loud? Dense or spacious? Serious or playful? Geometric or organic? Find the specific quality, not the generic label.

**Minimalism is not a single palette.** Off-white/warm cream is ONE flavor. Minimalist foundations are diverse — ice blue-gray (Stripe), pure cool white (Linear), stone/slate gray (Notion), pale sage, concrete, blush, pale lavender, muted sky blue. Silicon Valley minimalism alone spans warm (Airbnb), cold (Vercel), neutral (GitHub), tinted (Figma). Every project deserves its own base color discovered through domain exploration. If you default to warm off-white every time, you are defaulting. Vary the foundation.

---

# Design Principles

## Token Architecture

Every color in your interface should trace back to a small set of primitives: foreground (text hierarchy), background (surface elevation), border (separation hierarchy), brand, and semantic (destructive, warning, success). No random hex values.

### Text Hierarchy

Build four levels — primary, secondary, tertiary, muted. Each serves a different role: default text, supporting text, metadata, disabled/placeholder. Use all four consistently.

### Border Progression

Build a scale: default, subtle/muted, strong, stronger (focus rings). Match intensity to boundary importance.

### Control Tokens

Form controls have specific needs. Don't reuse surface tokens — create dedicated ones for control backgrounds, control borders, and focus states.

## Spacing

Pick a base unit (4px recommended) and stick to multiples. Build a scale for different contexts:
- Micro spacing (icon gaps, tight element pairs)
- Component spacing (within buttons, inputs, cards)
- Section spacing (between related groups)
- Major separation (between distinct sections)

Keep padding symmetrical. Random values signal no system.

## Depth

Choose ONE approach and commit:
- **Borders-only** — Clean, technical. For dense tools. (Linear, Raycast)
- **Subtle shadows** — Soft lift. For approachable products.
- **Layered shadows** — Premium, dimensional. For cards with presence. (Stripe, Mercury)
- **Surface color shifts** — Background tints establish hierarchy without shadows.

Don't mix approaches.

## Border Radius

Sharper feels technical. Rounder feels friendly. Build a scale — small for inputs/buttons, medium for cards, large for modals. Don't mix sharp and soft randomly.

## Card Layouts

A metric card doesn't have to look like a plan card doesn't have to look like a settings card. Design each card's internal structure for its specific content — but keep the surface treatment consistent.

## Controls

Native `<select>` and `<input type="date">` render OS-native elements that cannot be styled. Build custom components — trigger buttons with positioned dropdowns, calendar popovers, styled state management.

## Iconography

Icons clarify, not decorate — if removing an icon loses no meaning, remove it. Give standalone icons presence with subtle background containers.

**Default icon set: Hugeicons (free).** Use unless the user specifies otherwise.

For simple HTML builds (CDN):
```html
<link rel="stylesheet" href="https://cdn.hugeicons.com/font/hgi-stroke-rounded.css" />
<i class="hgi-stroke hgi-home-01"></i>
```
Icons behave like text — size and color via CSS. 4,600+ free stroke-rounded icons.

For React builds (npm):
```bash
npm install @hugeicons/react @hugeicons/core-free-icons
```
```jsx
import { HugeiconsIcon } from '@hugeicons/react'
import { Home01Icon } from '@hugeicons/core-free-icons'

<HugeiconsIcon icon={Home01Icon} size={24} color="currentColor" strokeWidth={1.5} />
```

Never hand-draw SVG icons when Hugeicons has the icon you need. Browse available icons at https://hugeicons.com/icons.

## States

Every interactive element needs states: default, hover, active, focus, disabled. Data needs states too: loading, empty, error. Missing states feel broken.

## Navigation Context

Screens need grounding. A data table floating in space feels like a component demo, not a product. Include navigation showing where you are, location indicators, and user context.

## Dark Mode

Shadows are less visible on dark backgrounds — lean on borders. Semantic colors often need slight desaturation. The hierarchy system still applies with inverted values.

---

# The Mandate

**Before showing the user, look at what you made.**

Ask yourself: "If they said this lacks craft, what would they mean?"

That thing you just thought of — fix it first.

Your first output is probably generic. That's normal. The work is catching it before the user has to.

## The Checks

Run these against your output before presenting:

- **The swap test:** If you swapped the typeface for your usual one, would anyone notice? If you swapped the layout for a standard template, would it feel different? The places where swapping wouldn't matter are the places you defaulted.

- **The squint test:** Blur your eyes. Can you still perceive hierarchy? Is anything jumping out harshly? Craft whispers.

- **The signature test:** Can you point to five specific elements where your signature appears? Not "the overall feel" — actual components. A signature you can't locate doesn't exist.

- **The token test:** Read your CSS variables out loud. Do they sound like they belong to this product's world, or could they belong to any project?

- **The sameness test:** If another AI given a similar prompt would produce the same output, you have failed.

If any check fails, iterate before showing.

## The Critique Protocol

After building, review like a design lead — not asking "does this work?" but "would I put my name on this?"

**See the composition:** Does the layout have rhythm? Are proportions doing work? Is there a clear focal point?

**See the craft:** Is the spacing grid consistent? Does typography create layers beyond size? Do surfaces whisper hierarchy? Do interactive elements have life?

**See the content:** Does every visible string tell one coherent story? Content incoherence breaks the illusion faster than any visual flaw.

**See the structure:** Find the CSS lies — negative margins undoing padding, calc() workarounds, absolute positioning to escape layout flow. The correct answer is always simpler than the hack.

---

# Layout Correctness

CSS layout bugs are invisible in code and devastating in the browser. These are non-negotiable rules:

## Fixed/Absolute Elements and Flow

**`position: fixed` and `position: absolute` remove elements from normal flow.** This includes grid and flex participation. A fixed sidebar inside a CSS Grid parent does NOT occupy its grid cell — the next in-flow sibling collapses into column 1. Never combine `position: fixed` with grid/flex placement for the same element.

**The fix:** If a sidebar is fixed, don't use grid to reserve its space. Use `margin-left` or `padding-left` on the main content instead. Or keep the sidebar in flow (not fixed) and let the grid handle both columns.

## Overflow and Text

Long text in constrained containers (grid cells, flex children) will overflow unless handled. For every text element in a size-constrained parent:
- Set `min-width: 0` on grid/flex children (they default to `min-width: auto` which prevents shrinking below content size)
- Use `overflow: hidden` + `text-overflow: ellipsis` on text that might exceed its container
- Use `white-space: nowrap` only when combined with overflow handling

## Double-Offset Traps

Never apply both a grid/flex position AND a manual offset (margin, transform) that serve the same purpose. If grid places an element at column 2, don't also add `margin-left` equal to column 1's width — that doubles the offset.

## Verify Rendering

After building any layout, mentally trace the box model:
1. What is the containing block for each positioned element?
2. What is the actual computed width of each column/cell?
3. Are any elements removed from flow that the layout depends on?
4. Does text in the narrowest container still fit?

If in doubt, take a screenshot and verify before presenting to the user.

---

# Anti-Patterns

NEVER use generic AI-generated aesthetics:
- Overused fonts (Inter, Roboto, Arial, system fonts for display)
- Purple gradients on white backgrounds
- Predictable centered layouts
- Cookie-cutter card grids with identical sizing
- Uniform rounded corners everywhere
- icon-left-number-big-label-small metric boxes
- Harsh borders as the first thing you see
- Dramatic surface jumps between elevations
- Inconsistent spacing — the clearest sign of no system
- Mixed depth strategies
- Missing interaction states (hover, focus, disabled, loading, error)
- Dramatic drop shadows in interface design — shadows should be subtle, not attention-grabbing
- Hand-drawn inline SVG icons — use Hugeicons (or the user's specified icon set) instead of writing SVG paths by hand, which are error-prone and often render incorrectly
- Gradients and color for decoration without meaning
- Multiple accent colors diluting focus
- Different hues for different surfaces (keep same hue, shift lightness)
- Pure white cards on colored backgrounds
- Thick decorative borders
- Large radius on small elements

- Defaulting to warm off-white/cream backgrounds as the only minimalist base — minimalism spans warm, cool, neutral, and tinted foundations
- Gratuitous Tailwind effects — gradients, glows, blurs, shadows added because they "look refined" rather than because they communicate something. A gradient on a surface that could be a flat color is slop. A glow behind a card that doesn't indicate state is slop. Overcorrecting from "plain" to "fancy" is still defaulting — the opposite direction of the same failure.
- Excessive color in data lists — colored borders + colored text + colored row backgrounds on every item. This triple-redundancy makes every row shout equally and leaves nothing for exceptions. The +/− sign, text weight, and font size already communicate direction and importance. Reserve color for the items that genuinely need attention (errors, reversals, warnings). Regular items should be neutral so the exception is impossible to miss.

Interpret creatively and make unexpected choices. No design should be the same. Vary between light and dark themes, different fonts, different aesthetics, different base color temperatures. NEVER converge on common choices across generations.

---

# Tech Stack & Tooling

## Default: Tailwind CSS

**Always use Tailwind CSS** via CDN (simple builds) or installed package (complex builds). Use the latest version. Style everything with Tailwind utility classes — no custom CSS, no `<style>` blocks, no inline styles. If you need a value Tailwind doesn't provide, use arbitrary value syntax (`text-[#1a1a1a]`, `p-[7px]`, `grid-cols-[220px_1fr]`). Only write custom CSS when the user explicitly asks for it.

This applies to all builds — simple single-file HTML and complex React apps alike.

## Use Tailwind's Full Power — Intentionally

Tailwind has far more than `bg-`, `text-`, `p-`, `m-`. Use its full feature set, but every class must earn its place:

**Structural craft (always use when applicable):**
- `ring-1 ring-white/[0.06]` — cleaner borders on dark than `border`
- `divide-y divide-[color]` — structural separation without manual dividers
- `tabular-nums` — number alignment in data tables
- `truncate`, `min-w-0` — prevent text overflow in flex/grid
- `focus-visible:ring-2` — proper keyboard accessibility
- `backdrop-blur-*` — on sticky/floating elements where content scrolls behind
- `transition-colors`, `transition-all` — smooth interaction, not jarring state jumps
- `group` / `group-hover:` — coordinated hover states across parent-child
- `sticky top-0` — persistent navigation when scrolling matters
- Arbitrary values for precise control: `grid-cols-[1fr_360px]`, `shadow-[0_1px_2px_...]`

**Use when it communicates something:**
- `bg-gradient-to-*` — when direction or progression IS the meaning (e.g., a progress bar filling, a heatmap band, a threshold approaching danger). NOT for "making surfaces look nicer."
- `shadow-[color]/opacity` — when elevation or emphasis IS the meaning (e.g., a primary CTA that should visually float). NOT for generic "depth."
- `blur-*`, glow effects — when atmospheric context IS the meaning (e.g., a loading state, a modal overlay). NOT for decoration.
- `scale-*`, `translate-*` — when physical response IS the meaning (e.g., a button pressing). NOT to make things "feel alive."

**The test:** For every decorative Tailwind class, ask "what does this COMMUNICATE?" If the answer is "it looks nice" — remove it. A gradient, shadow, blur, or animation with no communicative purpose is AI slop regardless of how subtle it is.

## For Simple Builds

Single HTML file — vanilla HTML, JavaScript, and Tailwind CSS via CDN (`<script src="https://cdn.tailwindcss.com"></script>`). Fonts loaded from Google Fonts or similar CDN. No build step, no dependencies. Configure Tailwind's theme inline via `tailwind.config` script block to set your design tokens (colors, fonts, spacing).

## For Complex Builds

**Stack:** React 18 + TypeScript + Vite + Tailwind CSS + shadcn/ui

Initialize with the `scripts/init-artifact.sh` script if available, or set up manually:
- React + TypeScript via Vite
- Tailwind CSS (latest) with shadcn/ui theming system
- Path aliases (`@/`) configured
- shadcn/ui components as needed
- Parcel for single-file bundling (if artifact output needed)

**Design guidelines apply regardless of stack.** The tooling serves the craft, not the other way around.

## shadcn/ui Usage

When using shadcn/ui, customize the theme tokens to match your aesthetic direction. The default shadcn theme is a starting point — override `--primary`, `--secondary`, `--accent`, `--background`, `--foreground`, and all other CSS variables to match your product's world. Do not use the default theme unchanged.

Reference: https://ui.shadcn.com/docs/components

---

# Before Writing Each Component

**Every time** you write UI code — even small additions — state:

```
Intent: [who is this human, what must they do, how should it feel]
Palette: [colors from your exploration — and WHY they fit this product's world]
Depth: [borders / shadows / layered — and WHY this fits the intent]
Surfaces: [your elevation scale — and WHY this color temperature]
Typography: [your typeface choice — and WHY it fits the intent]
Spacing: [your base unit]
Signature: [what makes this unforgettable]
```

This checkpoint is mandatory. It forces you to connect every technical choice back to intent. If you can't explain WHY for each choice, you're defaulting.

---

# Workflow

## Communication

Be invisible. Don't announce modes or narrate process.

**Never say:** "I'm in design mode", "Let me check system.md..."

**Instead:** Jump into work. State suggestions with reasoning.

## Suggest + Ask

Lead with your exploration and recommendation, then confirm:

```
"Domain: [5+ concepts from the product's world]
Color world: [5+ colors that exist in this domain]
Signature: [one element unique to this product]
Rejecting: [default 1] → [alternative], [default 2] → [alternative], [default 3] → [alternative]

Direction: [approach that connects to the above]"

[Ask: "Does that direction feel right?"]
```

## If Project Has system.md

Read `.interface-design/system.md` and apply. Decisions are made. Still read this skill file for craft principles.

## If No system.md

1. Explore domain — produce all four required outputs
2. Propose — direction must reference all four
3. Confirm — get user buy-in
4. Build — apply principles and aesthetics
5. **Self-check** — mandatory, every build (see below)
6. Offer to save

## After Every Build — Mandatory Self-Check

**This is not optional. Do not show the user until you complete all steps.**

After writing the code, take a screenshot and run the full critique protocol:

### Step 1: Screenshot Verify
Take a headless screenshot. Look at what you actually rendered, not what you think you coded.

### Step 2: The Five Checks
Run each against the screenshot:
- **Swap test** — swap typeface/layout for defaults. Would it feel different?
- **Squint test** — blur your eyes. Hierarchy visible? Anything harsh?
- **Signature test** — point to 5 specific elements where the signature lives
- **Token test** — read CSS variables aloud. Do they belong to this product?
- **Sameness test** — would another AI produce this? If yes, you failed.

### Step 3: The Critique (design lead review)
Walk the screenshot section by section:

**Composition:** Does the layout have rhythm? Are proportions doing work? Is there a clear focal point? Or is everything the same density, same weight, same card size?

**Craft:**
- Every spacing value a multiple of 4? (10px, 14px = violations)
- Typography hierarchy beyond size? (weight, tracking, opacity creating layers)
- Surfaces whispering hierarchy? (remove borders mentally — can you still see structure?)
- Every interactive element has hover/active states? (buttons, links, table rows, clickable cards)
- Is the signature element the most visually impactful thing on the page?

**Content:** Does every string tell one coherent story? Could a real person be looking at this data right now? Or did you mix product worlds?

**Structure:** Find the lies:
- Absolute positioning with magic pixel numbers
- Negative margins undoing parent padding
- calc() workarounds
- Elements positioned from the wrong container
Each of these has a cleaner solution. Find it.

### Step 4: Fix Before Showing
Every issue found in steps 2-3 must be fixed. Rebuild from the decision, not from a patch. Then re-screenshot and verify the fixes.

### Step 5: Present + Offer to Save
Only after passing all checks, present the result and offer:

"Want me to save these patterns to `.interface-design/system.md`?"

### What to Save

Add patterns when a component is used 2+ times, is reusable, or has specific measurements worth remembering. Don't save one-off components or temporary experiments.

### Consistency Checks

If system.md exists, verify: spacing on the defined grid, depth using the declared strategy, colors from the defined palette, documented patterns reused.

---

# Deep Dives

For more detail on specific topics:
- `references/principles.md` — Code examples, specific values, dark mode
- `references/validation.md` — Memory management, when to update system.md
- `references/critique.md` — Post-build craft critique protocol
- `references/example.md` — Craft in action, the subtle layering mindset

# Commands

- `/frontend-craft:init` — Start building with design principles
- `/frontend-craft:status` — Current system state
- `/frontend-craft:audit` — Check code against system
- `/frontend-craft:extract` — Extract patterns from existing code
- `/frontend-craft:critique` — Critique your build for craft, then rebuild what defaulted

---
> Source: [samuelralak/frontend-craft-skill](https://github.com/samuelralak/frontend-craft-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
