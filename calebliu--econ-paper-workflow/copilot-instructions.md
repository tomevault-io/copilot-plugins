## econ-paper-workflow

> > This file is auto-read by **OpenAI Codex** (project root, `~/.codex/AGENTS.md`) and by other agentic tools that follow the AGENTS.md convention. It mirrors the behavior of the Claude Code slash command `/论文workflow`.

# Empirical Economics Paper Workflow — Agent Instructions

> This file is auto-read by **OpenAI Codex** (project root, `~/.codex/AGENTS.md`) and by other agentic tools that follow the AGENTS.md convention. It mirrors the behavior of the Claude Code slash command `/论文workflow`.

## What this is

A 9-phase workflow for writing empirical economics papers (UK MSc / PhD / journal-submission level) using public data + R + DID/IV/panel-data identification. Validated on two real papers in `examples/case_studies.md`.

## When to activate this workflow

Trigger this workflow when the user:

- Says they're starting a new economics paper, dissertation, or empirical project
- Mentions an econ paper task with a deadline and word-count target
- Asks for `/论文workflow`, "paper workflow", "empirical paper", or similar
- Describes wanting to replicate or extend a known econ paper (Hjort-Poulsen, BHM, DJO, etc.)

When triggered, **do not improvise**. Follow the 9 phases below in order. Confirm context first (Phase 0) before any work.

## Phase 0 — Context confirmation (MANDATORY before starting)

Ask the user these 7 questions verbatim. Do not assume answers:

```
1. 论文类型：课程论文 / 毕业论文 / 期刊投稿？
   (Paper type: course paper / dissertation / journal submission?)
2. 字数范围 + 截止日期？
   (Word-count range + deadline?)
3. 主题状态：
   (a) 从零选题（我给你 5 个候选）
   (b) 已定主题（你告诉我）
   (c) 续写已有项目（在哪个目录？）
   (Topic status: from-zero / pre-defined / continue existing project?)
4. 引用格式：Harvard Manchester / APA 7 / AER 传统 / 其他？
   (Reference style?)
5. 计量方法：你课程教过哪些（DID / IV / RDD / FE / 其他）？
   (Identification methods you've been taught?)
6. 数据约束：必须公开数据？还是可以用机构访问？
   (Data constraint: public-only or institutional access?)
7. 工具栈：R / Stata / Python？
   (Tool stack?)
```

If the user says "按之前的" / "same as last time" / "use defaults", apply:

- Reference style: **Harvard Manchester**
- Tool stack: **R + public data**
- Project location: `~/Desktop/econ-papers/<paper-name>/`
- Default identification: **TWFE event study + Callaway-Sant'Anna robust**

## Phase 1 — Topic + econometric setting lock-in (~30 min)

If user requests from-zero brainstorming, generate **5 candidate topics**. Each must satisfy:

1. Data downloadable in ≤1 day
2. Identification doable in ≤3 days
3. Has anchor paper (top-5 econ journal preferred)
4. Submission-target-tractable scope
5. R/Stata packages mature and well-documented

Output a comparison table: `(Topic | Outcome | Data sources | Identification | Anchor paper | Difficulty)`. Then ask user to lock one.

After lock-in, write `docs/WORKFLOW.md` (per-project) with:
- Daily-milestone calendar to deadline
- Econometric specification (estimator + FE structure + clustering + heterogeneity dimensions)

## Phase 2 — Literature review (~1.5 h)

