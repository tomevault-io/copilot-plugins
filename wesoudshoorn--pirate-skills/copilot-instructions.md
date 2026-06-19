## pirate-skills

> |


## Router

Parse the user's command to detect mode:

- `/piratepage positioning` → Mode 1
- `/piratepage mock [type]` → Mode 2
- `/piratepage variations` → Mode 3
- `/piratepage fast <url>` → Fast Mode (shortcut below)
- `/piratepage` (no args) → check for `piratepage.json`:
  - **Missing:** start Mode 1 (positioning first)
  - **Exists:** greet by product name, then ask via AskUserQuestion:
    "What do you want to do?
    A) Create a new mock page
    B) Update your positioning
    C) Generate copy variations for a section in your code
    D) Iterate on an existing page"

**Context detection:** if the user says "variations", "options", "tones", or "give me 5" → Mode 3. If they name a page type or say "landing page" → Mode 2.

### Fast Mode

`/piratepage fast <url>` — zero interaction until the page is in the browser.

1. Extract URL. Prepend `https://` if no protocol.
2. WebFetch the URL to extract positioning data.
3. Auto-detect language. Default to English if unclear.
4. Pre-fill all 9 positioning answers. Do not ask for confirmation.
5. Set voice to Professional. Save to `piratepage.json`.
6. Set page type to Homepage. Build outline using Section Selection Guide (richest variants).
7. Generate single-tone HTML, run quality checks, open in browser.
8. Enter Mode 2 iteration loop.

---

## Mode 1: Positioning

Build and maintain product knowledge in `piratepage.json`. Everything else reads from this.

### Check Existing Data

```bash
[ -f piratepage.json ] && echo "FOUND" || echo "NOT_FOUND"
[ -f piratepage.json ] && cat piratepage.json || true
```

**If exists:** show current positioning summary, ask what to update.
**If new:** ask for URL or description via AskUserQuestion.

### URL + Language

If URL provided: WebFetch and extract positioning. If WebFetch fails: "Couldn't scrape that URL. Describe your product in a few sentences instead."

After scraping, confirm language: "Your site is in [detected language]. Want me to generate in [detected language]?"
Options: A) Yes. B) No, use English. C) Other.
If no language detected, default to English without asking.

### The 9 Positioning Questions (the forcing function)

These force you to think through positioning before any copy is generated. You can't skip them. They're what makes the output good.

From the extracted content, pre-fill all 9 answers as best guesses. Present in **3 batches**. For each batch, use a single `AskUserQuestion` call that shows the questions with their pre-filled answers and asks the user to respond A/B/C for each. **Do NOT output the questions as regular text — they MUST be inside the AskUserQuestion prompt so the user gets an interactive input field.** Wait for the user's response before presenting the next batch.

**Batch 1 — Identity:**
1. **What is your product?** Brief, plain-language description.
2. **What is it NOT?** Common misconceptions.
3. **Key takeaway?** If a visitor remembers ONE thing.

**Batch 2 — Positioning:**
4. **Word of mouth?** How an excited user describes it to a friend.
5. **Competitors?** Direct, indirect, and "doing it manually."
6. **How are you different?** Specific differences, not generic claims.

**Batch 3 — Conversion + Voice:**
7. **Why do users want this?** What progress are they trying to make?
8. **Objections/fears?** Hesitations at signup/purchase.
9. **Primary CTA?** What should they do next?
10. **Voice/tone?** Professional / Casual / Bold / Skip — I'll decide.

After all batches, save to `piratepage.json`.

"Positioning saved. Ready to generate a page (`/piratepage mock`) or try tone variations on existing copy (`/piratepage variations`)?"

---

## Mode 2: Mock Pages

Generate standalone HTML mockup pages using the section library.

### Prerequisites

Check `piratepage.json` exists. If not: "Run `/piratepage positioning` first (or `/piratepage fast <url>` for the quick version)."

### Page Type Selection

Ask via AskUserQuestion:

"What kind of page are we building?

