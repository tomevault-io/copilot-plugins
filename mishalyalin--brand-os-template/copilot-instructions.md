## brand-os-template

> This file is read automatically by Claude Code (and any Claude-compatible agent) when this repository is loaded as a workspace or plug-in. Anything written here applies to every session that touches the Brand OS.

# CLAUDE.md - Instructions for any Claude session loading this Brand OS as a plug-in

This file is read automatically by Claude Code (and any Claude-compatible agent) when this repository is loaded as a workspace or plug-in. Anything written here applies to every session that touches the Brand OS.

If you are a Claude session reading this, the rules below are mandatory, not advisory. The owner of this repo owns the brand voice. Get the voice right.

## First-time setup

If `00-foundations/positioning.md` still contains `{{ YOUR_POSITIONING_LINE }}` placeholders, the Brand OS has not been onboarded yet. Stop and run the wizard before generating anything else:

```bash
python3 tools/onboard.py
```

The wizard fills the canonical foundation files via interactive Q&A (about 25-30 questions, save-as-you-go, safe to interrupt and resume).

## What this repo is

The single source of truth for everything this brand says to a customer. Brand voice, positioning, persuasion canon (Behavioral Economics + Voss/NSTD + Cialdini-Sutherland), funnel architecture, touchpoint copy, evidence, decisions, anti-patterns.

Current facts live on one page: `LAYER0-LIVE.md`. Read it before trusting a fact found in prose anywhere else in the repo. The dated decision files in `06-decisions/` (live statuses in `06-decisions/INDEX.md`) are the ultimate source; any PR that changes a fact must update `LAYER0-LIVE.md` in place in the same PR.

If you are about to write or audit:

- An email or SMS
- A PDP, lander, or hero block
- A social caption (Instagram, TikTok)
- An affiliate brief or creator script
- A founder story or About page
- A confirmation, refund, or customer-service reply
- A press response
- An investor narrative that quotes customer-facing copy

then you are working inside this repo's domain. Read the rules below before generating anything.

## The Marketing Brain - ALWAYS query it first

Before answering any marketing question, drafting any customer copy, or proposing any tactic, invoke the Marketing Brain CLI:

```bash
python3 tools/marketing_brain.py search "<natural-language question>" --top 5
python3 tools/marketing_brain.py explain "<question>"
python3 tools/marketing_brain.py tactic <tactic_name>
python3 tools/marketing_brain.py for-stage <funnel_stage>
python3 tools/marketing_brain.py for-vector <content_vector>
python3 tools/marketing_brain.py icp
```

The Brain wraps four layers, each filtered through the layer above it:

- **Layer 0 - positioning anchors** (ICP + content vectors + Wall 1 + Wall 2 + founder anchor + forbids/licenses lists + voice register refs + never-name list). The strategic frame.
- **Layer 1 - cocktails** in `01-canon/cocktail-recipes.md`. Pre-vetted stacks with Wall-1 / Wall-2 hygiene applied. Use them first.
- **Layer 1.5 - canon principles** in 7 school files: `behavioral-economics.md` (21 BE principles), `nstd-tactics.md` (21 Voss / NSTD tactics), `cialdini-sutherland.md` (22 Cialdini + Sutherland + Ogilvy BSP), `llm-seo-canon.md` (6 LLM SEO and Content Engineering), `dtc-mechanics.md` (8 post-iOS-14 DTC operating mechanics), `subscription-mechanics.md` (5 subscription retention mechanics), `pricing-mechanics.md` (5 premium-pricing mechanics). 88 principles with explicit Where-to-use / Where-NOT guidance.

Optionally a **Layer 2 - raw evidence rows** if you have a third-party research corpus. Drop it as `01-canon/nudge-vault-raw-capture.txt` (one block per `--- ID ---` line) and the Brain auto-indexes it.

The Brain serves three surfaces:

- **CLI** via `python3 tools/marketing_brain.py search "<query>"` (stdlib only, no external API)
- **HTML web interface** via `python3 web/app.py` -> `http://127.0.0.1:8081/` (pages for search / ICP / canon / tactic / vector / stage / guidelines / assets / howto / stats)
- **JSON API** at `/api/search`, `/api/explain`, `/api/icp`, `/api/canon`, `/api/tactic/<name>`, `/api/for-vector/<key>`, `/api/for-stage/<name>`, `/api/list-tactics`, `/api/list-stages`, `/api/stats` for programmatic consumers

