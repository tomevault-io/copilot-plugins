## access-to-housing

> Access to Housing is a **knowledge-based PropTech intelligence platform** delivered as a Claude Custom Skill (`.skill` file). It is not a traditional code project — there is no runtime, no build step, no tests to run. All files are Markdown documentation that define analytical frameworks, scoring rubrics, data source references, and output templates.

# CLAUDE.md — Developer Context for Access to Housing

## What This Project Is

Access to Housing is a **knowledge-based PropTech intelligence platform** delivered as a Claude Custom Skill (`.skill` file). It is not a traditional code project — there is no runtime, no build step, no tests to run. All files are Markdown documentation that define analytical frameworks, scoring rubrics, data source references, and output templates.

**Version**: 31.0
**License**: MIT
**Author**: Doug Devitre (CoTrackPro)

## Repository Structure

```
SKILL.md                          ← Core skill definition (7 modules + Community Trust pod)
FAIR-HOUSING.md                   ← Compliance framework — READ FIRST before any edits
CONTRIBUTING.md                   ← Contribution guidelines
CHANGELOG.md                      ← Version history
assets/
  data-sources.md                 ← 85+ authoritative data sources with links
  api-reference.md                ← Developer API guide (auth, endpoints, rate limits)
  output-schemas.md               ← JSON schemas for all scoring module outputs
  glossary.md                     ← Plain-language term definitions
references/
  market-intelligence/pod.md      ← 16 modules
  investment-deal/pod.md          ← 16 modules
  risk-climate/pod.md             ← 6 modules
  property-intelligence/pod.md    ← 10 modules
  brokerage-ops/pod.md            ← 18 modules
  brokerage-strategy/pod.md       ← 10 modules
  community-trust/pod.md          ← 6 modules
.github/ISSUE_TEMPLATE/           ← 4 structured issue templates
```

## Key Architectural Decisions

1. **Markdown-only**: No code dependencies. The entire platform is documentation that instructs Claude how to analyze housing data. This keeps it accessible, versionable, and forkable.

2. **Fair Housing compliance is architectural**: Every module, scoring rubric, and output template is designed to avoid protected class characteristics and proxies. `FAIR-HOUSING.md` is not optional — it's foundational. Any edit must pass the compliance checklist.

3. **Modules are composable**: Core modules (1–7) feed into each other (detection → forecast → scoring → opportunity). The Community Trust pod runs alongside core modules, not instead of them. See the Cross-Module Reference Guide in `SKILL.md`.

4. **Output structure is standardized**: Every module output follows: Signal Summary → Key Findings → Detailed Analysis → Data Caveats → Next Steps. JSON schemas in `assets/output-schemas.md` define the structured form.

5. **Data sources are cited with vintages**: No invented data. Every data point must name its source and when it was published. This is a trust signal, not just good practice.

## Working on This Repo

### Before Making Changes
- Read `FAIR-HOUSING.md` — understand what's prohibited
- Read `CONTRIBUTING.md` — follow the style guidelines
- Check `CHANGELOG.md` — understand what changed in v31

### Common Tasks

**Adding a new module to an existing pod**:
1. Add the module to the pod's `pod.md` file following the existing format
2. Add trigger phrases to the Module Index table at the top
3. Update the module count in `README.md`, `SKILL.md`, and `CONTRIBUTING.md`
4. If it's a scoring module, add a JSON schema to `assets/output-schemas.md`
5. Run the Fair Housing compliance checklist

**Adding a new data source**:
1. Add to `assets/data-sources.md` in the appropriate category
2. If it has an API, add to `assets/api-reference.md`
3. Note which modules it supports
4. Update the source count in README if crossing a round number

**Updating a scoring framework**:
1. Edit the scoring rubric in both the pod file and `SKILL.md`
2. Update the JSON schema in `assets/output-schemas.md`
3. Verify the change doesn't introduce Fair Housing concerns

### File Relationships

```
SKILL.md (master skill definition)
  ├── references to → references/*/pod.md (detailed module docs)
  ├── references to → assets/data-sources.md (source citations)
  ├── governed by → FAIR-HOUSING.md (compliance rules)
  └── structured by → assets/output-schemas.md (JSON schemas)

README.md (public documentation)
  ├── mirrors counts from → SKILL.md
  ├── mirrors structure from → references/
  └── links to → all assets/ files
```

When changing module counts, scoring weights, or pod structure, update **all** files in the chain.

## Numbers to Keep in Sync

These values appear in multiple files and must match:

| Value | Current | Files |
|-------|:-------:|-------|
| Total modules | 80 | README.md, SKILL.md, CONTRIBUTING.md |
| Reference pods | 7 | README.md, SKILL.md, CONTRIBUTING.md |
| Data sources | 85+ | README.md, SKILL.md |
| SKILL version | 31.0 | SKILL.md, CHANGELOG.md |

## Conventions

- **Mermaid diagrams**: Use `flowchart` and `pie` types only — these are the only types that render reliably on GitHub
- **Scoring**: All scores use integer scales. Dimension scores sum to the total (no weighting multipliers)
- **Rates**: Always density-normalized (per 1K households, per capita) — never raw counts
- **Trends**: Use 3-year windows. Direction: up/down/flat for tables, Improving/Stable/Declining for prose
- **Fair Housing language**: "objective infrastructure metrics" not "neighborhood quality"; "density-normalized rates" not "crime statistics"; "proximity and access" not "school quality"

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dougdevitre) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-11 -->
