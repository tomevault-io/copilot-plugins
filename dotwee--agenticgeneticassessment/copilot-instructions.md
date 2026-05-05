## agenticgeneticassessment

> This is a multi-agent workflow for genetics-informed health assessment. The workflow splits analysis into 7 specialist agents that can run in parallel where dependencies allow.

# GitHub Copilot Custom Instructions for Agentic Genetic Assessment

## Project Context

This is a multi-agent workflow for genetics-informed health assessment. The workflow splits analysis into 7 specialist agents that can run in parallel where dependencies allow.

This file contains Copilot-specific guidance. Shared vendor-neutral guidance is in `AGENTS.md`.

## How to Use This Project

1. Read `instructions.md` for the global workflow rules and parallelism diagram.
2. Specialist prompts live in `specialists/` as XML files.
3. Use the VS Code code snippets (prefix `/interviewer`, `/data-engineer`, etc.) to quickly insert specialist prompts into Copilot Chat.
4. Follow the phased workflow: Interviewer + Data Engineer (parallel) → Geneticist → Clinician → Action Planner → Report Assembler → Reviewer.
5. If teammates use other harnesses (Codex, Claude Code, OpenCode), use `HARNESS-COMPATIBILITY.md` for equivalent MCP and subagent setup.

## Key Rules for All Agents

- Do NOT over-interpret single SNPs without phenotype confirmation.
- Always use conditional language: "if X is confirmed, consider Y."
- Flag safety concerns and contraindications explicitly.
- Respect user-stated constraints (budget, diet, preferences).
- This is educational, not medical advice.

## Knowledge Base

- `knowledge/variant-reference.md` — rsID reference tables by pathway
- `knowledge/lab-reference.md` — Confirmatory lab tests by pathway
- `knowledge/pubmed-evidence-lookup.md` — Curated PMID lookup tables by pathway
- `data/example-genotype.tsv` — Example genotype file for testing

---
> Source: [dotWee/AgenticGeneticAssessment](https://github.com/dotWee/AgenticGeneticAssessment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-02 -->
