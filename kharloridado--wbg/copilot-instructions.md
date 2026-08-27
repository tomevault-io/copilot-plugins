## wbg

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## CLAUDE.md — Project Rules

> Copy this whole folder for each new project, then fill in the placeholders below. This file tells Claude how to work on **this** project. It pairs with `project-context.md` (the values) and the installed OutSystems skills (the behavior).

## Project

- **Customer:** `World Bank Group`
- **Project:** `WBG`
- **Environment:** ODC (default) — change if O11
- **Class prefix:** `loop-`
- **Spacing base:** unconfirmed — grid spec TBD (do **not** flag values as "off the 4pt grid"). Scale extracted from Figma (`tokens/spacing.css`, descriptive `tiny`…`xhuge`); most steps are multiples of 4 but this is not a confirmed rule. See the 2026-06-16 Live Style Guide sync ("final grid spec not yet confirmed").
- **Token style:** OutSystems UI standard conventions
- **CSS methodology:** BEM (`block__element--modifier`)
- **Custom components:** vanilla JS Web Components wrapped in OutSystems Blocks
- **Accessibility target:** WCAG 2.2 AA — **fidelity-first**

## The one rule that matters most here

**Build the design exactly as specified. Never silently change a brand color, value, or token to satisfy accessibility or to "tidy" the design.** When the design conflicts with accessibility, brand, or token rules, implement it faithfully and raise a **finding** (see `findings/`). The finding carries the recommendation back to design; the code stays true to the mockup until design responds or the brand owner signs off.

Implementation-level accessibility that does NOT change the visual design — focus rings in the design's own colors, keyboard handlers, ARIA, semantic HTML, reduced-motion, labels — is applied automatically.

## Findings routing (used by the `outsystems-design-findings` skill)

```
findings.ticketing     = github            # default; alternatives: notion | jira
findings.ticket_target = <owner/repo>       # + optional GitHub Project name
findings.slack_channel = <#channel | none>  # GitHub Slack app or Slack connector
findings.gate          = high+              # high+ opens issues + notifies; medium/low batch to the register
```

Findings become **GitHub Issues filed as Bugs** (Bug issue type + `bug` label) in the project repo, created via `gh` from Claude Code (works on the private repo, no MCP). Labels: `finding` + `bug` + type (`a11y`/`brand`/`token`/`consistency`) + `sev:*`. A structured issue form lives at `.github/ISSUE_TEMPLATE/finding.yml`. The local register mirror is `findings/findings-register.md`; payloads are written to `findings/tickets/`.

## Code handover (this dev works mainly in OutSystems)

Generated code (CSS, Web Component `.js`, Block instructions) is **not** the end of the chat — it is handed over as a **GitHub issue filed as a Task and assigned to the developer**, so they add it into OutSystems themselves. Label `handover` + `task`; form at `.github/ISSUE_TEMPLATE/handover.yml`; example body in `handover/`.

**Rule: the handover ticket must CONTAIN the JS/CSS to copy into ODC — not just point at a repo path.** Every `handover/*.md` carries a `## Code to paste into ODC` section with the verbatim artifact(s) in a collapsed `<details>` block (source path in the `<summary>`). Tokens travel via `dist/tokens.css` (its own paste) so they aren't duplicated there. Embed only what the developer hand-places: block CSS overrides and Web Component `.js`. The blocks are generated from source by `node build/embed-handover-code.mjs` (idempotent) — re-run it after editing a source file, and add new handovers to its `MAP`.

**Rule: every handover also carries a `## Build in ODC with Mentor Studio` section** — a ready-to-paste prompt for **ODC Mentor Studio** that scaffolds the OutSystems side (Block, attribute bindings, event wiring, Client Actions). Mentor is a logic/data agent — it does *not* author the CSS or the Web Component, so the prompt fences those off as already-pasted and aims Mentor only at the wiring. The same `embed-handover-code.mjs` generates it: archetype-aware by default (Web Component / native-widget restyle / Style-Guide reference), or fully filled when the `MAP` entry supplies a `mentor` spec (see `loop-toast.md` for the worked example). The reusable template lives at `handover/MENTOR-STUDIO-PROMPT.md`.

```bash
gh issue create --title "[handover] <component> — add in OutSystems" \
  --body-file handover/<artifact>.md --label "handover,task" --type "Task" \
  --assignee @me --repo <owner/repo>
```

Both findings (bugs) and handovers (tasks) can live on the same GitHub **Project** board — a kanban view with a `Status` column you drag items across.

## Build pipeline

