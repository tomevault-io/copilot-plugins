## ctrl-alt-design

> This is the persistent brain for the `ctrl-alt-design` repo (Next.js + Tailwind, deployed to Vercel).

# CLAUDE.md — the elleta.design constitution

This is the persistent brain for the `ctrl-alt-design` repo (Next.js + Tailwind, deployed to Vercel).
Every session, every ticket, every agent obeys this file. It is the constitution, not notes.
Pairs with `docs/portfolio-conformance-spec.md` (visual contract), `docs/portfolio-ia-spec.md`
(navigation/IA), and `docs/harness-and-baseline.md` (how to make changes safely).

If anything below conflicts with what a prompt asks, STOP and surface the conflict — do not silently
override the constitution.

---

## 0. Prime directives (read first)
1. **Harness yourself.** Do not create whatever you want. Do the smallest change that satisfies the task.
   No new components, routes, patterns, colors, or copy unless the task requires them.
2. **One implementation.** Edit the LIVE component/route and delete the old one. Never leave old + new
   both rendering. If a change isn't global, you edited a dead copy — grep for orphans before finishing.
3. **Spec before build.** For anything beyond a one-line fix, write a spec first (see §8). Do not vibe-code.
4. **Baseline before change.** Before altering an existing page, rebuild/confirm its current state from the
   real components as a baseline, THEN apply the change (see `docs/harness-and-baseline.md`).
5. **Prove it globally.** After any change, run `npm run gate`, tsc, all routes 200 (light + dark), and the
   NDA content-grep. Report a diff summary. Green or it isn't done.

## 1. Tokens (never hardcode)
- **No hardcoded hex or px in components.** Reference tokens only. No arbitrary Tailwind `text-[Npx]` /
  `bg-[#...]`. Spacing and type come from the scale, not ad-hoc values.
- **Body min 16px.** Never smaller for reading text.
- **No pure white and no pure black** as surfaces/text. Warm neutrals only.
- BELLA core: ground `#F5F4EF` (light) / navy `#1B1B40` (dark); ink `#1A1720` / `#F4EFE6`;
  accent iris `#5B4BD1` / periwinkle `#A79CE2`. **No amber anywhere.**
- Cascade trap: BELLA's unlayered `:root` beats `@theme`. Keep app theme tokens in an unlayered
  `:root` that loads AFTER imports so they win.

## 1b. IA (nav)
- Primary nav (Elleta, 2026-07-17, supersedes the four-item cap): **Work · System · Skills ·
  About · Contact**. /design-system is a first-class page (the system inspecting itself);
  the footer "See the system" colophon link stays.
- **Work toolbar (amended 2026-07-20, Pass E task 3; supersedes the 17 Jul filter-row note).**
  ONE toolbar row above the library: find-your-fit search on the LEFT (always visible), view
  switcher on the RIGHT (SegmentedControl, Cards · Map · Table, always visible). The chip row
  beneath the search is the library's ONE skill/type filter, in EVERY view (the former CASE and
  SKILL rows are deleted); one stable order everywhere: toolbar, chip row, count, content. Cards
  is the default and IS the curated composition (featured CHIP, ranked case grid, Explorations),
  and it filters like every view. Sort renders only where order means something: table headers,
  never the Map. No hidden explore state: the view lives in the URL (`view` param, back/forward
  safe, defaults keep clean URLs).

## 2. Layout
- One centered container, **max-width 1240px**, consistent horizontal padding, every page. Never full-bleed text.
- Vertical rhythm from the scale (`--space-section` = 96px desktop). No inline/ad-hoc paddings.
- Cards fill the grid evenly (equal heights, consistent gaps).

## 3. Type
- **Exactly two typefaces (revised 2026-07-17, supersedes the hero-only lock).** Unique 700 = ALL
  display headings: home hero headline, page titles, section headers, case-study display headlines,
  and the keycap brand lockup, always all-caps with the established accent-word treatment where the
  design already does that. Every display heading renders through the ONE `ui/Heading` primitive
  (tiers: hero / page / section / case). Page openings are FLAT (eyebrow + Heading, the Work
  pattern); bubble page headers are parked (last live at e25eefc, may return in the expression
  pass). The elevation/orb tokens stay: keycaps, the home cluster, and the About portrait still
  consume them.
- Unique never renders below 24px except the keycap logo (the gate enforces this), and never in
  body, UI, card titles, eyebrows, meta, nav links, buttons, or chips.
- **Unique never renders inside a Card (Elleta, 2026-07-21, card-voice).** Cards use Geist only;
  Unique stays page-tier (the Heading primitive: section heads and heroes). Card statements use
  the shared `.card-statement` recipe (Geist 700 at `--font-card-title`), card titles the shared
  `.heading-item`. Enforced by the Unique-in-card check in `audit:reuse`.