1. Download anchor paper PDF + 1 critique paper PDF to `Notes/`
2. Run `pdftotext -layout` and `grep` for appendix tables (you'll need the exact parameter values later)
3. Identify 5+ closely related papers via Google Scholar / SSRN
4. Write `Notes/lit_review.md` with: anchor's contribution → gap → your contribution → table of related work

## Phase 3 — Data acquisition (~1 h, NOT 24-48h registration loops)

**Order of attempts:**

1. Public direct links (no registration). Try R-package hardcoded URLs first (e.g. `sboysel/afrobarometer` had live URLs that bypass the registration wall).
2. Hardcoded parameter tables from anchor paper appendix (write these into `Code/00_setup.R` as constants).
3. Government / IGO portals (World Bank `WDI` package, FRED, Eurostat, IMF).
4. Only if all above fail: institutional access (Manchester library WRDS, ICPSR).

Document **every** source URL + access date in `Notes/DATA_ACQUISITION.md`.

## Phase 4 — R analysis pipeline (~2.5 h)

Create exactly 6 scripts in `Code/`:

| File | Purpose |
|------|---------|
| `00_setup.R` | Library loads, constants, hardcoded anchor parameters |
| `01_clean_<source>.R` | Encoding-safe ingest (try `latin1`/`windows-1252` fallback via `tryCatch`); panel construction |
| `02_compute_distance.R` | Spatial joins (use `st_make_valid()` + `sf_use_s2(FALSE)` for invalid GADM polygons) |
| `03_main_regression.R` | Multi-spec static + event study with `fixest::feols`/`feglm` |
| `04_robustness.R` | Callaway-Sant'Anna (`did::att_gt`) + 300-permutation placebo + heterogeneity (gender/urban/age) |
| `05_tables_figures.R` | `modelsummary::msummary` → `Results/Tables/`; `ggplot2` event-study plots → `Results/Figures/` |

**Preferred-spec selection: NEVER use max |t|**. Pick by ex ante criterion (sample median, pre-registered in WORKFLOW.md, or theoretical motivation). Document the choice in §4 of the paper.

## Phase 5 — First draft (~2 h)

Write Markdown source with YAML frontmatter for Pandoc:

```yaml
---
title: "..."
author: "..."
date: "..."
output:
  word_document:
    reference_docx: ref.docx
bibliography: refs.bib
csl: harvard-manchester.csl
---
```

Structure: **IMRAD + Lit Review** (`Abstract → §1 Intro → §2 Lit → §3 Data → §4 Method → §5 Results → §6 Discussion → References → Appendix`).

**Pitfalls to avoid:**
- Don't write LaTeX directly — use Markdown + Pandoc
- Don't double-number sections (either strip manual numbers OR drop `--number-sections`, never both)
- Don't fabricate numbers — every number in the paper must come from `Results/Tables/*.csv`

Compile: `pandoc full_paper_v1.md -o paper_v1.docx --reference-doc=ref.docx --citeproc --csl=harvard-manchester.csl`

## Phase 6 — Multi-layer review (~1 h)

Three **independent** passes (do not merge):

1. **Content review** — find OVB candidates, identification threats, missing mechanism discussion. Use `claesbackman/AI-research-feedback` 2-agent protocol if available.
2. **Format review** — missing citations, hyphenation inconsistencies, double-numbered sections, broken figure refs. Use `oceangis/skill_academic-writing-skills` checklist.
3. **Reference verification** — for every citation, web-search DOI + journal volume/issue/pages. Output `Paper/references_verified.csv` (one row per ref, with `verified_doi` column).

## Phase 7 — Word-count adjustment (~30 min)

Per-section count via `awk`. Expand thin sections **substantively**. Each added paragraph must contain at least one of:

- A specific number (effect size, sample size, p-value)
- A citation (with year and finding)
- A mechanism explanation
- A limitation acknowledgement
- A concrete future-research step

**Do not pad with filler.** No "in conclusion" / "as we have seen" / "moreover" stuffing.

## Phase 8 — Reference-format adaptation (~30 min)

Apply target style:

- **Harvard Manchester**: single quotes for article titles, `pp.` prefix, `doi:` lowercase, `Available at:` for grey lit, `(Accessed: <date>)` for websites
- **APA 7**: standard CSL, double quotes
- **AER traditional**: drop DOI from journal articles, italics on journal names

Always check the user's official handbook PDF first.

## Phase 9 — Version archive

Maintain immutable versioned files: `Paper/full_paper_v1.md` → `v9.md`, never overwrite. Approximate semantics:

- v1: first draft
- v2: tone polish
- v3-v4: external editor pass
- v5: numbers + figures verified
- v6: DOIs added
- v7: format fixes
- v8: reference-style adaptation
- v9: word-count final

Generate `.docx` for every `vN.md`.

## Project structure (standard)

```
<paper-name>/
├── Code/         # 6 R scripts (00-05)
├── Data/         # Raw / Intermediate / Clean
├── Results/      # Tables (.csv/.tex), Figures (.pdf)
├── Notes/        # DATA_ACQUISITION.md, lit_review.md, anchor PDFs
├── Paper/        # full_paper_vN.md, vN.docx, reviews, references_verified.csv
└── docs/         # WORKFLOW.md (per-project detailed playbook)
```

## Single-day time budget (8 hours total)

| Phase | Time |
|-------|------|
| 1 — Topic | 30 min |
| 2 — Literature | 1.5 h |
| 3 — Data | 1 h |
| 4 — R analysis | 2.5 h |
| 5 — First draft | 2 h |
| 6 — Review | 1 h |
| 7 — Word count | 30 min |
| **Total** | **8 h** |

## Departures from this workflow

- **Long-form (≥10,000 words)** — expand §3 (descriptives + balance tables), §4 (formal proof if applicable), §5 (sub-sample + mechanism splits), §6 (external validity), Appendix (full robustness suite)
- **Short brief (<3,000 words)** — collapse phases 5-9 into a single editorial pass
- **Non-DID identification** (RDD / IV / RCT replication) — Phase 4 changes accordingly. Ask user about new identification setup before proceeding.

## Codex-specific tool mapping

This file uses generic instructions. Codex's actual tool names differ from Claude Code:

| Action | Codex tool | Claude Code tool |
|--------|------------|------------------|
| Run command | `shell` | `Bash` |
| Read file | `shell` (`cat`) or read | `Read` |
| Edit file | `apply_patch` | `Edit` |
| Write file | `apply_patch` | `Write` |
| Web search | `web_search` (if enabled) | `WebSearch` |

When this file says "run", "read", "write", "search the web" — use whatever tool your platform exposes for that operation.

## Anti-patterns (do not do these)

- ❌ Skip Phase 0 questions and start writing immediately
- ❌ Use `max |t|` to select preferred specification
- ❌ Fabricate numbers in the paper
- ❌ Pad word count with filler instead of substantive content
- ❌ Write LaTeX directly instead of Markdown + Pandoc
- ❌ Double-number sections (manual `# 1 Introduction` + `--number-sections`)
- ❌ Skip reference verification (every cited paper needs a verified DOI)
- ❌ Overwrite `full_paper_v1.md` instead of creating `v2.md`

## Reference

- Detailed playbook: [`docs/WORKFLOW.md`](docs/WORKFLOW.md)
- Data acquisition guide: [`docs/DATA_ACQUISITION.md`](docs/DATA_ACQUISITION.md)
- Case studies: [`examples/case_studies.md`](examples/case_studies.md)
- R script templates: [`templates/Code/`](templates/Code/)
- Paper template: [`templates/Paper/full_paper_template.md`](templates/Paper/full_paper_template.md)

---
> Source: [CalebLiu/econ-paper-workflow](https://github.com/CalebLiu/econ-paper-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