- Source tokens live in `tokens/` (`colors.css`, `spacing.css`, `typography.css`).
- `npm run build:theme` → **two ODC pastes** (the 2026-07-16 token split):
  **`dist/tokens.css`** — the design tokens ONLY (single consolidated `:root` +
  device-scoped token redefinitions like `body.phone` type steps) — and
  **`dist/theme.css`** — everything else (`@font-face`, base rules, utility
  classes, widget/component overrides; NO tokens). Both pasted into the ODC
  Theme editor; token edits only require re-pasting `dist/tokens.css`.
  Assembled by `build/build-theme.mjs` (comment-preserving), which prepends an
  **OutSystems-UI-style Table of Contents + section banners** to EACH file and
  keeps the source provenance/finding comments. **Rule: both dist files must
  always carry this TOC + sectioning** — never ship a flat, comment-stripped
  theme. Section order follows the `@import` order in `tokens/index.css`; add a
  file's title to the `META` map in the build script when you add a new token file.
- **Token change report (every build):** the assembled token set is diffed
  against the committed baseline `tokens/tokens.lock.json`; added/modified/removed
  tokens are printed classified **branding** (colors, semantic roles, OSUI brand
  retints) / **foundation** (spacing, typography, radius, border, shadows) /
  **component** (`component-*.css` + block `:root` vars), and recorded newest-first
  in `tokens/TOKEN-CHANGELOG.md`. Both files are tracked and generated — commit
  them with the token change; never edit them by hand.
- `npm run build:theme:ship` → the **customer deliverable**: same two dist files
  but with the ordinary `/* … */` provenance/finding notes
  stripped, keeping the `/*!` **head, Section Index, and section banners** (so it
  still satisfies the TOC + sectioning rule above — it is not a flat theme). Verified
  byte-identical to the commented build after minification (strip touches only
  comments/whitespace). Re-run `build:theme` to restore the commented dev copy.
- `npm run watch:theme` for live rebuilds while iterating.
- `npm run build:theme:min` → `dist/theme.min.css` via lightningcss (optional
  minified build; strips comments, so NOT the file pasted into ODC).

**Icon-font pipeline (separate subsystem).** FontAwesome 6 Pro is **self-hosted**
from the desktop package, not the CDN. The chain is `convert:fa-otf` (OTF→woff2,
one-time per FA version) → `gen:fa-css` (rebuild `all.css` from `core-template.css`
+ `metadata/icons.json`) → `build:fontawesome` → `dist/fontawesome.css` to paste
into ODC. The declared family is **`'Font Awesome 6 Pro'`**; the legacy
`'FontAwesome'` family is **deliberately never declared** — that name belongs to
OSUI's native Icon widget and redeclaring it clobbers the widget. `gen:icon-data`
emits `src/components/loop-icon-data.js` (`window.LoopIconData`) powering the Style
Guide's icon reference. FA Pro assets are **licensed — do not redistribute**; pin 6.x.
**IcoMoon export/maintenance branch (optional, off the same source).** `gen:icomoon`
emits one `dist/icomoon/loop-fa-<style>.selection.json` per style (solid/regular/light)
for re-import at icomoon.io — **FA codepoints preserved**, canvas `height:512` with
per-glyph `width`s so paths are used **verbatim** (lossless — never scaled, which would
corrupt the ~38% of glyphs with arc commands). `gen:icomoon-data` then merges those
selection.json back into `dist/icomoon/loop-icomoon-data.js` (`window.LoopIcoMoonData`,
with paths) so `<loop-icon-reference>` can render glyphs as **inline SVG** (preview
fetches the JSON via its `src` attr; ODC loads the global). Because these carry the
licensed vector artwork, **all IcoMoon outputs stay in gitignored `dist/`** (like
`metadata/icons.json`) — never committed.

### All commands