Layer 0 is the GATE: any tactic that does not serve at least one of the content vectors AND respect both walls gets rejected. Beneath that gate, priority is cocktail > canon principle > raw evidence row.

The Brain never fabricates. If a query has no hit in the corpus, it returns `no-match` and tells the caller to gather more evidence rather than guessing.

## The seven hard rules

These are filled by the onboarding wizard into `00-foundations/brand-voice.md`. CI enforces them. A PR that violates one fails the Brand Voice Check workflow.

Examples of strong hard rules (replace with yours during onboarding):

- Short hyphens only - no em or en dashes
- No emojis in customer-facing copy
- No exclamation marks in customer-facing copy
- No medicinal vocabulary (food/supplement brands)
- Period-terminated declarative cadence
- Every external claim must trace to a primary source
- No fabricated numbers, dates, customer counts, or studies

Edit `00-foundations/brand-voice.md` and re-run `tools/marketing_brain.py rebuild-index` after onboarding to lock yours in.

## Names and identity

The onboarding wizard captures founder name + co-founder names + on-camera policy. Once filled into `00-foundations/founder-stories.md`, treat those names as immutable - misspelling or substituting them is a brand failure.

## Two voice registers (do not mix them)

The brand speaks in two registers. They are not interchangeable. Pin every touchpoint-copy file to one register at the top of the file.

- **Register A - founder narrative.** Content authored in the founder's voice (manifesto, About page, welcome flow E3, podcast intro). First-person or third-person narrative. Can carry the discipline arc. Wall 1 is less strict because the founder describes their own experience, not product claims. No medicinal vocabulary about what the product does.
- **Register B - customer testimonial.** Content authored as a customer (review widgets, IG testimonial captions, affiliate scripts, day-7 review-request email). Must follow the 5-beat honest-attribution formula (see `01-canon/cocktail-recipes.md` "The honest-attribution testimonial cocktail"). Customer discipline + goal own the outcome. Product owns the adherence-rescue mechanism, never the outcome. Wall 1 is very strict.

Mixing registers is the most common voice failure. A founder who sounds like a customer ("I lost N kg with our product") is a Wall-1 violation. A customer who sounds like a founder ("Our product is what got me through") is creepy. Each register stays in its lane.

Full detail: `00-foundations/brand-voice.md` "Two voice registers" section.

## Qualifier discipline rule (CI-enforced)

Any reference in customer copy to the customer's INPUT or GOAL must pair with a customer-ownership qualifier within 200 characters. The qualifier-guard step in `.github/workflows/brand-voice-check.yml` enforces this on every PR. If the guard fires, fix the trigger phrase by adding the qualifier - do not bypass.

Triggers and qualifiers are configurable. Default trigger phrases match a common diet/discipline brand pattern ("stick with your diet" / "reach your goal" / "achieve your goal"). Default qualifiers include both second-person ("the diet you chose" / "you chose") and third-person ("the diet they chose" / "their goal") customer-ownership phrases.

The guard skips files that still contain `{{ PLACEHOLDER }}` markers - it only fires on authored customer copy. This means foundation-file documentation that uses trigger phrases as examples does not trigger the guard.

Reference pattern: `06-decisions/REFERENCE-positioning-load-bearing-elements.md`.

## ICP-defensiveness on founder labels

Do not label the founder by specific lifestyle / diet / identity in customer copy if that label polarizes a meaningful share of the ICP. Use customer-owned framing instead. "Vegan founder" / "Keto founder" / "Marathoner founder" / "[Religion] founder" all polarize. "Founder on their own diet" / "Founder with their own discipline" / "Founder with their own goal" do not.

The founder's specific lifestyle stays in long-form interview content where the reader has self-selected to hear the detail. It does not appear in hero / manifesto / paid-ad creative where it can flip a stranger against the brand before they hear the rest.

Full detail: `00-foundations/founder-stories.md` Rule 2.

## When you produce a manifesto

The brand manifesto lives at `00-foundations/manifesto.md` and renders as the home page of the Brand OS web (`/` route) once authored. It has exactly seven sections in this order. Each section carries a specific function. Drop any one and the manifesto loses its load.

1. **3-line hero** - compressed mission, period-terminated, one breath per line
2. **Why we exist** - founder origin in narrative form (apply `00-foundations/founder-stories.md` Rule 1 chronological correctness + Rule 2 ICP-defensiveness)
3. **What we do** - operational mission lifted from `00-foundations/positioning.md` Mission V1
4. **Where we are going** - vision, future-state language
5. **What we believe** - 5 belief statements
6. **What we are not** - 3-line negation block (Wall-1 protection by explicit denial; the single place medicinal-register words appear in the manifesto)
7. **You** - reader-as-protagonist + community identity close, ending with the founder sign-off

