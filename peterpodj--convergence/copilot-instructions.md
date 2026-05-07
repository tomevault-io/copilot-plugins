## convergence

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This repo **is** a Claude skill (the deliverable), not a project that consumes one. It contains the `convergence-investigation-2.skill/` directory (v2.0 of the methodology, WFGY-enhanced) plus a root `README.md` and `infographic.svg` used as the public landing page.

Convergence Investigation is a general-purpose research methodology that orchestrates six investigative sub-methods under strict epistemic-tier discipline. It produces a standard deliverable bundle — tier-disciplined knowledge graphs, investigation reports, comparison tables, handoff documents, and interactive visualizations. The methodology is project-agnostic: equally effective for cross-domain academic investigation, corporate due-diligence, journalistic source-mapping, institutional-history projects, or any research requiring multiple sources and rigorous synthesis.

There is no build system, no test suite, no package manifest. Edits to the skill (SKILL.md + reference modules) are the work.

## Architecture of the skill (what to understand before editing SKILL.md)

The skill encodes a **4-phase pipeline** (Phase 1: Triage → Phase 2: Parallel Sub-Investigations → Phase 3: Synthesis → Phase 4: Delivery), plus Phase 2.5 (Authoritative-Source Pathway) and Phase 3.5 (optional WFGY enhancements). The pipeline selects from **seven sub-methods**, each backed by a reference module under `convergence-investigation-2.skill/references/`:

1. Structural-parallel documenter — `structural-parallels.md`
2. Bridge-figure mapper — `bridge-figures.md` (the single highest-value finding class; includes role-context tension scoring)
3. Document parser — `document-parsing.md`
4. Corporate-jurisdiction verifier — `corporate-jurisdiction.md`
5. Knowledge-graph version updater — `graph-integration.md` (includes holographic validation protocol)
6. Handoff generator — `handoff-generation.md`
7. Visualization artifact generator — `visualization-artifacts.md`

Plus cross-cutting modules: `epistemic-tiers.md` (the credibility gate, with tension-tier formalism) and `s-class-taxonomy.md` (WFGY 3.0 structural parallel vocabulary).

Load-bearing invariants that edits MUST preserve:

- **Four epistemic tiers** (`confirmed` / `high` / `medium` / excluded) with strict graph-admission rules: only `confirmed` and `high` enter the graph. The audit-gate pseudocode in `epistemic-tiers.md` is referenced elsewhere — if you change tier names, update every reference.
- **Apophenia audit** (5 checks: tier, bridge-figure, chain, counter-evidence, novelty) runs before delivery. This is the credibility firewall; don't weaken it.
- **File discipline** — paths like `/mnt/project/`, `/home/claude/<thread-name>/`, `/mnt/user-data/outputs/` inside SKILL.md describe the **runtime sandbox** where the skill executes, *not* this repo. Don't "fix" them to local paths when editing the skill.
- **No case-study-specific names in documentation.** Examples in reference modules use generic placeholders (Subject A, Institution X, Year 0 / Year 13). Do not reintroduce named real individuals when expanding examples.

## Repo layout

```
README.md                                  # Public landing page (renders with infographic)
infographic.svg                            # Landing-page header graphic
SECURITY.md                                # Security-reporting policy
.github/workflows/                         # Claude Code review / PR workflows
convergence-investigation-2.skill/
├── SKILL.md                               # Methodology specification, YAML frontmatter
├── README.md                              # Skill-level overview + change log
├── references/                            # 9 sub-method / discipline modules
└── scripts/holographic_validation.py      # WFGY Semantic Holography implementation
```

The `convergence-investigation-2.skill/` name keeps the `.skill` suffix so it can be directly linked into your skills directory (e.g., `~/.claude/skills/convergence-investigation` or similar). If renaming or flattening, update the root `README.md` installation section and the link anchors in its Documentation section.

## Operations

No commands to run. There is no test runner; `holographic_validation.py` has an `if __name__ == "__main__"` illustrative example and no test harness — run it directly (`python3 convergence-investigation-2.skill/scripts/holographic_validation.py`) to sanity-check changes to the holographic metrics.

## Editing conventions

- SKILL.md frontmatter fields (`name`, `description`, `version`) are referenced by the Claude Code skill loader — preserve their structure. `description` is the trigger text the model sees; edits to it change when the skill fires.
- Reference modules are written in close-packed prose with inline code blocks for JSON schemas and Python snippets. Keep the tone; these are read by the model at runtime and verbosity costs tokens.
- When adding a new sub-method or reference module, update: (1) the "Seven Sub-Methods" table in SKILL.md, (2) the "Reference modules" list at the bottom of SKILL.md, (3) the root `README.md` Documentation section.
- Examples and "Validated Example" tables in reference modules must use generic placeholders. Never paste real named individuals, specific dates tied to identifiable incidents, or recognisable institution names from live investigations into this repo.

---
> Source: [peterpodj/convergence](https://github.com/peterpodj/convergence) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-04-24 -->
