## personal-fitness

> This repository is a personal performance and rehabilitation system for an adult recreational athlete:

# Personal Fitness — Claude Project Instructions

## Project Purpose
This repository is a personal performance and rehabilitation system for an adult recreational athlete:
- Returning from 14 months of right knee patellar tendinopathy (now 2–3/10 pain)
- Managing postural dysfunction: anterior pelvic tilt, lax TVA, inhibited scapular movement, rib flare
- Dual-sport: ice hockey (skating power, edge confidence) and golf (left-handed, natural fade, 10–13 handicap)
- **Single tool**: `index.html` — execution, logging, knowledge, and review in one file, backed by Airtable

## Session Startup Protocol

**Every new session must begin by confirming or creating a feature branch:**

```bash
git checkout main && git pull   # sync first
git branch --show-current       # confirm not on main
git checkout -b feat/<topic>    # if starting new work
git push -u origin feat/<topic> # establish remote tracking
```

Never commit directly to `main`. All development happens on feature branches.

## Git & PR Rules

- Feature branches: `feat/<short-description>` or `fix/<short-description>`
- Commit messages: imperative present tense, specific ("Add tonight mode breathwork cue" not "Update file")
- **Full test gate before any PR to main:**
  1. HTML validates (no unclosed tags, valid structure)
  2. All JavaScript functions defined before called — IIFE structure must be syntactically valid
  3. Mobile viewport renders correctly (check meta tags, touch targets ≥ 44px)
  4. Dark/light mode both functional
  5. All exercise cards expand/collapse
  6. Progress tracking persists (localStorage check)
  7. CSV export produces valid output
- PRs require explicit user approval — do not create PRs automatically
- **Pause after each phase** and report completion — user manages credit usage

## Project Structure

```
/
├── index.html              # Single-file app (CSS + JS inline, no build step)
├── inputs/
│   ├── plan.txt            # v3 implementation spec (authoritative reference)
│   └── Claude-Hockey and golf player rehabilitation program.md
├── CLAUDE.md               # This file
├── MEMORY.md               # User and project memory
└── .claude/
    └── commands/           # Slash command skills
        ├── hockey.md
        ├── golf.md
        ├── body-movement.md
        └── breathwork.md
```

## Coding Standards (index.html)

- **Single file** — no split files, no build step, all CSS + JS inline. Hard constraint.
- **No frameworks** — vanilla HTML/CSS/JS only
- **Dark mode default**, light mode toggle, WCAG AAA contrast
- **Touch targets ≥ 44px** on all interactive elements
- **Fonts**: Inter for body, JetBrains Mono for data/metrics
- **localStorage** first — always write locally before async Airtable calls
- CSS custom properties for all colors — never hardcode hex in component styles
- JavaScript: IIFE pattern, event delegation via `data-*` attributes, no `onclick=` in HTML
- Accessibility: semantic HTML, aria-labels on icon-only buttons, focus-visible rings
- **Units**: lbs and inches throughout — never metric
- All Airtable errors silent-with-snackbar — app must never crash offline

## JS Architecture (must be maintained)

```
APP_CONFIG       — constants, field options, FIELD_DESCRIPTIONS, PHASE_EXERCISES
StorageService   — localStorage read/write (credentials, draft, theme, metrics, milestones)
AirtableService  — all API calls (create/patch/fetch for SESSIONS, WEEKLY_METRICS, MILESTONES)
SessionMachine   — IDLE → STARTING → ACTIVE → LOGGING → SAVING → COMPLETE
UI               — Nav, Today, Log, Progress, Learn, Settings, Snackbar, BottomSheet,
                   FieldInfo, ResumeBanner, Setup (all sub-objects of UI)
App              — init() entry point, _handleClick(), _syncFromAirtable()
                   ⚠️ All methods must be INSIDE the App object literal
```

**Critical**: `App` is a plain object literal. Every method must end with `,` not `};` or it will
be orphaned outside the object, causing a silent syntax error that breaks all JS.

## Airtable Configuration

- **Base**: fitness-tracker · **Base ID**: app1kMd9ytcLcS09C
- **Token**: stored in localStorage (AT_PAT) — never in code, never in git
- **Required token scopes**: data.records:read, data.records:write, schema.bases:read, schema.bases:write
- **Tables**: SESSIONS, WEEKLY_METRICS, MILESTONES
- **Primary field rule**: First field of each table must be plain `singleLineText` (Name) — Airtable requirement
- **testConnection endpoint**: `GET /meta/bases/{baseId}/tables` (not `/meta/bases/{baseId}`)

## Efficiency Rules (credit management)

- Write, don't read — use a single `Write` or `Edit` call per phase, trust the output
- No pre-read exploration when content is already in context
- One tool call per phase — no incremental edits mid-phase
- Pause and report after each phase — user decides when to continue

## Refinement Model — Feynman Loop + Karpathy Research Cycle

Before making any recommendation, protocol change, or exercise modification:

### Phase 1 — Feynman Clarity Check
```
EXPLAIN: Can I state this in one plain sentence a tired person understands at 10pm?
GAP:     What am I uncertain about? What am I assuming?
DEEPEN:  Address the gaps — what does the domain evidence say?
SIMPLIFY: Return to plain language with gaps filled and assumptions stated.
```

### Phase 2 — Karpathy Research Loop
```
HYPOTHESIS:  What outcome does this approach predict?
SIGNAL:      What would we observe if it's working vs. not?
MEASURE:     Which tracking columns or qualitative signals capture this?
REFINE:      If signal is absent after N sessions, what is the next iteration?
```

Use this when proposing exercise changes, answering mechanics questions, or evaluating new features.
Do NOT use it as a reason to over-hedge — output should be a direct recommendation.

## Domain Context Summary

See MEMORY.md for full user profile. Key constraints for any change:
- Program must work at night, when user is fatigued and stressed
- `Tonight Mode` (3-exercise minimum session) must always be reachable in one tap from Today tab
- Witness/observer framing — exercises described as things to notice, not just perform
- Confidence and energy are the root goals; athletic identity is secondary
- Equipment: resistance bands, step, 10lb kettlebell, some dumbbells
- The left-handed golf fade is stable — never suggest removing it

## Skill Commands Available

| Command | Purpose |
|---|---|
| `/hockey` | Hockey-specific mechanics, skating analysis, on-ice drill design |
| `/golf` | Golf biomechanics for left-handed fade player, drill design |
| `/body-movement` | Movement pattern analysis, kinetic chain, postural correction |
| `/breathwork` | Breathing mechanics, TVA/diaphragm protocols, nervous system regulation |

Invoke these when the task requires deep domain intelligence beyond general fitness knowledge.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/feemurk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
