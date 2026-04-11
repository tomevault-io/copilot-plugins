## flexible-web-checker

> Goal: Get enough context fast. Parallelize discovery. Stop when actionable.

<context_gathering>
Goal: Get enough context fast. Parallelize discovery. Stop when actionable.

Method:
- Start broad -> fan out to focused subqueries.
- Parallelize varied queries; read top hits. Deduplicate paths & cache; don't repeat.
- Avoid over-searching. Run targeted batch if needed.

Stop Criteria:
- Can name exact content to change.
- Top hits converge (~70%) on one area.

Escalation:
- If signals conflict/fuzzy: Run one refined parallel batch, then proceed.

Depth:
- Trace only modified symbols or direct contracts. Avoid transitive expansion.

Loop:
- Batch search -> Minimal plan -> Complete task.
- Re-search only if validation fails or new unknowns. Action > Search.
</context_gathering>

<self_reflection>
- First, define a rubric (5-7 categories) for a "world-class one-shot web app".
- Iterate internally until the solution hits top marks in all categories.
- Critical: Do not show the rubric to the user.
</self_reflection>

<persistence>
- Continue until the query is FULLY resolved. Do not yield early.
- On uncertainty: Research or deduce the most reasonable approach. Never stop.
- Do not ask for confirmation on assumptions. Decide, proceed, and document for reference later.
</persistence>

<code_editing_rules>
<principles>
- Readability: No non-standard chars/emojis in code/comments.
- Maintainability: Follow directory structure & naming conventions.
- Consistency: Adhere to design system (tokens, typography, spacing, components).
- Visuals: High quality (spacing, hover states) per OSS guidelines.
</principles>

<stack_defaults>
Backend:
- Framework: Django 5.2 (Python)
- Plugins: django-safedelete, django-redis (future)

Frontend:
- Coding: TypeScript 5, React 19, SCSS(sass) 1.69.7
- Styling: Tailwind CSS 4
- Icons: Font Awesome 6
- Dialog: SweetAlert2
</stack_defaults>
</code_editing_rules>

<project_details>
<instruction>
CRITICAL: You MUST read the current content of [README.md](README.md) BEFORE taking any action.
You MUST strictly adhere to the "Development Rules" (開発ルール) section within README.md.
</instruction>
<python_execution>
You must activate the venv environment before running python commands.
Activate venv:
source venv/bin/activate
</python_execution>
</project_details>

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kakehashi-inc)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/kakehashi-inc)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
