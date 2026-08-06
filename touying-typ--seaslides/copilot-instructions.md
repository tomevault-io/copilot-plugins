## seaslides

> > **Start here.** This is the project entry point for AI agents.

# AGENTS.md

> **Start here.** This is the project entry point for AI agents.

---

## Project Overview

**SeaSlides Typst Slides Skill** — an AI-driven presentation generation system. Full Mode uses
multi-role collaboration (Strategist -> Image_Generator -> Executor); Quick Mode authors directly
from an existing theme. Both convert source documents (PDF, DOCX, URL, Markdown) into
Typst/Touying slides exported as PDF/PNG/PPTX.

**MUST read `skills/seaslides-typst-slides-skill/SKILL.md`** before any presentation generation task or repo modification. SKILL.md is the authoritative workflow that owns project creation, role switching, serial execution, and quality gates.

---

## Standalone Workflows

| User Intent | Workflow |
|-------------|----------|
| Fast first usable deck with reasonable defaults | `${SKILL_DIR}/workflows/quick-mode.md` |
| Topic-only request (no source material) | `${SKILL_DIR}/workflows/topic-research.md` |
| Phase B resumption in a fresh chat | `${SKILL_DIR}/workflows/resume-execute.md` |
| Data charts need coordinate calibration | `${SKILL_DIR}/workflows/verify-charts.md` |
| Narration / video after post-processing | `${SKILL_DIR}/workflows/generate-audio.md` |
| Animation tuning (explicit user request) | `${SKILL_DIR}/workflows/customize-animations.md` |
| Visual self-check (explicit user request) | `${SKILL_DIR}/workflows/visual-review.md` |
| New template creation | `${SKILL_DIR}/workflows/create-template.md` |
| Theme selection | `${SKILL_DIR}/workflows/select-theme.md` |

---

## Execution Requirements

- **Template creation**: `${SKILL_DIR}/templates/theme_design_spec_reference.md`
- **Typst/Touying constraints**: `${SKILL_DIR}/references/shared-standards.md`
- **Canvas formats**: `${SKILL_DIR}/scripts/config.py`
- **Icon library**: `${SKILL_DIR}/templates/icons/index.json`

---

## Required Conventions

1. **Style rules**: Follow `docs/rules/` when editing prompt files, Python scripts, or other code/prose
2. **Language consistency**: Markdown files must maintain single-language consistency within each directory

---

## Compatibility Boundary

This repo is **a workflow/skill package, not an app or service scaffold**. Do not assume generic project conventions (CI/CD, linters, etc.). On any conflict, SKILL.md takes priority.

---

## Command Quick Reference

```bash
# ── Source conversion ──
python ${SKILL_DIR}/scripts/source_to_md/pdf_to_md.py  FILE.pdf  -o out.md
python ${SKILL_DIR}/scripts/source_to_md/doc_to_md.py   FILE.docx -o out.md
python ${SKILL_DIR}/scripts/source_to_md/excel_to_md.py FILE.xlsx -o out.md
python ${SKILL_DIR}/scripts/source_to_md/ppt_to_md.py   FILE.pptx -o out.md
python ${SKILL_DIR}/scripts/source_to_md/web_to_md.py   "URL"     -o out.md

# ── Project management ──
python ${SKILL_DIR}/scripts/project_manager.py init PROJECT_NAME
python ${SKILL_DIR}/scripts/project_manager.py workspace WORKSPACE --entry MAIN.typ --prepare
python ${SKILL_DIR}/scripts/project_manager.py import-sources PROJECT sources/
python ${SKILL_DIR}/scripts/validate_project.py ./projects/PROJECT

# ── Image tools ──
python ${SKILL_DIR}/scripts/image_gen.py --prompt "..." --backend openai -o img.png
python ${SKILL_DIR}/scripts/image_search.py --query "..." -o img.jpg
python ${SKILL_DIR}/scripts/analyze_images.py ./projects/PROJECT

# ── Compilation & quality ──
python ${SKILL_DIR}/scripts/typst_compiler.py ./projects/PROJECT --format pdf
python ${SKILL_DIR}/scripts/typst_compiler.py ./projects/PROJECT --format pptx
python ${SKILL_DIR}/scripts/typst_quality_checker.py ./projects/PROJECT

# ── Audio narration ──
python ${SKILL_DIR}/scripts/notes_to_audio.py ./projects/PROJECT
```

For an editor-owned or otherwise existing workspace, use the `workspace`
command and pass the active workspace-relative entry to compile, quality, and
validation tools. Do not run `init` inside an existing workspace.

---

## Core Directories

| Path | Role |
|------|------|
| `skills/seaslides-typst-slides-skill/SKILL.md` | **Main authority** — read first |
| `skills/seaslides-typst-slides-skill/references/` | Role definitions, tech specs |
| `skills/seaslides-typst-slides-skill/scripts/` | Python tools |
| `skills/seaslides-typst-slides-skill/templates/` | Themes, charts, icons, packages |
| `skills/seaslides-typst-slides-skill/workflows/` | Standalone workflows |
| `docs/` | Research, surveys, documentation |
| `examples/` | Example projects & benchmarks |
| `projects/` | User workspace (gitignored) |

---
> Source: [touying-typ/seaslides](https://github.com/touying-typ/seaslides) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-06 -->
