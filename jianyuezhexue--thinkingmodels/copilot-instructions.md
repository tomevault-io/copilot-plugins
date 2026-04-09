## thinkingmodels

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Chinese "Thinking Models Platform" (思维模型平台) - a business planning and documentation project for a SaaS product. It is currently in the **ideation/planning phase** with business documentation complete but **no application code yet written**.

The project aims to build a platform that uses structured thinking models and AI to help users think deeply about complex problems.

**Current State:**
- Business plans, market research, and technical architecture documents (in Chinese)
- Python scripts for generating PowerPoint presentations
- Empty `frontend/` and `backend/` directories (not yet populated)
- No build system, test suite, or CI/CD configured

## Planned Architecture (From Documentation)

When development begins, the planned stack is:
- **Backend:** GoLang + Gin framework
- **Frontend:** Vue.js + Element Plus
- **Database:** MySQL + Redis
- **AI Service:** Third-party LLM APIs (Kim2.5 or Minimax)

## Python Presentation Scripts

The repository contains Python scripts for generating business plan presentations:

- `create_presentation.py` - Basic PowerPoint generation using `python-pptx`
- `create_enhanced_presentation.py` - Enhanced version with background images
- `create_visual_elements.py` - Generates PNG backgrounds using PIL

**Dependencies:** `python-pptx`, `PIL` (Pillow)

Run individually: `python create_presentation.py`

## Important Files

- `authorization.md` - **SENSITIVE** (contains database credentials) - This file is gitignored and must never be committed
- Business plan documents (in Chinese): `businessPlan.md`, `marketResearch.md`, `techCostAnalysis.md`, `ratingSystem.md`, `legalDisclaimer.md`

## README Accuracy Note

The existing `README.md` is misleading - it describes a thinking models library (docs/, src/, tests/, examples/) that does not exist. The actual project is a SaaS business plan with no code implementation yet.

## Naming Conventions

Documentation files use **camelCase** naming (e.g., `businessPlan.md`, `projectIdea.md`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jianyuezhexue)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/jianyuezhexue)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
