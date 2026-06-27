## 000-global

> Global rules for Exam-Prompt — AI exam preparation system


# Exam-Prompt — Universal Agent Rules

## Project Identity
This is a **universal skill hub** that transforms any AI coding agent into a personal exam tutor for **any university worldwide**. It contains 24 reusable skills in `skills/<name>/SKILL.md` format.

## Key Files
- `AGENTS.md` — XML registry of all available skills (auto-discovered by agents)
- `CLAUDE.md` — Quick skill reference table
- `USE-CASES.md` — 33 real-world workflow examples
- `CONTRIBUTING.md` — How to contribute sample papers and skills
- `.cursorrules` — Legacy cursor rules (this .mdc file replaces/extends it)

## How to Operate

### When User Asks an Exam Question
1. Read the relevant `skills/<name>/SKILL.md` file
2. Follow its instructions step-by-step
3. The skill contains all rules for generating the correct output format

### When User Asks for Answers
- Load `skills/answer-writer/SKILL.md`
- Follow its 15-phase pipeline
- Output: Direct answer, no preamble, bolded keywords, numbered points

### When User Asks for Notes
- Load `skills/notes-generator/SKILL.md`
- Auto-select format based on subject type and time available
- Include spaced repetition schedule and mnemonics

### When User Asks for PYQ Analysis
- Load `skills/pyq-analyzer/SKILL.md`
- Run 13+ statistical analyses
- Output probability-ranked predictions

### When User Asks for a Different University
- Load `skills/universal-university-adapter/SKILL.md`
- Detect university from PDFs or description
- Route to correct pattern-specific output

## Architecture
```
skills/                      # 24 skill modules (SKILL.md format)
├── answer-writer/           # A+ answer generator
├── notes-generator/         # 12+ note formats
├── pyq-analyzer/            # PYQ analysis
├── imp-topics-generator/    # IMP topics
├── assignment-writer/       # Assignments
├── exam-paper-generator/    # Question papers
├── subject-prompt-bank/     # Sample answers
├── universal-document-reader/ # PDF/image to text
├── document-generator/      # MD to PDF
├── universal-university-adapter/ # Any university
├── flashcard-generator/     # Anki/Quizlet
├── study-planner/           # Schedules
├── mcq-practice-generator/  # MCQs
├── viva-oral-exam-prep/     # Viva prep
├── last-minute-crammer/     # Emergency plans
├── lab-report-writer/       # Lab reports
├── formula-sheet-generator/ # Formula sheets
├── mind-map-generator/      # Mind maps
├── case-study-solver/       # Case studies
├── essay-grader/            # Answer grading
├── cross-subject-mapper/    # Subject connections
├── sppu-exam-simulator/     # Exam simulation
├── session-config/          # Student profile
└── qp-fetcher/              # PYQ auto-fetch
```

## Constraints
- Never modify a `skills/<name>/SKILL.md` file unless asked
- Never make up university patterns — use the adapter skill
- Always prefer reading the SKILL.md over guessing the output format
- Keep answers directly actionable — no meta-commentary

---
> Source: [pinakdhabu/Exam-prompt](https://github.com/pinakdhabu/Exam-prompt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-26 -->