A) Homepage — the full story: what it is, why it matters, proof, CTA (6-10 sections)
B) Product page — deep feature showcase (6-8 sections)
C) Service page — trust-first, process-focused (5-7 sections)
D) Pricing page — plans front and center (4-6 sections)
E) Customer Story — one customer's journey from problem to results (5-7 sections)
F) Competitor page — 'Product vs X' comparison that wins the search (6-8 sections)
G) Feature Tour — deep-dive on a single feature, outcome-first (6-8 sections)
H) Use Case page — 'Product for X' vertical landing page (5-7 sections)"

If `[type]` was passed as argument, use it directly.

**Competitive Mode:** if the user mentions a competitor URL and the page type is NOT Competitor, WebFetch it and surface differentiation: "Your competitor leads with [X]. How do you compare?" Frame copy to resonate with visitors who already considered the competitor. Don't attack by name.

### Outline + Generation

1. Build section outline using Page Type Templates + Section Selection Guide below. Respect `preferredSections` from `piratepage.json` if present.
2. Present outline with rationale. Ask: A) Generate it. B) Change something. C) Different page type.
3. Generate single-tone HTML using the voice from `piratepage.json`. One clean file, no variations chrome.
4. Run all quality checks. Open in browser.

### HTML Template (single tone)

```html
<!DOCTYPE html>
<html lang="{language-code}">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{productName} — {keyTakeaway}</title>
  <script src="https://cdn.tailwindcss.com"></script>
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
  <style>
    body { font-family: 'Inter', system-ui, sans-serif; }
  </style>
</head>
<body class="antialiased bg-white text-neutral-900">
  <!-- navbar -->
  <!-- sections from outline -->
  <footer class="border-t border-neutral-200 py-6 text-center text-sm text-neutral-400">
    Generated with PiratePage
  </footer>
</body>
</html>
```

### Reading Reference Files

For each section in the outline, read the corresponding reference HTML from the skill's `references/` directory:

```bash
find ~/.claude/skills -name "SKILL.md" -path "*/piratepage/SKILL.md" 2>/dev/null | head -1 | xargs dirname 2>/dev/null || \
find .claude/skills -name "SKILL.md" -path "*/piratepage/SKILL.md" 2>/dev/null | head -1 | xargs dirname 2>/dev/null || \
echo "NOT_FOUND"
```

If a file doesn't exist, generate HTML using Tailwind classes consistent with the page style.

### Image Placeholders

Reference files contain wireframe UI mockups as placeholders — not grey boxes. These use `aspect-ratio` for sizing (never `min-height`):

- **Large (aspect-video):** Dashboard wireframes with browser chrome. Hero with-screenshot, screenshot sections.
- **Medium (aspect-[4/3]):** Form/card wireframes. Hero with-image, features-list spotlight.
- **Small (aspect-[3/2]):** Mini list or card-preview wireframes. Features-grid showcase, how-it-works steps.

Copy wireframe HTML directly from reference files. Each keeps a `[img: {description}]` label at bottom-right. Do not replace wireframes with grey boxes.

### Invented Content Warning

```html
<!-- NOTE: Testimonials below are AI-generated examples. Replace with real customer quotes before publishing. -->
```

### Auto-Open

```bash
open "{slug}-landing-page.html" 2>/dev/null || xdg-open "{slug}-landing-page.html" 2>/dev/null || echo "Open {slug}-landing-page.html in your browser."
```

### Iteration Loop

Two-phase flow. Every transition is a forced `AskUserQuestion` choice.

**Phase 1: What's Next?**

"Your page is ready. What's next?

A) Style it — I'll apply a visual design (read `references/visual-upgrade.md` for the full system)
B) Change the copy
C) Try different tones for specific sections → generates 5 variations inline
D) Export — output a clean HTML file
E) Done"

- **A → Visual Upgrade** (read `references/visual-upgrade.md` from the skill directory, follow that system)
- **B → Phase 2** (copy editing)
- **C → Mode 3** (variations on sections of this page)
- **D → Export** (Markdown and/or JSON if requested)
- **E → Exit**

**Phase 2: Copy Editing**

"What do you want to change?

A) Rewrite a specific section
B) Shuffle section order
C) Add/remove a section
D) Regenerate whole page with feedback
E) Actually, let's style it instead"

