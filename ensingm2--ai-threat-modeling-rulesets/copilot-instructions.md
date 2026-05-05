## ai-threat-modeling-rulesets

> **Project:** LLM Instruction Sets for Threat Modeling

# GitHub Copilot - Threat Modeling Framework

**Project:** LLM Instruction Sets for Threat Modeling

---

## Quick Start

1. **Read:** `.ai-instructions/core/entry-point.md`
2. **Read:** `.ai-instructions/skills/workflow-guide.md`
3. **⚠️ MANDATORY - Ask user BOTH questions in single prompt:**
   - **Operational Mode:** Collaborative or Automatic?
   - **Critic Review Mode:** With Critic or Without Critic? (recommend: Without for single-agent efficiency)
4. **Load:** `.ai-instructions/modes/[selected-mode].md`
5. **Confirm BOTH selections → Begin Stage 1**

**CRITICAL:** Steps 3-5 are NON-NEGOTIABLE. Never start Stage 1 without asking BOTH questions.

---

## ⚠️ CRITICAL RULES (Details: `.ai-instructions/skills/shared/critical-rules.md`)

| Rule | Summary |
|------|---------|
| **Mode First** | Ask user for mode BEFORE Stage 1 - never assume |
| **Critic Selection** | Ask user about Critic Review mode at startup - default OFF for single-agent |
| **Never Fabricate** | All claims need sources; use confidence levels |
| **Batched Execution** | One phase per response (Work OR Critic, not both) |

---

## Key Resources (All paths from `.ai-instructions/skills/`)

| Resource | File |
|----------|------|
| **Workflow** | `workflow-guide.md` |
| **Critical Rules** | `shared/critical-rules.md` |
| **Terminology** | `shared/terminology.md` |
| **Confidence Levels** | `shared/confidence-calibration.md` |
| **Output Specs** | `shared/output-file-requirements.md` |

---

## 6-Stage Outputs → `[target]/output/threat-model/`

`00-final-report.md` ← `01-system-understanding.md` → `02-data-flow-analysis.md` → `03-threat-identification.md` → `04-risk-assessment.md` → `05-mitigation-strategy.md`

---

## Copilot Notes

- Read instruction files at stage start (limited dynamic loading)
- Use conversation context to track stage progress
- Reference previous outputs for cross-stage consistency

---

*Full documentation: `.ai-instructions/README.md` and `.ai-instructions/core/entry-point.md`*

---
> Source: [ensingm2/AI-threat-modeling-rulesets](https://github.com/ensingm2/AI-threat-modeling-rulesets) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-04 -->
