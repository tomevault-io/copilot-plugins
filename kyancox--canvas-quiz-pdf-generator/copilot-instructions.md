## canvas-quiz-pdf-generator

> Generate individual PDFs per question type per student from Canvas quiz CSVs and LaTeX rubrics.

# Canvas Quiz PDF System

## Project Goal
Generate individual PDFs per question type per student from Canvas quiz CSVs and LaTeX rubrics.

## Architecture
- Config-driven: One config file per quiz
- Generic core modules work for all quizzes
- Proprietary configs are git-ignored
- HTML templates preserve student data exactly (no parsing/modification)

## Code Style
- Follow PEP 8
- Use type hints
- Descriptive variable names
- Docstrings for all public functions
- Keep functions focused and single-purpose

## Key Constraints
- Never modify student HTML answers
- Support MathJax for equation rendering
- Generate separate PDFs by question type for TA grading efficiency

## Development Workflow
- Stop at important milestones for user review
- ALWAYS stop after generating HTML templates for inspection
- Stop after major feature completions to allow testing
- Reference files from parent directory (original MVP) when needed: ../quiz*_solutions_rubric.tex, ../Quiz*.csv

## Critical Template Requirements
- TikZ graphs MUST be converted to images (SVG/PNG) and embedded in HTML
- Question parts (a, b, c) MUST all appear with their question text
- Solution boxes MUST be preserved and styled distinctly
- Each part needs its own answer placeholder (e.g., Part A → {{ANSWER_1_1}}, Part B → {{ANSWER_1_2}})

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kyancox) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-14 -->