After every copy change, loop back to Phase 1.

---

## Mode 3: Variations

Generate 5 tone variants for a section in the user's actual codebase. They preview options with a picker toolbar, choose favorites, and get clean code back.

### Target Identification

Ask what to generate variations for. The user can:
- Point to a file + line range: "the hero in `src/app/page.tsx` lines 15-45"
- Describe it: "the hero section"
- Say "all copy sections on this page"

Read the target code. Identify the section type (hero, pain, features-grid, etc.).

**Skip list** — these are factual/external content, not generated copy: testimonials, social-proof, stats, results, pricing, comparison-table, code-sample, news, showcase. If the user targets one of these, explain why and offer to help rewrite manually instead.

### Variant Generation

Requires `piratepage.json` for positioning context. If missing, prompt to run `/piratepage positioning` first.

Generate 5 tone variants using the Tone Definitions below. Rules:

- **Same HTML structure, same classes, different text only.** Rewrite: headlines, subheadlines, body copy, bullet text, CTA text, eyebrow text.
- **Do NOT change:** class attributes, HTML element structure, number of elements, image/wireframe placeholders, section order.
- Each variant must pass all standard quality checks.
- Variants must be meaningfully different — not synonym swaps. Each tone leads with a different angle.
- Each variant must feel like a coherent voice throughout its section.
- For JSX/TSX files: use `className` instead of `class`. The `hidden` attribute works as-is in React.

### Picker Markup

Wrap the section with `data-pp-pick` / `data-pp-option`:

```html
<div data-pp-pick="hero" class="contents">
  <div data-pp-option="1" class="contents">
    <!-- version 1 -->
  </div>
  <div data-pp-option="2" class="contents" hidden>
    <!-- version 2 -->
  </div>
  <div data-pp-option="3" class="contents" hidden>
    <!-- version 3 -->
  </div>
  <div data-pp-option="4" class="contents" hidden>
    <!-- version 4 -->
  </div>
  <div data-pp-option="5" class="contents" hidden>
    <!-- version 5 -->
  </div>
</div>
```

Always generate 5 options, numbered 1-5. The picker toolbar shows numbered pills. First option visible, rest have `hidden`. The `contents` class on wrappers prevents layout shifts (CSS `display: contents`).

### Script Injection

Add the picker script before `</body>`. Inject once even if multiple sections have pickers.

**Default — load from CDN** (simple, works in most apps):

```html
<script src="https://cdn.jsdelivr.net/gh/wesoudshoorn/pirate-skills@latest/piratepage-picker.js"></script>
```

**Fallback — inline, if the target app has a CSP that blocks external scripts.**
Symptoms: browser console shows `violates the following Content Security Policy directive: "script-src ..."` for the jsdelivr URL. Common in apps that explicitly configure CSP (middleware, `next.config`, or `<meta http-equiv>`).

In that case, read `piratepage-picker.js` from the skill directory (same directory as `SKILL.md` — see detection snippet in "Reading Reference Files" above) and inject its contents inline instead:

```html
<script>
{contents of piratepage-picker.js}
</script>
```

This is a dev-time tool only — do not include it in production builds. For JSX/TSX in a React app: add the script tag to the app's layout file temporarily (e.g. `layout.tsx`), or create a small temporary HTML preview file that renders the component.

### Preview + Selection

Present: "Your page has copy variants for N sections. Toggle between tones using the toolbar. Tell me which tone works for each section, or paste the URL hash."

### Cleanup

After the user picks:
1. For each `data-pp-pick` group: keep the selected `data-pp-option` contents, remove all others.
2. Unwrap the `data-pp-pick` div (replace with its children).
3. Unwrap the remaining `data-pp-option` div (replace with its children).
4. Remove the injected `piratepage-picker.js` script block.
5. Verify no leftover `data-pp-*` attributes remain.
6. Save the clean file.

---

## Copywriting Rules

Battle-tested. 81/81 blind test accuracy. Apply to every line of copy.

### Headlines: Clear Before Clever
Every headline must name what the product does or who it's for. Reading only headlines top-to-bottom should tell the complete value story. If someone reads only the headline and skips the body, they should still know they're looking at a page about the product's category.