Workflow:

1. Lock positioning first (`00-foundations/positioning.md` Mission V1/V2/V3 + three pillars + 5 load-bearing elements)
2. Lock founder arc (`00-foundations/founder-stories.md` Rule 1 + Rule 2)
3. Fill `00-foundations/manifesto.md` - replace every `{{ PLACEHOLDER }}` marker
4. Verify all 5 load-bearing positioning elements are present (manifesto.md has a verification checklist)
5. Run brand voice check CI locally (em-dash + exclamation + qualifier guard)
6. Open PR. CI green. Merge. Manifesto auto-renders as the `/` home page.

Iteration policy: manifesto versions follow semantic versioning. Patch bumps (v1.0.1) for typos. Minor bumps (v1.1.0) for surgical edits that add imagery / scenes / reframes WITHOUT changing structure. Major bumps (v2.0.0) for structural rewrites (changing the 7 sections or the 5 load-bearing elements). Every iteration ships with a decision record in `06-decisions/`.

Reference pattern: `06-decisions/REFERENCE-manifesto-architecture.md`.

## When you produce customer copy

Workflow:

1. Run the Brain to find the right cocktail or tactic with evidence
2. Pin the file to one voice register (A founder narrative OR B customer testimonial) in its front matter
3. Draft the copy stacking the recommended tactic(s) inside that register
4. Run the brand voice check (em-dash scan, exclamation scan, medicinal scan, qualifier-guard scan)
5. Run a regulatory exposure check if the copy makes any factual or pricing claim
6. Save to the appropriate `03-touchpoint-copy/` file
7. Open a PR; let CI run the same checks; the owner reviews

## When you produce a decision record

Save it as a date-stamped file in `06-decisions/`. Format: `YYYY-MM-DD-<slug>.md`. Append-only. Body should include: what was decided, when, why, what alternatives were considered, what the trigger event was.

## When you produce evidence

Save to `05-evidence/<topic>/` with a date-stamped filename. Append-only. Never edit or delete an evidence file - if it's superseded, add a new one referencing the old one.

## When you produce a new cocktail

Cocktails go in `01-canon/cocktail-recipes.md` as `### <Name>` blocks. Each cocktail must include:

- Tactic stack (which behavioral principles are layered)
- Funnel stage (where it fires)
- Verbatim copy or copy template
- Primary citation for each principle
- Wall-1 / Wall-2 hygiene confirmation
- Notes on when NOT to use

Re-run `python3 tools/marketing_brain.py rebuild-index` after adding a new cocktail.

## Multi-variant default output mode

For any customer-facing copy generation slot (hero, PDP h1, ad headline, email subject + preview, microcopy, captions, push, SMS, founder essay opener, press boilerplate, About openers, welcome-flow openings), the default output is **N variants (4-6) per slot**. Each variant:

- Cites a specific canon principle (Layer 0 / BE / Voss-NSTD / Cialdini-Sutherland / DTC / Pricing / LLM SEO)
- Is tagged with Wall-1 and Wall-2 hygiene status
- Is star-weighted 1-5 stars based on lens-fit + decider-veto risk + load-bearing principles served
- One variant marked PREFERRED with a one-sentence rationale

Single-variant mode is reserved for locked canon (manifesto, mission V1, hero 3-line, identity close, founder sign-off), single-word labels, time-sensitive ops, legal/regulatory copy, and surfaces where the brand owner specified exact wording.

Full pattern: `06-decisions/REFERENCE-multi-variant-default-mode.md`. Cocktail implementation: "The multi-variant decision-support cocktail" in `01-canon/cocktail-recipes.md`.

## Category-anchor reframe (legal vs customer mental categories)

A product can legally be classified as one thing (food / supplement / cosmetic) while customer-side living in a different mental category (the protocol-stack slot / the discipline tool / the recovery tool). The two anchors are separable. If your price feels wrong against the customer's mental reference class, fix the reference class - not the price.

Apply this pattern when your price feels "too expensive" against the customer's mental reference class, even though it is competitive against the actual peer set in the customer's life. The hero copy and PDP h1 are where the reframe is made: lead with what the product DOES, hold the product noun.

Full pattern: `06-decisions/REFERENCE-category-anchor-reframe.md`. Cocktail implementation: "The category-anchor reframe cocktail" in `01-canon/cocktail-recipes.md`.

