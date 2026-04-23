## thailand-hr-interviews

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Qualitative research repository for the **Thailand Human Rights Report 2025** (World Justice Project). Contains expert interview transcripts and structured analyses following the WJP Rule of Law Index framework.

## Analysis Pipeline

**Read `WORKFLOW.md` before making changes.** The pipeline has 5 mandatory steps:

```
Inputs (INT##.txt + INT##_Summary Insights.txt)
    ↓
Step 1: Interview Analysis → Outputs/interviews/INT##_Analysis.md
    ↓
Step 2: Dimension Synthesis → Outputs/synthesis/DIMENSION_#_*.md
    ↓
Step 3A: Quote Bank → Outputs/QUOTE_BANK.md
Step 3B: Chapter 2 Draft → Outputs/CHAPTER2_DRAFT.md
    ↓
Step 4: Dashboard HTML → Outputs/dashboard.html
    ↓
Step 5: Consistency Test → Outputs/consistency/metrics_v#.yaml (MANDATORY)
```

### Running Each Step

Use the prompts in `pipeline/prompts/` with the corresponding template from `pipeline/templates/`:

```
# Step 1: Single interview
Analiza la entrevista INT## siguiendo pipeline/prompts/01_interview_analysis.md

# Step 2: Single dimension (run all 5: 0-4)
Genera síntesis para Dimensión # siguiendo pipeline/prompts/02_dimension_synthesis.md

# Step 3A
Genera Quote Bank siguiendo pipeline/prompts/03_quote_bank.md

# Step 3B
Genera Chapter 2 Draft siguiendo pipeline/prompts/04_chapter2_draft.md

# Step 4
Genera dashboard HTML siguiendo pipeline/prompts/05_dashboard_html.md

# Step 5 (NEVER SKIP)
Ejecuta test de consistencia siguiendo pipeline/prompts/00_consistency_test.md
```

## Analytical Framework (CODEBOOK.md)

| Dimension | Topic | Sub-indicators |
|-----------|-------|----------------|
| 0 | Current Status | 0.1-0.4: Overall assessment, challenges, progress, context |
| 1 | Civil and Political Rights | 1.1-1.6: Religion, Discrimination, Expression, Assembly, Life/Security, Privacy |
| 2 | Participation | 2.1-2.4: Complaints, NGO checks, Civic participation, Information access |
| 3 | Labor & Property | 3.1-3.2: Labor rights, Expropriation |
| 4 | Justice | 4.1-4.8: Civil access, Criminal/Civil independence, Discrimination, Delays, Due process, Impartiality |

### Required Tags

**Cross-cutting:** `[CHAL]` `[HIGH]` `[QUOTE]` `[QUANT]` `[CONTRAST]`

**Vulnerable groups:** `VG-MW` (Migrant Workers), `VG-MM` (Malay Muslims), `VG-IP` (Indigenous), `VG-WM` (Women), `VG-LG` (LGBTQ+), `VG-YT` (Youth), `VG-PA` (Political Activists), `VG-HR` (HR Defenders), `VG-JN` (Journalists)

**Geographic:** `GEO-DS` (Deep South), `GEO-BK` (Bangkok), `GEO-RU` (Rural), `GEO-NA` (National)

## Critical Rules

### Interview Analysis (Step 1)
- **Two inputs required**: `INT##.txt` (transcript) + `INT##_Summary Insights.txt` (analyst notes)
- **Summary Insights are ground truth** - do not contradict analyst interpretations
- Use fragment IDs: `INT##-###` (e.g., `INT01-001`)
- **All quotes must be verbatim** from transcripts, never paraphrased

### Quote Traceability
All quotes must trace through the full chain:
```
Transcript → Interview Analysis → Dimension Synthesis → Quote Bank/Chapter 2
```

### Anchor Quotes
`pipeline/anchor_quotes.yaml` contains 22 critical quotes that **must appear in every pipeline run**:
- 16 critical priority (100% required)
- 6 important priority (80% target)

### Consistency Test (Step 5) - Pass Criteria

| Test | Threshold |
|------|-----------|
| Structural Validation | 100% sections present |
| Quote Traceability | 100% verifiable |
| Anchor Quotes (critical) | 100% present |
| Anchor Quotes (important) | ≥80% present |
| Code Validation | 0 invalid codes |
| Drift Analysis | No 🚨 alerts unresolved |

**Pipeline run is NOT complete until all tests pass.**

## Anonymization

Use interview codes `INT01-INT11` only. Expert names exist only in `/Original` folder.

## WJP Index Trends to Address

**Improvements to validate:** Privacy (+32% decade), Life/Security (+6%), Religion (+10% 5yr), Criminal justice at decade highs

**Declines to explain:** Equality (-2% last year), Expression (-4%), All Participation indicators declining, Labor (-5% 2020-2025), Civil Justice at decade lows

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aspardog) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-10 -->
