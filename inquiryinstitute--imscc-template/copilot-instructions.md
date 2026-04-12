## imscc-template

> Assignment definitions for imscc_template


# `assignments/`

One folder per assignment, e.g. `assignments/01-homework/`.

## `assignment.json`

Required fields (see imscc_mcp / canvas_cc):

- `identifier`, `title`, `assignment_group_identifier_ref` (must exist in `course/assignment_groups.json`).
- `body` **or** `body_file` (HTML file relative to this folder). If **`instructions.myst.md`** exists, it **overrides** `body_file` at assemble time (**`docs/MYST.md`**).
- `points_possible`, `grading_type` (e.g. `points`), `workflow_state`.
- `submission_types` — e.g. `["online_text_entry"]`, `["online_upload"]`, `["online_text_entry","online_upload"]`.

Optional:

- `due_at`, `unlock_at`, `lock_at` (strings; IMS/datetime as expected by canvas_cc).
- `rubric_identifier` — Must match a rubric `identifier` in `rubrics/*.json`.
- `rubric_use_for_grading`, `rubric_hide_score_total` (booleans).
- **`github_classroom`** — Embeds GitHub Classroom + **Codespace** + repo + optional **Course book** / **Lab workbook** links. See **`docs/GITHUB_CLASSROOM.md`**. **Model:** **one GitHub org per class offering** + **one workbook repo per learner**; **`course_repository`**, **`course_book_path`**, per-assignment **`lab_book_path`**, **`classroom_assignment_url`**. Create with **`gh repo create … --template …`** or **`npm run gh-classroom`** (dedupes shared **`org/repo`**). The field is **stripped** from the IMS JSON output (only affects merged HTML). **LMS ↔ Classroom (LTI roster) and autograding → gradebook:** **`github-classroom-lti.mdc`** + **`docs/GITHUB_CLASSROOM.md`** § *LTI* — align **`points_possible`** with Classroom max points; document how autograder scores reach the LMS (CSV, Populi MCP, etc.).

Link each assignment from a **module** in `modules/sequence.json` using `content_type` `Assignment` and `identifierref` = this assignment’s `identifier`.

Skip `_template`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/InquiryInstitute)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/InquiryInstitute)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