### Banned Headline Patterns
- "Everything you need to [verb]"
- "One [thing] for [generic benefit]"
- "Our clients love our [quality]"
- "The [category] you can actually [verb]"
- "From [X] to [Y]" when both are generic
- "[Brand Name] — [generic tagline]"
- "[Generic adjective] [category] for [vague audience]"

### Swap Test (Hero + CTA Only)
Could a competitor use this exact headline? If yes, rewrite.
- **Track 1:** Name a specific thing (feature, number, technology). "Email API built on React"
- **Track 2:** Describe it so specifically only YOU match. "Gentle sleep coaching: baby sleeping through the night in 2 weeks"

### Headline Formulas (adapt, never copy verbatim)
- **Outcome:** "{Outcome} without {pain}" / "Turn {input} into {outcome}"
- **Problem:** "Never {bad thing} again" / "Stop {pain}. Start {pleasure}."
- **Audience:** "The {category} for {target audience}"
- **Differentiation:** "Finally, {category} that {benefit}"
- **Proof:** "{Number} {people} use {product} to {outcome}"

### Eyebrows
2-4 words max. Product fact, not section label, not a sentence.
BAD: "FEATURES", "OUR PROCESS", "WHY CHOOSE US", "EVERYTHING YOU NEED"
GOOD: "REACT EMAIL API", "60 SECONDS", "TRUSTED BY 2,400+ TEAMS"

### Content Rules
- Headlines: 10 words max, no quotation marks
- Subheadlines: 1-2 sentences max
- Pair every feature with its benefit
- Testimonials: 1-2 sentences, specific outcomes, name + role + metric
- Every section has ONE clear CTA
- CTA formula: [Action Verb] + [What They Get]. Never: "Submit", "Learn More", "Click Here", "Get Started"
- CTAs feel low-risk: free trials, guarantees, "no credit card required"

### Voice
Imperfect rhythm, conversational but not forced. Specific over vague. One strong claim beats three weak ones. Show empathy before selling. Pain sections open with "you" or a scenario, not the product.

### Tone
Confident but not arrogant. Helpful but not pushy. A little funny when it fits.

### Never Sound Like AI
Banned phrases: "That being said", "It's worth noting", "At its core", "In today's digital landscape", "Let's delve into", "When it comes to", "Seamlessly", "Effortlessly", "Cutting-edge", "Revolutionize", "Let's dive in", "Without further ado", "I'm thrilled to announce", "We're proud to..."
Banned vocabulary: delve, crucial, robust, comprehensive, nuanced, leverage, foster, showcase, pivotal, transformative, game-changing, paradigm.

---

## Tone Definitions

The 5 tones used for Mode 3 variations:

1. **Punchy** — Short. Strong verbs. High energy.
2. **Conversational** — Warm, like a knowledgeable friend.
3. **Benefit-focused** — Leads with outcomes. "So what does this mean for me?"
4. **Problem-aware** — Names the frustration before the solution.
5. **Bold-confident** — Assertive. Strong claims backed by specifics.

**Skip list** (factual/external content — no tone variants):
testimonials, social-proof, stats, results, pricing, comparison-table, code-sample, news, showcase

---

## Quality Self-Validation

Run BEFORE presenting to user. Fix failures silently.

1. **Swap Test** — Hero + CTA headlines. Could a competitor use it? Rewrite if yes.
2. **Headline Scan** — Read headlines top-to-bottom. Do they tell the value story for THIS product?
3. **Specificity Check** — Every section must reference something specific (name, feature, number, technology).
4. **Banned Pattern Scan** — Check all copy against banned phrases and vocabulary.
5. **Eyebrow Check** — 2-4 words, product facts, not labels, not sentences.
6. **FAQ Headline Check** — Must NOT be "FAQ" or "Frequently Asked Questions." Lead with the #1 objection.
7. **Wireframe Check** — Product UI sections must use wireframe placeholders from reference files (aspect-ratio sized, not min-height grey boxes).
8. **CTA Check** — [Action Verb] + [What They Get]. No "Learn More" or "Get Started."
9. **Topic Anchoring Check** — Every headline must make clear what the page is about. If a headline could be on any website, it's too generic.
10. **Variation Distinctness Check** (Mode 3 only) — Read the headlines of all 5 tones for each section. They must be meaningfully different. Each tone leads with a different angle.