- Geist = everything else. Eyebrows stay Geist caps with `--tracking-eyebrow`.
- **Numbers in columns are right-aligned and tabular (Elleta, 2026-07-28, readability
  audit).** Any figure that sits in a column beside other figures (a table cell, a grid
  column, a stat row) uses `text-align: right` and `font-variant-numeric: tabular-nums`,
  so digits share a width and the values share a right edge. A column of numbers that
  starts wherever the previous word ended is not a column. Prose numbers are unaffected.
  Enforced by the numeric-alignment check in `audit:structure`.

## 4. Color & dark mode
- **Colour affordance rule (refined 2026-07-17):** saturated iris at body scale means INTERACTIVE,
  and only that. Eyebrows/kickers: weight 700, tracked, NEVER iris; on case-scoped surfaces they
  wear that case's identity colour (`--case-*-text`, AA on their ground); on neutral surfaces
  `--color-eyebrow` (ink-soft). Inline body links are iris AND underlined. Decorative purple uses
  periwinkle tints. Display headings keep their iris accent word. Enforced by the no-iris eyebrow
  check in `audit:structure` + the live AA sweep in `audit:contrast`.
- Every surface/text/border resolves from semantic tokens via `[data-theme="dark"]`. No hardcoded values.
- Dark mode is a first-class contract on EVERY surface, not an afterthought — case pages included.
- The dark keycap logo must not bloom a heavy glow on navy; tone the plate/shadow.

## 5. Controls (one taxonomy — see conformance spec §7)
The raised **keycap** is reserved for the brand logo and TRUE actions only. Do not use it for filters,
toggles, or sort.
- **Button (grammar v5 + primary pick, 2026-07-20):** purple means clickable at every tier.
  PRIMARY = the calm filled iris keycap, the one 3D moment per view (max ONE); hover gains the
  travelling border light (the SHARED .trace-host recipe, never a copy); focus ring independent of
  the trace; reduced motion shows the static accent ring. SECONDARY = flat iris outline, iris text,
  no fill, no elevation (periwinkle on dark and fixed-dark chrome). TERTIARY = text link, iris +
  underlined. The neutral keycap is retired.
- **SegmentedControl:** mutually exclusive views (e.g. TABLE/MAP/TIMELINE). Single-select, `aria-current`,
  lighter than a keycap.
- **FilterChip:** multi-select filters. Flat/outline, `aria-pressed`. Not a keycap.
- **Select:** RETIRED 2026-07-27, pending a real dropdown surface. The primitive had no
  consumer once the /design-system specimen showcase was retired, and /work sorts through
  table headers by design, so nothing on the site renders a dropdown. Rather than keep
  dead code alive with an audit allowlist (see the no-exemptions rule in section 9), the
  component, its contract entry and its manifest taxonomy line were deleted together.
  Restore from git the moment a surface genuinely needs one; do not rebuild it from
  scratch.
- **Tag:** non-interactive metadata. Visually distinct from FilterChip.
- **StatusPill:** quiet status (e.g. "current focus"), non-interactive.
- One light source, upper-left: highlights top-left, shadows down-right (bubbles, keycaps, cards).

## 6. Copy & voice
- **Positioning term is "AI-enabled" / "AI enablement".** Never "AI-augmented" or "AI-assisted". Keep the
  phrase in one constant and reference it.
- **No em or en dashes (—, –) anywhere.** Use a period, a comma, or "that".
- **No email address rendered anywhere on the site** (2026-07-17: scrapers harvest plaintext;
  the contact form is the channel, LinkedIn the alternative; the send address lives server-side).
- Decision-led, NDA-safe, honest. No invented metrics or exaggerated outcomes.

## 7. NDA (hard rule)
- No real internal screens, dashboards, metrics, or client tool/team names from any employer or
  client. Abstract to a descriptor ("a UN agency in Geneva"). Recreated/abstract diagrams only.
  INTERNAL terms live in `_private/nda-terms.txt` (gitignored, banned EVERYWHERE), merged with the
  global `~/.claude/nda-terms.txt`; the pre-commit hook and `audit:nda` read from both, so no name
  is ever written in a committed file, this one included.
- **Employer scoping (2026-07-20, Pass E task 9).** Employment history is public; case content stays
  abstracted. Employer and engagement org names live in `_private/nda-employers.txt` (gitignored)
  and are banned everywhere EXCEPT `components/ExperienceSection.tsx` and
  `components/ResumeModal.tsx`. No other file is exempt, ever. The library data, tags, and matrix
  stay name-free; case studies keep industry-not-client naming, recreated artifacts, and their
  disclosure lines.
