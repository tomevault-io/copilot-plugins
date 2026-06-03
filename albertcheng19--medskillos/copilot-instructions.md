## medskillos

> MedSkillOS is a medical-grade agent skill operating system for clinician-reviewed workflows.

﻿# AGENTS.md

MedSkillOS is a medical-grade agent skill operating system for clinician-reviewed workflows.

## Non-negotiable rules

- Do not add copied textbook content, proprietary guideline text, or paywalled clinical summaries.
- Do not add autonomous diagnosis or treatment behavior.
- Do not add hidden shell or network execution.
- Preserve schemas and tests when changing skills.
- High-risk medical skills must require human review.
- All executable scripts must be deterministic and safe by default.
- New skills must include `SKILL.md`, `medskill.yaml`, schemas, `risk.md`, tests, and at least one fixture.

## Preferred workflow

1. Read `medskill.yaml`.
2. Read `SKILL.md`.
3. Inspect schemas.
4. Run `python scripts/self_test.py` if available.
5. Update tests with behavior changes.
6. Summarize medical safety implications in PRs.

---
> Source: [albertcheng19/MedSkillOS](https://github.com/albertcheng19/MedSkillOS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-03 -->
