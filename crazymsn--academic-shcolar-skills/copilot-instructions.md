## academic-shcolar-skills

> This repository is a cleaned redistribution mirror of Academic Research Skills.

# Agent Instructions

This repository is a cleaned redistribution mirror of Academic Research Skills.
Preserve upstream attribution, `LICENSE`, and `NOTICE.md`; the license is
CC BY-NC 4.0 and includes a non-commercial restriction.

## Runtime Support

- Claude Code uses the native four-skill layout:
  `deep-research/`, `academic-paper/`, `academic-paper-reviewer/`, and
  `academic-pipeline/`.
- Codex uses the router skill in `codex/academic-research-suite/`; install it
  with `scripts/install-codex-skill.ps1` or `scripts/install-codex-skill.sh`.
- OpenCode uses `opencode.json` and `.opencode/agents/academic-research-suite.md`.

## ARS Routing

Read only the workflow needed for the current task:

- Research, literature review, systematic review, meta-analysis, fact-checking,
  or research-question narrowing: `deep-research/SKILL.md`.
- Paper planning, outline, drafting, abstract, citation checks, disclosure, or
  format conversion: `academic-paper/SKILL.md`.
- Peer review simulation, editorial decisions, or re-review:
  `academic-paper-reviewer/SKILL.md`.
- End-to-end research-to-paper work: `academic-pipeline/SKILL.md`.

If the user only gives a broad paper topic without a clear research question,
start with deep-research Socratic scoping. Do not draft an outline until the
research question is concrete enough to answer.

For current facts, citation verification, regulations, venue rules, or source
claims, browse or otherwise verify against primary/official sources. Do not
invent citations, data, experiments, or reviewer findings.

When editing this mirror, keep changes small and clearly marked. Do not restore
removed GitHub Actions, third-party HTML snapshots, or showcase PDF binaries
unless the user explicitly accepts the redistribution risk.

---
> Source: [crazymsn/academic-shcolar-skills](https://github.com/crazymsn/academic-shcolar-skills) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-05-28 -->