## Founder credential is problem-anchored, never product-anchored

**Founder credentials describe what the founders DID and what PROBLEM they solved. They never describe what the PRODUCT IS.** Product reveal comes later in the copy, after the problem frame has been established in the reader's head.

Apply the verb-not-noun lesson to the founder credential paragraph: WHO, WHAT THEY DID, FOR WHOM. The founder credential does NOT answer what the product is, where it came from, or how it is made. Those answers come AFTER the problem-frame is locked.

Full pattern: `06-decisions/REFERENCE-founder-credential-problem-not-product.md`. CI enforcement: add "founder credential product-noun scan" to your `.github/workflows/brand-voice-check.yml` if you want this rule enforced on every PR.

## Phase 1 narrow / Phase 2 expand (PMF sequencing)

When your brand has done category-anchor work and your pricing depends on the new mental category holding, Phase 1 customer-facing copy addresses your narrowest core ICP only. Phase 2 expansion copy widens to friend-of-friend / adjacent audiences after the Phase 1 base validates the category-anchor in the wild.

The non-core-ICP friend-of-friend social-discovery channel is real and observable, but stays out of Phase 1 hero / PDP / paid social / IG / email / press copy. Surfacing those voices in Phase 1 collapses the category-anchor reframe because non-core-ICP audiences have no reference class to slot the new category into.

This is not exclusion - anyone who finds the brand and chooses to buy is welcomed. It is sequencing of which voices surface in Phase 1 customer copy.

Set a data-driven transition trigger (launch + 90 days OR N active subscribers; NPS target; press wave landed). When the trigger fires, Phase 2 expansion ADDS non-core-ICP testimonials ON TOP of the core-ICP base.

Full pattern: `06-decisions/REFERENCE-pmf-sequencing-phase1-narrow-phase2-expand.md`.

## Context exclusion (targeting rule, not gatekeeping rule)

Not all target markets share the pain your product solves. Exclude markets where the pain does not land from your acquisition targeting (paid social geo, creator partnerships, IG / TikTok briefs, PR pitches). Welcome anyone who finds the brand organically and chooses to buy.

The exclusion criterion can be palate / climate / life stage / income tier / existing-solution density / cultural-regulatory.

The wording is load-bearing: this is a **targeting rule**, not a **gatekeeping mechanism**. "We will not target X audience in our acquisition channels because the pain we solve does not land for them as a daily condition; we welcome anyone from X who finds us and wants to buy" - this reads as honest product-pain-fit. "We will not sell to X audience" - this reads as exclusion-as-judgement and creates legal exposure.

Full pattern: `06-decisions/REFERENCE-context-exclusion-targeting-not-gatekeeping.md`.

## Contrarian voices: capture as evidence, not as canon override

When a stakeholder voice contradicts a locked canon decision, capture the dissent as evidence in `05-evidence/contrarian-hypotheses/<date>-<short-slug>.md`, not as a canon edit.

The locked decision stands until the priority-hierarchy authority that locked it re-decides. Capture trigger conditions for re-evaluation so the contrarian voice gets honest reconsideration when those conditions land.

Three failure modes this rule prevents: (a) silent burying (brand learns the same lesson twice); (b) unauthorized override (priority hierarchy violated); (c) conflict-avoidance dilution (canon becomes mushy and unactionable).

The contrarian voice is honored by being captured, not by overriding. Honest record-keeping wins over silent burying or unauthorized override.

Full pattern: `06-decisions/REFERENCE-contrarian-evidence-not-canon-override.md`.

## Loading this repo as a plug-in into a fresh Claude session

```bash
# 1. Clone (or use as template)
git clone <your-fork-url>
cd <repo>

# 2. First-time only: onboard the brand
python3 tools/onboard.py

# 3. Build / rebuild the Brain index
python3 tools/marketing_brain.py rebuild-index

# 4. Local web interface (optional)
python3 -m venv .venv
.venv/bin/pip install -r web/requirements.txt
.venv/bin/python web/app.py
# open http://127.0.0.1:8081/

# 5. Open Claude Code / Cursor / any agent in this dir
# CLAUDE.md (this file) auto-loads the rules
```

Stdlib only. No external API. No vector DB. `git clone` + `python3` + ready.

## License

MIT. See `LICENSE`. Anyone can fork, modify, deploy.

---
> Source: [mishalyalin/brand-os-template](https://github.com/mishalyalin/brand-os-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
