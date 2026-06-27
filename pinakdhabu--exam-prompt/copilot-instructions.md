## exam-prompt

> This is a universal skill hub for AI-powered exam preparation. It contains 24 reusable skills in

# GitHub Copilot Instructions — Exam-Prompt

## Project Purpose

This is a universal skill hub for AI-powered exam preparation. It contains 24 reusable skills in
`skills/<name>/SKILL.md` format that guide AI agents to generate exam answers, notes, analysis,
assignments, and more for ANY university worldwide.

## Key Architecture

- `skills/` — 24 SKILL.md files, each a complete instruction set for one exam task
- `AGENTS.md` — XML registry of all skills (auto-discovered by agents)
- `examples/` — Sample papers + solutions for SPPU Computer Engineering
- `USE-CASES.md` — 33 documented workflow examples

## For Exam Answers

When asked to write an exam answer, load `skills/answer-writer/SKILL.md` and follow:

1. Direct answer, no preamble
2. Bold technical terms on first use
3. Numbered points for body, tables for comparisons
4. Definition + points + diagram + example + closing (for 4+ marks)
5. 1 mark = 2-3 lines, ~3 lines per additional mark

## For Notes

When asked to generate notes, load `skills/notes-generator/SKILL.md` which supports 12+ formats.

## For PYQ Analysis

When asked to analyze past papers, load `skills/pyq-analyzer/SKILL.md` which supports 13+
statistical methods.

## University Patterns Supported

SPPU 2019/2024, VTU, JNTU, Mumbai Univ, AKTU, RGPV, IIT/NIT, North American, UK/Oxford/Cambridge,
Generic (5 units, 100 marks, 3 hours)

## Rules

- Always read the relevant SKILL.md before responding
- Never make up university patterns — use `universal-university-adapter` skill
- Default: SPPU 2019 Pattern if no university specified
- This repo is 100% free and open source — no paid features

---
> Source: [pinakdhabu/Exam-prompt](https://github.com/pinakdhabu/Exam-prompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
