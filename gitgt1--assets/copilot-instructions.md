## assets

> These are the user's requirements. They outrank the spec file, outrank anything

# Startup Discovery Tool

## THE CONTRACT — read before any work, every session

These are the user's requirements. They outrank the spec file, outrank anything
in this document, and outrank whatever seems technically convenient. If a
requirement and an implementation disagree, the implementation is wrong.

| # | Requirement | Verified by |
|---|---|---|
| C1 | The search universe is **companies that have NOT raised institutional VC**. Proprietary deal flow is the entire point. A company already in a funding database has been priced by someone else. | `tests/test_contract.py::test_c1_universe_is_unfunded` |
| C2 | The deliverable is **2 named startups** to propose to b2venture, each with a reason. Not a platform, not a dataset. | `tests/test_contract.py::test_c2_deliverable_exists` |
| C3 | Lean. The pipeline is a means to C2, never the product. | reviewed, not automated |
| C4 | No fabricated companies, exits, or figures in any output shown to a human. | `tests/test_contract.py::test_c4_no_synthetic_in_deliverable` |

### Pre-flight ritual — mandatory, every session, before writing code

State these three lines back before touching anything. If any cannot be
answered from the contract above, **ask rather than assume**:

1. Which contract item does this work serve?
2. What is the search universe, and does it satisfy C1?
3. What does the user see at the end of this turn?

### The stop rule

**A data source that cannot contain the target universe is disqualified, no
matter how good its data is.** Check this before building an adapter, not after.

Concretely: a funding database indexes rounds. If a company has never raised, it
has no round, so it cannot appear. No amount of filtering recovers it. This is
what went wrong once already — 51,591 rounds ingested and scored before anyone
noticed the target universe was structurally absent from the source. Fitness of
a source is a property to test first, not a caveat to add later.

### Drift check

Any turn that produces no user-visible progress toward C2 must open by saying
so. Three such turns in a row is a stop-and-ask, not a cue to build more.

---

Module 1 (Market Signal Layer) of the tool specced in `SPEC_discovery_tool_v1.md`.
Read the spec before changing metric logic — most of what looks like an arbitrary
choice in the code is a locked decision with a rationale there.

## Commands

    make setup    # venv + deps
    make test     # 61 tests, ~1s
    make thin     # Phase 1 scale (60/sector) — fast, CE on single-digit exits
    make run      # Phase 2 scale (1800/sector) — the meaningful numbers
    make deals    # crowding on deal count instead of capital volume
    make recommend  # ranked startups for b2venture (the actual product)

Real data goes through the CSV source rather than the generator:

    .venv/bin/python -m discovery.cli --source csv \
      --companies data/real/companies.csv \
      --rounds data/real/rounds.csv \
      --aggregates data/real/sector_aggregates.csv

Schema and worked examples in `data/templates/`. Omit `--aggregates` and
Crowding falls back to the generator — the run then stays flagged synthetic,
because a partially-real run must not be presented as a real one.

Output lands in `out/`: `sector_yield.csv` (the module contract),
`sector_yield.json`, and `SECTOR_YIELD.md` (phone-readable).

## Current state

- **Phase 1: done.** Metric engine, source-adapter layer, merge rule, CSV/JSON/MD
  output, tests.
- **Phase 2: in progress.** The pipeline is real; the *data* is still synthetic.
  The primary source is the **Tech.eu Funding Explorer** free API (~51.7k rounds,
  8,743 exits with disclosed value, EUR-normalised at the ECB rate on each round
  date). No application or key needed. There is no Crunchbase dependency.
- **Recommender: thin end-to-end slice works.** Fund profile in, ranked startups
  out, with buckets, wildcards and a rationale per company. Two of seven scoring
  dimensions (team, technology) await a judgement layer; weights renormalise over
  what is actually scored and every row reports `dimensions_scored`.
- No frontend yet. It is the most fun and least load-bearing part.

## The one thing to keep straight

`src/discovery/sources/synthetic.py` generates a **fake** cohort. It exists so
the arithmetic could be built and tested before real data arrives. Every run
stamps `synthetic: true` in the manifest and prints a warning.

No number this repo currently produces is a finding about European venture. When
real data lands, the ranking will change. Do not quote these figures to anyone.

## Architecture

    config.py      locked decisions: cohort window, taxonomy, thresholds
    models.py      Company / FundingRound / ExitEvent / SectorMetrics
    metrics.py     CE, HR, V, C, yield index, labels — deterministic, no LLM
    pipeline.py    merge rule, ranking, output writers
    fund.py        stated thesis in, revealed thesis computed from investor names
    scoring.py     per-company dimensions, buckets, wildcard rule
    recommend.py   the actual deliverable: ranked startups + rationale
    sources/       pluggable adapters; metric layer never knows the tier
      csvfile.py       real data in, strict schema, blank != 0
      synthetic.py     the generator — FAKE, see below
      companies_house.py  Tier 2, needs an API key

Adding a source means implementing `CompanySource` in `sources/` and recording
provenance on each row. Nothing in `metrics.py` should ever need to change to
accommodate a new source.

## Rules that are load-bearing

1. **The CE denominator is disclosed equity only.** Grants and debt are excluded
   (SPEC §4.2). Including a robotics company's €20M Horizon grant would wreck CE
   for exactly the capital-intensive sectors the metric exists to treat fairly.
2. **Missing data is `None`, never `0.0`.** A company that has not exited is not
   a company that exited for nothing. Zeroing drags every median toward zero.
3. **CE is a median, never a mean.** One outsized outcome must not define a sector.
4. **The arithmetic stays in Python.** No LLM computes a score. Same inputs must
   give the same output byte for byte (SPEC §6.1) — there is a test for this.
5. **Score, never filter** (SPEC §2). Applies to Module 3; do not add a cutoff
   that deletes rows.
6. **Absolute CE values are optimistic and always will be** (SPEC §4.6). Only the
   relative ranking between sectors is defensible. Say so in any output surface.
7. **Two floors gate CE, not one.** SPEC §4.4's n>=30 is on companies, but the CE
   median actually rests on the disclosed-exit subset, which is far smaller. Both
   floors must clear before `ce_meets_floor` is true. Failing sectors are flagged,
   never suppressed — suppressing would delete information the reader needs.

---
> Source: [GitGT1/assets](https://github.com/GitGT1/assets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-12 -->