- The NDA check greps file **contents across the whole tree**, not diffs or filenames — renamed files hid
  names before. Never rely on the diff alone.
- **Apple exception (Elleta, 2026-07-21, deliberate — do NOT "fix").** Apple is a public past
  employer (B2B & Consumer Sales Representative, 2011-2013) referenced only in the two Experience
  surfaces, but the word is deliberately NOT in `_private/nda-employers.txt`: "Apple" also appears
  legitimately as a platform name in Apple Music/Podcasts URLs and comments (VinylPlayer, the About
  podcast list, a globals.css comment, audit-copy, two briefs), so a repo-wide ban would fail the
  gate on all of them. Verified 2026-07-21: outside the two exempt surfaces, "Apple" occurs nowhere
  as an employer reference. If protection is ever wanted anyway, teach `audit:nda` per-term
  allowlisting first; do not just add the word to the list.

## 7b. Branch flow (Elleta, 2026-07-21, via Cowork — every session follows this)
- **No direct pushes to main.** Terminal sessions work on short-lived branches
  (`<type>/<slug>`, e.g. `fix/thesis-theme`), open a PR, and merge only on green.
- The PR carries the template: what changed, local gate output (CI cannot run
  audit:nda — the private lists never leave this machine), screenshots for
  anything visual, and the Elleta-approval line for content changes.
- Include the Vercel preview link in the PR body once the bot posts it.
- CI (`.github/workflows/gate.yml`) runs tsc + the full gate on every PR and
  every push to main; the `gate` check is REQUIRED by branch protection, and
  force-pushes to main are blocked.
- **Break-glass (approved history surgery only, e.g. the NDA scrub class of
  event):** temporarily lift protection with
  `gh api -X DELETE repos/emcdanie/ctrl-alt-design/branches/main/protection`,
  do the approved surgery, then re-apply protection immediately (the settings
  are recorded in `docs/branch-protection.json`; re-apply with
  `gh api -X PUT .../protection --input docs/branch-protection.json`). Every
  break-glass use gets a line in `claude-progress.md` with her approval.

## 8. Working method (spec → review → execute)
Use the `portfolio-spec` skill. For any non-trivial task:
1. I give intent (often a screenshot / Figma link / description).
2. You generate `specs/<slug>/design.md`, `requirements.md`, `tasks.md`. **Stop and let me review.**
3. On my go, execute the checked-off tasks start to finish.
4. Verify against the gate. Report a diff summary + before/after screenshots where visual.

## 9. The gate (`npm run gate`) — un-regressable
Must pass before any work is "done":
- `audit:structure` — per-case route dirs, container/section system, no arbitrary `text-[Npx]`, no amber.
- `audit:contrast` — WCAG AA (AAA-minded); Unique below 24px fails outside the keycap logo.
- `audit:copy` — fails on `—`/`–` and on "AI-augmented" / "AI-assisted".
- `audit:controls` — keycap used as filter/toggle/sort fails; >1 primary per view fails; filters/toggles
  missing `aria-pressed`/`aria-current` fail.
- `audit:fonts` — any face other than the Unique/Geist tokens fails; Unique outside the Heading
  primitive, home hero, or keycap lockup fails; any mono family reference fails.
- `audit:tokens` — colour literals and raw spacing (>=4px) in `app/**`/`components/**` fail;
  `token-waiver:` inline comments mark the reviewed proto-exact/artwork exceptions.
- `audit:parity` — every case-study slug has exactly one `WORK_ITEMS` row and vice versa; side
  tables for case identity (the deleted `EXTRA_CASES` pattern) fail.
- `audit:contract` — every component in the contract exists, token $refs resolve, no entry for a
  deleted component (the machine-readable component contract at /api/bella.json). bella.json is
  GENERATED from source (tokens + `lib/bella/component-contract.json`); patch the source, never
  the served artifact.
- `audit:axe` — axe-core over every route in BOTH themes; zero violations to pass (needs-review
  nodes are counted, not failed, and verified by hand when they change).
- `audit:type` — no Card surface renders reading text below 16px COMPUTED; the shared
  `.card-body` recipe never computes below 18px; sitewide, any P/LI with own text past ~40
  chars computes >= 16px. Metadata rows (tags/pills/eyebrows/kickers) are a deliberate
  separate tier and exempt.
- `audit:visual` — one ground on /design-system (band backgrounds equal the page ground,
  no exceptions since the 23 Jul DS2 no-wash port), sibling specimen cards render equal
  heights, cover placeholders clear 3:1 against both gradient stops, both themes.
