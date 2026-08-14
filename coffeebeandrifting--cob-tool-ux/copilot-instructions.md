## cob-tool-ux

> Read this first. It's the business/history context that complements `.cursorrules`

# COB (Coordination of Benefits) Tool — Project Brief for Claude

Read this first. It's the business/history context that complements `.cursorrules`
(which covers code structure, DOM contracts, and design-system rules — read that too
before touching the wireflow HTML).

---

## What this project is

Internal Cigna tool for **Coordination of Benefits (COB)** recovery analysis — helps
COB analysts and contracting teams find claims where a competitor carrier underpaid
relative to Cigna's contracted rate, build a competitive rate comparison per payer/TIN,
and produce a report contracting teams use in negotiation prep.

**This is a completely separate project from "Contract Modeling"** (a different
wireflow repo, `coffeebeandrifting/contract-modeling-ux`). Do not conflate them.

## Repo & environments

- GitHub: `coffeebeandrifting/cob-tool-ux` (never `alive0x0` — wrong account, unrelated)
- Reference wireflow (source of truth for UX/interaction): `wireflows/cob-claim-universe-wireflow.html`
  - Live: https://coffeebeandrifting.github.io/cob-tool-ux/wireflows/cob-claim-universe-wireflow.html
    (has intermittently 404'd — fall back to local file if so)
  - Local: `file:///opt/data/home/cob-tool-ux/wireflows/cob-claim-universe-wireflow.html`
- Cursor prototype (mock data, used to design/test UI changes before porting): `localhost:5188`
- Real dev branch (actual API data — the branch UX changes get ported into): `localhost:3000`,
  maintained by dev "Fran"

## Design system — the one rule that matters most

The real app uses **Cigna Canvas**, an internal design system. When writing prompts
for Cursor (or any code agent) to build/modify screens:

- **Never suggest a specific hex color.** Describe semantic intent only —
  "success", "error/destructive", "warning/caution", "informational", "neutral/muted" —
  and let Canvas resolve the actual token.
- Status badges, designation tags, credibility pills — always look up color via a
  `STATUS_META`-style map, never a hardcoded color map.
- See `.cursorrules` §11–§12 for the full component-by-component semantic mapping
  and the "Canvas Rebuild Protocol."

## How UX changes actually get made (workflow)

1. Design/spec the change in the reference wireflow (`wireflows/cob-claim-universe-wireflow.html`)
   or discuss the decision directly.
2. Prototype it in Cursor against mock data (`localhost:5188`) to validate shape/behavior.
3. Port to Fran's real branch (`localhost:3000`) — bind to real API fields, never copy
   mock data arrays over. If a field doesn't exist yet in the API response, add the
   column with a `—` fallback and a `// TODO: expected API field name` comment; get
   backend to add the field — UI lands first.

**Cursor prompting rules learned the hard way:**
- Do ONE section at a time (header, then tabs, then table). Sending everything in one
  prompt produces flat/unstyled, semantically-wrong output.
- Explicitly say what to *remove* by exact label — "remove the JSX/HTML entirely, don't
  just hide with CSS." Cursor won't infer removal.
- End every prompt with "show me [section] before moving to the next" to gate progress.
- Never rename API fields to match wireframe variable names.
- Never touch data fetching / replace real data with mock data when porting.

## Canonical UX decisions (living spec — also in skill `cob-wireflow-ux-decisions`)

- **Projects table (Screen 3):** column order Priority → Due Date → Analyst → Status →
  Project Name → System → CB Type → Primary TIN → Primary Ratesheet → Contract Office →
  A–D System → Cigna Spend → COB Spend → Claim Count → Next Negotiation Day →
  Contractor → chevron. Entire row is clickable (Gmail-style hover, chevron fades in on
  hover, not a dedicated column). "Cigna Spend" (overall) and "COB Spend" (recovery $)
  are two distinct columns — never merge. "CB Type" values are FAC/PROF (not IPOP/IHOP).
- **Project Detail header KPIs, left→right:** Due Date | Next Negotiation | COB Analyst
  | Contractor. Due Date always filled (default = 11 months before Next Negotiation;
  analyst can override, shown in accent color when overridden).
- **Eligible Claims table:** `Include` column (sticky left, replaces old Exclude/Hide) —
  unchecking dims data cells but keeps Include/Note/Xnet interactive. Toolbar order:
  `[Initial Claims State]` `[Find & Update]` (not "Find & Replace") `[Start Repricing]`.
  Full column order, AG Grid migration details, and hidden-by-default columns are in
  `.cursorrules` §7 and skill `cob-eligible-claims-ui-spec`.
- **Competitive Comparison tab:** two-pane push layout (not overlay) — right payer panel
  animates 0→680px, left pane narrows. Credibility values are Low / Met / High (never
  "Med"). N= claim count is always read-only.
- Full canonical spec, status value tables, and known visual gaps to fix are in
  `.cursorrules` (this repo) and skills `cob-wireflow-ux-decisions`,
  `cob-projects-table-spec`, `cob-eligible-claims-ui-spec`, `cob-analysis-design`.

## Discovery research (why the tool looks the way it does)

Stakeholder interviews (Jun–Jul 2026) with COB analysts (Suki Sanford, Joni
Griglione-Masters) and contracting (Robert DiMauro, John Mitchell, Katherine Wells,
Andrew Patel) drove most UX decisions. Key findings:

- Analysts don't trust COB data alone — they cross-validate against state transparency
  files and Aon Hewitt UCST data. COB is one input into a larger "triangulation."
  Credibility (claim volume) must be visible early, not buried in a column.
  ("Low"/"Met"/"High" credibility levels; N<5 = Low/insufficient.)
- There's a real difference between "variance" and "gap" that needs explicit labeling —
  don't assume analysts read numeric deltas the same way.
- Analysts want anomalies (unknown product codes, negative variances) auto-flagged, not
  something they have to manually spot.
- Contracting wants a one-sentence verdict / talking points, not raw columns.
- Full discovery docs are checked into `wireflows/discover5/` (session transcripts +
  feedback) and summarized in skill `cob-analysis-design` references.

## Known open items (as of Jul 2026)

- Cancelled project state UX pending stakeholder confirmation (Generate Report
  available or not — 3 options on the table, see `.cursorrules` §4 area / skill
  `cob-wireflow-ux-decisions`).
- Insurance ID column flagged as a possible PHI/compliance concern — verify before
  making it prominent in any UI.
- `.cursorrules` §15 lists specific visual gaps between the Cursor-built prototype and
  reference wireflow (unstyled designation tags, native `<select>` for credibility,
  wrong footnote banner color, etc.) — treat each as a required fix, not a suggestion,
  when picking up prototype work.

## Where to look for more

- `.cursorrules` (this repo, root) — canonical, code-level: DOM ID contracts, JS
  function names, screen state machine, exact column specs, Canvas component mapping,
  validation script. Load this before writing any code or Cursor prompt.
- `wireflows/discover5/` — raw discovery session transcripts.
- If working inside Hermes/this agent's environment, skills `cob-wireflow-ux-decisions`,
  `cob-analysis-design`, `cob-projects-table-spec`, `cob-eligible-claims-ui-spec`,
  `cob-eligible-claims-table` carry the same knowledge plus extra Cursor prompt
  templates and copy-pasteable JS/HTML snippets.

---
> Source: [coffeebeandrifting/cob-tool-ux](https://github.com/coffeebeandrifting/cob-tool-ux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
