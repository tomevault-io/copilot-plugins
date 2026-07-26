## claude-immigration-attorney

> Skills and knowledge base for immigration petition work. For attorneys and self-petitioning candidates. Covers O-1A, EB-1A, and EB-2 NIW.

# Claude Immigration Attorney

Skills and knowledge base for immigration petition work. For attorneys and self-petitioning candidates. Covers O-1A, EB-1A, and EB-2 NIW.

## Knowledge Base — READ THIS FIRST

**Before drafting any petition, read the relevant knowledge files.** They contain best practices, evidence hierarchies, and argument patterns distilled from real cases.

```
knowledge/
├── overview-o1a-eb1a.md       # O-1A/EB-1A standard, Kazarian two-step, adjudicator behavior
├── overview-niw.md            # NIW standard, Dhanasar three-prong, letter structure
├── criteria/                  # O-1A/EB-1A criteria (used by o1-petition-narrative)
│   ├── 01-awards.md
│   ├── 02-membership.md
│   ├── 03-published-material.md
│   ├── 04-original-contributions.md
│   ├── 05-scholarly-articles.md
│   ├── 06-critical-employment.md
│   ├── 07-high-remuneration.md
│   └── 08-judging.md
├── prongs/                    # NIW Dhanasar prongs (used by niw-petition-narrative)
│   ├── 01-substantial-merit.md
│   ├── 02-well-positioned.md
│   └── 03-national-interest-balance.md
├── evidence-hierarchy.md      # Universal evidence weighting (Tier 1-4)
├── argument-patterns.md       # Reusable argument structures
└── uscis-policy-alerts.md     # Key policy citations (PA-2022-03, Kazarian, Dhanasar)
```

**When to read what:**
- Optional pre-draft strength check (`case-strength-assessor`, if used) → read `evidence-hierarchy.md` + relevant overview + each `criteria/` or `prongs/` file you rate + `uscis-policy-alerts.md` as needed
- Optional publication table (`publication-citation-table`, if used) → read `criteria/05-scholarly-articles.md`, skim `criteria/03-published-material.md`, `evidence-hierarchy.md`, `prongs/02-well-positioned.md` as needed
- Drafting O-1A/EB-1A → read `overview-o1a-eb1a.md` + all relevant `criteria/` files + `argument-patterns.md`
- Drafting NIW → read `overview-niw.md` + all `prongs/` files + `argument-patterns.md`
- Any petition → read `evidence-hierarchy.md` + `uscis-policy-alerts.md`

## Skills

| Skill | What It Does | Reads Knowledge |
|-------|-------------|-----------------|
| `document-summary-arrangement` | Indexes source documents, triages by importance, classifies by type | — |
| `case-strength-assessor` | **Optional.** Pre-draft assessment from the document index — rates criteria/NIW prongs STRONG–INSUFFICIENT when the team wants that pass | `criteria/` or `prongs/`, `evidence-hierarchy.md`, overviews |
| `publication-citation-table` | **Optional.** Scannable summary + full TSV, C3 side list, lead C5 picks — C5 vs C3, dedupe, `NEEDS VERIFICATION` | `criteria/05-scholarly-articles.md`, `criteria/03-published-material.md`, `evidence-hierarchy.md`, `prongs/02-well-positioned.md` |
| `o1-petition-narrative` | Drafts O-1A/O-1B support letters with evidence enrichment | `criteria/`, `overview-o1a-eb1a.md` |
| `eb1a-petition-narrative` | Drafts EB-1A green card petitions — stricter than O-1A, adds sustained acclaim + totality argument | `criteria/`, `overview-o1a-eb1a.md` |
| `niw-national-importance-research` | Researches gov sources, national plans, federal funding for the NIW national importance argument | `prongs/01-substantial-merit.md`, `overview-niw.md` |
| `niw-petition-narrative` | Drafts EB-2 NIW self-petition letters (Dhanasar framework) | `prongs/`, `overview-niw.md` |
| `petition-audit` | Audits any petition letter — verifies every claim, flags errors, produces severity-rated report | All relevant knowledge files |
| `expert-letter-drafter` | Drafts expert/reference/recommendation letters — assigns criteria to experts, ensures no overlap | `criteria/`, `prongs/`, `evidence-hierarchy.md` |
| `rfe-response-drafter` | Analyzes USCIS RFE, diagnoses original petition weaknesses, drafts point-by-point response with new evidence | All relevant knowledge files |

## Workflow

```
Source documents → document-summary-arrangement → document index
                                                       ↓
              knowledge/ (read first) → o1-petition-narrative            → O-1A petition package  ─┐
                                      → eb1a-petition-narrative          → EB-1A petition package ─┤→ petition-audit → audit report
                                      → niw-national-importance-research → research document       │
                                        → niw-petition-narrative         → NIW petition package  ──┘
```

**Optional (not required):** `case-strength-assessor` on the document index — pre-draft strength report, if the team wants it. **`publication-citation-table`** — summary + TSV for publication-heavy cases, if the team wants it.

## Planned Skills

- **Case Chronology** — Builds a timeline from the document index
- **Contract Review** — Clause-by-clause analysis with risk flags

## Key Rules

- **Every claim backed by evidence** — document exhibit or verified web source, no exceptions
- **No fabrication** — if it's not in the documents, don't invent it
- **No client PII in knowledge/** — all case lessons must be anonymized
- **Lawyer decides** — skills produce drafts for attorney review, not legal advice

## workspace/

Gitignored. Client work goes here: `workspace/<matter-name>/`. Never commit client data.

---
> Source: [juntoku9/claude_immigration_attorney](https://github.com/juntoku9/claude_immigration_attorney) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-26 -->