- `audit:debt` — nothing rots quietly: a doc citing a file that does not exist, a token
  nothing consumes through a `var()` chain, a gate table describing audits that no longer
  run, or an audit tracking a selector that matches nothing. Static analysis, about a second.
- `npm test` runs FIRST in the gate: the pure functions in `lib/bella/dtcg.ts` (vitest). A
  broken function fails in a second rather than after two minutes of browser work. Tests are
  not an audit and do not change the derived count.
- tsc clean; all routes 200 (light + dark); NDA content-grep clean.

**No per-element exemptions (Elleta, 2026-07-27, hard rule).** The gate has no opt-out.
If something cannot pass an audit, it does not get to be live DOM. No `data-example`, no
skip attributes, no "ignore this node" hooks, ever. **An audit you can opt out of is not
an audit.** When an illustration genuinely has to show rule-breaking output (a bad
example, an off-brand artifact), it ships as a PICTURE, a flat `<img>` with a describing
`alt`, so there is nothing for an audit to read and therefore nothing to skip. The only
allowlists that may exist are the file-level ones already recorded in the audit scripts,
each with its reason inline; do not extend them to keep new code alive.

**The one allowlist, and why it is not the same hole.** `audit:debt` reads
`scripts/lib/debt-allowlist.json`. That rule above governs RENDERED OUTPUT: if an element
cannot pass, it does not get to be live DOM. `audit:debt` judges CODE INVENTORY, where
"this doc is a dated historical record" is a true fact about intent that no static analysis
can derive. It is capped so it cannot grow into the hole: every entry needs a written reason
and a date, the audit prints the count on every run, it FAILS above 15 entries, and it FAILS
on any entry older than 180 days that has not been re-dated.

## 10. How this file was built and stays alive
Like a real steering doc: when something keeps going wrong, research it, fix it, and record the fix HERE
(or in the paired spec) so it never breaks again. Update this constitution deliberately, not with churn.
When something breaks more than once, record the fix as a file in `docs/fixes/` and reference it here;
keep `docs/fixes/README.md` current. Before debugging a familiar-feeling symptom, check that folder first.

---

# Repo operations (kept from the previous harness file)

## Before doing anything
1. Read the **most recent session record** in `docs/session-*.md` (newest by date). It is the
   backward record: what shipped, what broke, what was learned, and which decisions are still
   open. It is committed, so it survives; `claude-progress.md` is local-only and does not.
   **Start with `docs/session-2026-07-28.md`.**
2. Read `claude-progress.md` — current verified state and last session's forward handoff.
3. Read `feature_list.json` — pick the highest-priority item not yet passing. One item at a time.
4. If the task involves a prototype, open its folder README first (e.g. `prototypes/finviz-3/README.md`).

## Repo-specific working rules
- **Prototypes are single-file.** Each lives in `prototypes/<name>/index.html`, self-contained (inline CSS/JS, no build step). A README maps design decisions to their sources.
- **Never modify `Artifacts/*/versions/`** — those are historical snapshots.
- **Design work needs design verification.** "It renders" isn't done. Done = interactions verified, hooks/copy checked against the relevant brief, WCAG basics considered, README updated.
- **Evidence before passing.** Update `feature_list.json` only with a note on how it was verified. Never delete or reword entries — only change status and evidence.
- **Content drafts** (LinkedIn etc.) belong in Notion's Content Lab, not this repo — except `prototypes/linkedin-preview/`.
- **File locations:** save deliverables into THIS folder — never cloud drives or scratch folders Elleta can't see. NDA-sensitive material goes in `_private/` (gitignored).
- **The pre-commit hook false-positives** the Apple Music album id in `components/VinylPlayer.tsx` as a phone number — that file stays uncommitted (see `docs/fixes/`).

## End of session
- Write or update the **session record** at `docs/session-<YYYY-MM-DD>.md`: what shipped, what
  broke and what it taught, open decisions, standing debt. It is committed and it is what the
  next session reads first. Be honest about what was not verified.
- Update `claude-progress.md`: what was done, how verified, known risks, next best action.
- Leave no half-finished prototype states.
- If git is in use for the change, commit with a descriptive message.

## Key references
- Layout & frame contract: `DESIGN.md` (tokens, ramps, recorded exceptions; audit tooling points here).
- Finviz project: `finviz-event-storming.md` + `finviz-ai-solution-canvas.md`, brief at interface-design-patterns-ux-training.notion.site (Brief #2).
- Voice & content rules: the `linkedin-post` skill (installed in Claude, not this repo).

---
> Source: [emcdanie/ctrl-alt-design](https://github.com/emcdanie/ctrl-alt-design) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