| Command | What it does |
| --- | --- |
| `npm install` | Install build deps (lightningcss-cli, sass). |
| `npm run build:theme` | Assemble `tokens/*.css` → `dist/tokens.css` (design tokens only, single `:root`) + `dist/theme.css` (classes/overrides, no tokens), both commented + TOC'd. Paste BOTH into ODC. Also diffs tokens vs `tokens/tokens.lock.json` and logs branding/foundation/component changes to `tokens/TOKEN-CHANGELOG.md`. |
| `npm run watch:theme` | Same, rebuilding on token changes. |
| `npm run build:theme:ship` | Customer deliverable: same two dist files with ordinary comments stripped, `/*!` TOC + section banners kept. Paste both into ODC. |
| `npm run build:theme:min` | Minified `dist/theme.min.css` (not for ODC paste). |
| `npm run check:live-theme` | Diff the LIVE ODC theme CSS against a fresh local build (normalizes ODC minification + url fingerprints). Exit 0 in sync / 1 drift / 2 live unreachable. Run daily by the "live-theme drift check" cloud Routine (`loop/ROUTINES.md` §4), which files a `theme-drift` issue on drift. |
| `npm run gen:color-utilities` | Generate `.background-*` / `.text-*` utility classes (`tokens/color-utilities*.css`). |
| `npm run gen:type-utilities` | Generate `.font-size-*` / `.font-weight-*` classes (`tokens/typography-utilities.css`). |
| `npm run gen:spacing-utilities` | Generate directional margin/padding classes (`tokens/spacing-utilities.css`). |
| `npm run gen:icon-data` | Emit `src/components/loop-icon-data.js` (`window.LoopIconData`) from the vendored FA manifest for the Style Guide icon reference. |
| `npm run build:fontawesome` | Adapt vendored FA6 Pro CSS + woff2 into self-hosted ODC assets (`dist/fontawesome.css`); drops legacy v4/v5 `@font-face` so it won't clobber OSUI's native icon widget. |
| `npm run gen:fa-css` | Reconstruct `vendor/fontawesome-6/css/all.css` from `core-template.css` + FA `metadata/icons.json`. |
| `npm run gen:icomoon` | Emit IcoMoon-importable `dist/icomoon/loop-fa-{solid,regular,light}.selection.json` (one per style, FA codepoints preserved, lossless height-512 paths) so the icon font can be re-imported/maintained at icomoon.io. Requires the gitignored `metadata/icons.json` (same prereq as `gen:fa-css`). `--minify` for compact output. |
| `npm run gen:icomoon-data` | Merge the `dist/icomoon/*.selection.json` into `dist/icomoon/loop-icomoon-data.js` (`window.LoopIcoMoonData`, with paths) for `<loop-icon-reference>`'s inline-SVG rendering in ODC. Reads the selection.json (not `icons.json`) so an IcoMoon re-export round-trips. `--lean` omits paths. |
| `npm run convert:fa-otf` | One-time-per-version: convert FA6 Pro desktop OTFs → woff2 via `wawoff2`. |
| `npm run build:osui` | Compile the vendored OutSystems UI submodule → `preview/vendor/outsystems-ui/outsystems-ui.css` (the preview's real OSUI base). |
| `npm run preview` | Zero-dep static server (port 8088) serving `preview/index.html` over http:// for the local component preview. |
| `node build/embed-handover-code.mjs` | Idempotently embed source CSS/JS into the `handover/*.md` "Code to paste into ODC" blocks. Re-run after editing a handed-over source file. |

The `gen:*` outputs are GENERATED — edit the generator in `build/`, not the emitted `tokens/*-utilities.css`. There is no separate test/lint step; the **`@checker` agent** is the validation gate (see below).

## Hard rules (inherited from figma-to-outsystems orchestrator)

1. Never edit the OutSystems UI module directly.
2. Never validate in Service Studio Preview alone — publish and test in a real browser.
3. Never hard-code design values — always `var(--token)`. If a value has no token, that's a `design-token` finding.
4. Never silently substitute a brand color/value/token for accessibility — flag it.
5. Never drop a finding to avoid friction — log it at minimum.
6. For custom components: vanilla JS Web Components only (no Lit/Stencil/React).
7. Never attach classes by mutating OutSystems UI internals — use `ExtendedClass`.

## Workflow

Hybrid: Claude.ai for design analysis + generation, Claude Code for Git operations on the private repo.

## Architecture map

The repo turns Figma designs into three OutSystems-pasteable artifacts: **theme tokens**, **block CSS overrides**, and **Web Component JS**. How they fit together:

**Token layering (`tokens/`).** `tokens/index.css` is the single `@import` manifest and the order is load-bearing:
primitives (`colors`, `spacing`, `typography`, `radius`, `border`, `shadows`) → semantic role layer (`semantic-colors.css`) → generated utility classes → per-component token files (`component-*.css`) → **OutSystems UI overrides LAST** (`outsystems-ui-overrides.css`, `-header`, `-alert`, `-feedback-message`) so their `:root` redefinitions win over the framework defaults. `build/build-theme.mjs` lifts every file's `:root` into ONE consolidated block in `dist/tokens.css` (classes/overrides land in `dist/theme.css`), keeps comments, prepends an OutSystems-UI-style TOC to each output, and diffs the token set against `tokens/tokens.lock.json` — every build reports added/modified/removed tokens classified branding/foundation/component into `tokens/TOKEN-CHANGELOG.md`. When you add a token file: add it to `index.css` AND to the `META` map in `build-theme.mjs`.

**`src/` — two delivery shapes:**
- `src/blocks/*.css` — BEM `ExtendedClass` overrides that **restyle native OutSystems UI widgets** (button, dropdown, switch, text-field…). Prefer overriding native `.btn`/`.dropdown`/etc. classes over building parallel systems. These are handed over as paste-in CSS.
- `src/components/*.js` (+ matching `.css`) — vanilla JS **Web Components** for L5 builds that don't exist in OSUI (`loop-alert`, `loop-modal`, `loop-toast`, `loop-system-alert`, plus the Live Style Guide reference components `loop-*-reference.js`). No Lit/Stencil/React.

**Local preview (`preview/`).** `preview/index.html` is the Live Style Guide harness. Layer 1 is the **real** compiled OSUI CSS (`npm run build:osui`), layer 2 is the two theme pastes (`dist/tokens.css` then `dist/theme.css`), layer 3 is the `src/` overrides + Web Components. Chrome must stay token-only and class-only (no inline styles / ad-hoc hex). Serve with `npm run preview`, then validate in a real browser — never trust Service Studio Preview for Web Components.

**The design loop (`.claude/` + `loop/`).** `/design-loop` (or `loop/run.sh`) drives an autonomous Figma→OutSystems loop defined by `loop/goal.md`. Per component: **`@maker`** builds one artifact faithfully → **`@checker`** independently validates (fidelity, token-only, BEM, Web Component correctness, accessibility flag-don't-fix) and returns PASS/FAIL. On PASS it commits, opens a handover Task, and updates the Style Guide. State is resumable via `loop/state.json`; `loop/REPORT.md` summarizes the run. The **spec of record** is the frozen Figma snapshot at `loop/refs/<item-id>/` (`spec.md` + `variables.json` + `figma.png`) — the orchestrator snapshots it via Figma MCP **before** the maker runs, because subagents have no Figma access; both maker and checker judge against the ref, never live Figma, and **no ref ⇒ the item goes `needs-human`, never built**.

**The agentic-review gate (lean, single checker).** The `@checker` is the project's code review. It runs in this order — detail lives in `.claude/agents/checker.md`:
1. **Deterministic gate (hard wall):** `npm run build:theme` must exit 0 + schema/contrast resolve, *before* any subjective judgment. A broken build is an instant FAIL.
2. **Risk-tiered depth:** scrutiny scales to blast radius — `trivial` (utility/config) gets a glance, `core` (L5 Web Components, interactive composites) gets the full stack. Not a uniform gate.
3. **Adversarial finding challenge:** every finding (raised or suspected) must survive a refutation against *real rendered usage* before it's filed (the FND-011 pattern). Refuted ones are `not-reproduced` (register-only, never a bug). Never flag "off the 4pt grid" — the spacing base is TBD (the FND-005/013/018/022 false-positive class).
4. **Decision-log capture:** `@maker` and `@checker` emit their reasoning / alternatives ruled out / assumptions; these persist to `state.json` and the handover so the human reviewer isn't reconstructing intent.
5. **Review metrics:** `loop/REPORT.md` carries a `## Review metrics` block each run (auto-pass vs needs-human, findings filed vs challenged-out, rounds, tier coverage, det-gate pass rate).

**Two GitHub outputs.** Design conflicts become **findings** (Bug issues; mirrored in `findings/findings-register.md`, payloads in `findings/tickets/`). Generated code becomes **handovers** (Task issues; bodies in `handover/*.md` with the verbatim code embedded by `build/embed-handover-code.mjs`). Both routed via `gh` per the routing block above.

**`vendor/outsystems-ui/`** is the OSUI git submodule — the source of truth for real rendered widget DOM/SCSS when designing an override. Read it; never edit it. **`vendor/fontawesome-6/`** holds the licensed FA6 Pro assets (see the icon-font pipeline above).

**Other directories & root docs.** `outsystems-widgets-reference/` — captured real rendered widget HTML to anchor a restyle on (vendored SCSS can be stale). `style-guide/` — Live Style Guide page sources. `design/` and `docs/` (incl. `docs/meetings/`) — design and meeting notes. `preview/vendor/` also vendors providers (Flatpickr, virtual-select). Root docs worth knowing: `GETTING-STARTED.md` (setup), `RELEASING.md` (release steps), `deliverables.md` (the per-deliverable source of truth mirrored on the GitHub Project board), `project-context.md` (customer/project values referenced at the top of this file), and `CHANGELOG.md`.

**No test/lint tooling** exists (no eslint/prettier/jest/tsconfig) — the deterministic build gate (`npm run build:theme` exits 0) plus the `@checker` agent are the validation gate by design.

---
> Source: [kharloridado/wbg](https://github.com/kharloridado/wbg) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-27 -->