### Page-Type-Specific Quality Checks

Run IN ADDITION to the standard checks, based on page type.

**Competitor Page** (checks 11-16):
11. **Factual Accuracy** — Every claim about the competitor must be verifiable from their public site.
12. **Specificity** — Comparison points cite concrete differences, not vague claims.
13. **Honesty Check** — Honest assessment section genuinely acknowledges 1-2 competitor strengths.
14. **Migration Clarity** — Migration steps are concrete and actionable.
15. **Search Intent Match** — Satisfies "[Product] vs [Competitor]" search intent.
16. **No Legal Risk** — No trademark misuse, no competitor UI screenshots, verifiable facts only.

**Feature Tour Page** (checks 11-15):
11. **Outcome-First Headlines** — Hero names what the user achieves, not the feature name.
12. **Wireframe Quality** — Every how-it-works step has a wireframe showing that step's UI.
13. **Step Clarity** — Each step is a single clear action.
14. **Code Validity** — If code-sample included, must be syntactically valid.
15. **Internal Linking** — Related features reference real product features.

**Use Case Page** (checks 11-15):
11. **Audience Swap Test** — Replace target audience with a different one. If copy still works, it's too generic.
12. **Vocabulary Check** — Pain points use the audience's industry terms.
13. **Scenario Specificity** — Pain section describes scenarios this audience actually faces.
14. **Feature Relevance** — Every feature has a clear "why this matters to [audience]" connection.
15. **Proof Relevance** — Social proof comes from the same audience vertical.

---

## Section Selection Guide

### Available Sections (21 types)

**Navigation:**
- `navbar` — Sticky top nav bar. 2 variants: default (text logo), with-logo-placeholder (image logo slot). Always placed first, before hero.

**Opening sections:**
- `hero` — The 5-second pitch. 5 variants: default, with-image, with-proof, with-screenshot, with-checklist.

**Problem sections:**
- `pain` — Name the frustration. 2 variants: default (numbered list), before-after (two-column contrast).

**Solution sections:**
- `how-it-works` — 3-4 step process. 2 variants: default (text), with-images (screenshots per step).
- `features-grid` — Scannable feature cards. 5 variants: default, bento, icon-grid, showcase, tabs.
- `features-list` — Detailed feature descriptions. 3 variants: default, spotlight (alternating), highlight (with quotes).
- `screenshot` — Full-width product UI. 2 variants: default, with-benefits.

**Proof sections:**
- `testimonials` — Customer quotes. 3 variants: default (stack), featured (single large), with-results (metric cards).
- `social-proof` — Logo bar. Company logos and trust badges.
- `stats` — Key metrics. 3 variants: default (grid), with-context, proof-bar (single line).
- `results` — Customer outcomes. 2 variants: metrics (numbers), story (narrative).

**Objection handling:**
- `faq` — Accordion Q&A. Address objections with the user's actual voice.
- `comparison-table` — Side-by-side vs competitor.
- `pricing` — Plans and pricing. 2 variants: default (multi-plan), single.

**Closing sections:**
- `cta` — Final call to action. 3 variants: default, with-proof, with-trust.

**Special sections:**
- `text-block` — Long-form narrative (founder story, mission).
- `showcase` — Portfolio/examples. 3 variants: grid, cards, logo-cards.
- `statement` — Single bold claim, full-width.
- `news` — Timeline of announcements.
- `code-sample` — Code blocks with syntax highlighting.
- `founder-story` — Personal narrative with signature.

### Variant Selection Rules

**Default to the richest variant.** Screenshot placeholders and visual elements make pages more compelling. Only choose simpler variants when the product genuinely has nothing visual. When in doubt, include a screenshot placeholder.

- **Hero:** Default: `with-screenshot`.
  - Fall back to `with-checklist` or `with-proof` for non-visual products.
  - `default` (text-only) should almost never be chosen.
- **Features-grid:** Default: `bento` with screenshots.
  - Fall back to `icon-grid` for 6+ brief features.
  - Fall back to `tabs` for complex sequential features.
- **How-it-works:** Default: `with-images`.
  - Fall back to `default` for non-visual products only.

### Page Type Templates

**Homepage** (6-10 sections):
- **MUST:** navbar, hero (with-screenshot preferred), 1-2 feature sections (bento preferred), cta
- **SHOULD:** pain OR how-it-works (with-images), social-proof, testimonials, faq

**Product Page** (6-8 sections):
- **MUST:** navbar, hero (with-screenshot), features-grid or features-list, cta
- **SHOULD:** how-it-works (with-images), testimonials or stats
- **SKIP:** pain

**Service Page** (5-7 sections):
- **MUST:** navbar, hero, how-it-works (with-images), cta
- **SHOULD:** testimonials or results, features-list
- **SKIP:** features-grid, pricing

**Pricing Page** (4-6 sections):
- **MUST:** navbar, hero (minimal), pricing, faq, cta
- **SHOULD:** testimonials (ROI-focused), comparison-table
- **SKIP:** pain, how-it-works

**Customer Story** (5-7 sections):
- **MUST:** navbar, hero, results, cta
- **SHOULD:** pain or text-block, testimonials
- **SKIP:** pricing, features-grid

**Competitor Page** (6-8 sections):
- **MUST:**
  - navbar
  - hero — switch-pitch: name the competitor, lead with why someone would consider switching
  - comparison-table — 5-8 dimensions, include 1 where competitor wins
  - 2-3 features-list spotlight — deep-dive differentiators
  - faq — comparison objections
  - cta
- **SHOULD:**
  - text-block — honest assessment of competitor strengths
  - how-it-works with-images — migration guide
- **SKIP:** pain, pricing, social-proof

**Feature Tour Page** (6-8 sections):
- **MUST:**
  - navbar
  - hero — outcome-first: what this feature lets you achieve
  - pain — scoped to this feature's problem
  - how-it-works with-images — walkthrough with wireframe per step
  - features-grid — capabilities, 3-4 cards
  - cta
- **SHOULD:** code-sample (technical audience), showcase (use case mini-stories), features-list (related features)
- **SKIP:** pricing, comparison-table, social-proof

**Use Case Page** (5-7 sections):
- **MUST:**
  - navbar
  - hero — audience-specific vocabulary
  - pain — 3-4 audience-scoped pain points
  - features-grid or features-list — features through audience lens
  - cta — audience-specific language
- **SHOULD:** how-it-works (their workflow), testimonials or results (same vertical), faq (audience objections)
- **SKIP:** pricing, comparison-table, code-sample

### Narrative Arcs

- **Homepage:** Recognition → Problem → Solution → Proof → Objections → Action
- **Product:** Clarity → Value → How it works → Proof → Action
- **Service:** Promise → Understanding → Process → Proof → Expertise → Action
- **Pricing:** Context → Plans → Comparison → Objections → Proof → Action
- **Customer Story:** Hook → Challenge → Solution → Results → Broader proof → Action
- **Competitor:** Switch Pitch → Quick Comparison → Deep Differentiators → Honest Assessment → Migration → Objections → Action
- **Feature Tour:** Outcome Promise → Problem Scoped → Walkthrough → Capabilities → Use Cases → Related → Action
- **Use Case:** Audience Recognition → Their Pain → How It Works for Them → Features (Their Lens) → Proof (Their Peers) → Objections → Action

---

## Notes

- Stay conversational. Natural language in AskUserQuestion prompts, not robotic lists.
- Briefly explain 2-3 key copy decisions after presenting the page.
- **Section Gallery:** Users can run `/piratepage-gallery` to browse all section types and variants visually.
- Never present without quality checks.
- `piratepage.json` is persistent state. Update on every change.
- Detect skill directory dynamically — never hardcode paths.
- When language is non-English, generate natively. Don't translate.

---
> Source: [wesoudshoorn/pirate-skills](https://github.com/wesoudshoorn/pirate-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
