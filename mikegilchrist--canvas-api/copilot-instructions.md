## canvas-api

> Canvas LMS archival tool. Exports Classic Quiz submissions, assignment

# Canvas.API

Canvas LMS archival tool. Exports Classic Quiz submissions, assignment
submissions (SpeedGrader data), discussion boards, and Student Analysis
reports via the Canvas REST API. Output formats: JSON, HTML, SQLite, CSV,
and downloaded student files.

## Repository Layout

```
src/                        Python modules + CLI entrypoints
output/                     Generated output (gitignored, contains student PII)
canvas.api.conf.example     Example user profile config
README.TOKEN.SETUP.md       How to create and store a Canvas API token
EXAMPLES.txt                Offline mock pipeline walkthrough
CHANGELOG.md                Release history
```

## User Profile: `~/.canvas.api.conf`

INI-style config read by all API-calling CLIs. Eliminates the need to
pass `--base-url` and `--token-file` on every invocation.

```ini
[default]
base_url = https://utk.instructure.com
token_file = ~/.canvas_token
```

- Loaded by `load_profile()` in `io_utils.py`
- CLI arguments always override config values
- Missing config file is silently ignored (args become required)

## CLI Entrypoints (src/)

All prefixed with `canvas.api.` for tab-completion. Run any with `--help`.

**API scripts** (require base_url + token via config or CLI):

| Script                                  | Required Args     | Purpose                                          |
|-----------------------------------------|-------------------|--------------------------------------------------|
| `canvas.api.list_courses.py`            | (none)            | List courses; `-y YEAR`, `-s SEMESTER`           |
| `canvas.api.list_quizzes.py`            | COURSE_ID         | List quizzes in a course                         |
| `canvas.api.list_assignments.py`        | COURSE_ID         | List assignments; `--group NAME`                 |
| `canvas.api.report_to_csv.py`           | COURSE_ID         | Download quiz Student/Item Analysis CSVs         |
| `canvas.api.submissions_to_files.py`    | COURSE_ID         | Download assignment submissions and files        |
| `canvas.api.list_discussions.py`        | COURSE_ID         | List discussion topics                           |
| `canvas.api.discussions_to_json.py`     | COURSE_ID         | Download discussion threads to JSON              |
| `canvas.api.to_json.py`                 | COURSE_ID         | Export quiz submissions to canonical JSON        |
| `canvas.api.quiz_summary.py`            | COURSE_ID         | Per-student summary: score, time, words, wpm     |

**Unified wrappers** (dispatch to underlying scripts):

| Script              | Usage                                                 | Purpose                          |
|---------------------|-------------------------------------------------------|----------------------------------|
| `canvas.api.list`   | `canvas.api.list courses [-y YEAR] [-s SEMESTER]`     | List courses, quizzes, etc.      |
|                     | `canvas.api.list COURSE_ID {quizzes\|assignments\|discussions}` |                        |
| `canvas.api.get`    | `canvas.api.get COURSE_ID discussions [TOPIC_ID]`     | Download/export data             |
|                     | `canvas.api.get COURSE_ID assignments [ASSIGNMENT_ID]` |                                 |
|                     | `canvas.api.get COURSE_ID quizzes [QUIZ_ID] [--format FORMAT]` |                        |

The `quizzes` noun on `canvas.api.get` supports `--format csv|json|summary`
(default: csv). Maps to `report_to_csv.py`, `to_json.py`, and
`quiz_summary.py` respectively.

**Offline scripts** (no API access needed):

| Script                                  | Required Args  | Purpose                                     |
|-----------------------------------------|----------------|---------------------------------------------|
| `canvas.api.json_to_html.py`            | JSON_DIR       | Render canonical JSON to per-submission HTML |
| `canvas.api.json_to_sqlite.py`          | JSON_DIR OUTDB | Load canonical JSON into SQLite             |
| `canvas.api.json_to_all.py`             | JSON_DIR OUTDB | Run json_to_html + json_to_sqlite together  |
| `canvas.api.mock_to_json.py`            | MOCK_DIR       | Convert offline mock payloads to canonical JSON |

### Common Flags

- `-v, --verbose` -- detailed progress output
- `-n, --dry-run` -- show what would be done without making changes
- `-u, --update` -- skip items whose output already exists
- `--anonymize` -- replace student PII with anonymous identifiers

## Library Modules (src/)

| Module           | Purpose                                           |
|------------------|---------------------------------------------------|
| `canvas_http.py` | HTTP layer: GET/POST with auth, pagination, raw download |
| `canvas_api.py`  | Canvas endpoint wrappers (courses, quizzes, assignments, discussions, submissions, reports, rubrics) |
| `io_utils.py`    | JSON I/O helpers, `read_token()`, `load_profile()`, `sanitize_for_filename()` |
| `model.py`       | Canonical JSON schema for quiz submissions        |
| `render_html.py` | HTML rendering for quiz submissions               |
| `sqlite_writer.py` | SQLite schema and insert logic                  |

## Canvas API Notes

- Quiz Reports API is async: POST to trigger, poll progress, GET file URL,
  download CSV (no auth header on final download -- verifier in URL)
- Assignment Submissions API is synchronous; file attachment URLs also use
  verifier tokens (no auth header)
- Discussion Topics `/view` endpoint returns full threaded discussion
  (participants, entries, replies) in a single non-paginated response
- UTK Canvas returns 403 on `/courses/:id/quizzes/:id/questions` even for
  instructors -- Student Analysis CSV bypasses this
- Classic Quizzes and New Quizzes have separate APIs; this tool targets
  Classic Quizzes

## CRITICAL: Never Delete Published Assignments or Quizzes

Once a quiz or assignment is published on Canvas, **NEVER delete it** to
replace it with a new version unless explicitly instructed by the user.
Deleting a published quiz destroys student submissions -- even temporarily
lost work causes real harm. Instead, **update the existing quiz in place**.

This applies to all Canvas objects that can accumulate student work:
quizzes, assignments, discussion boards, pages with student interactions.

- `canvas.api.upload_quiz.py` currently offers to delete existing quizzes
  with the same session prefix. When a quiz is already published and has
  submissions, **decline the delete** and update instead.
- If a script lacks an update-in-place path, do the update manually via
  the Canvas API or web UI rather than delete-and-recreate.

## Authentication

Store your Canvas API token in `~/.canvas_token` (chmod 600).
See README.TOKEN.SETUP.md for step-by-step instructions.

With a profile config (`~/.canvas.api.conf`), no auth arguments are
needed on the command line. Without it, pass `--token-file ~/.canvas_token`
or `--token <inline-token>`.

## Output and PII

- All output goes under `output/` which is gitignored
- CSV and submission downloads contain student names, Canvas IDs, SIS IDs
- Use `--anonymize` to replace PII with Student_01..Student_NN
- Never commit student data to the repository

## Filename Conventions

- Underscores separate major sections, periods separate words within a section
- Example: `S01_2025-01-16_Mutations.Teamwork_student.analysis_12345-67890_2026-02-19.csv`
- `sanitize_for_filename()` in `io_utils.py` handles the slug conversion

## Development

- Python 3.7+, stdlib only (no pip dependencies)
- All scripts run from the repo root: `src/canvas.api.script.py ...` (no `python3` prefix)
- CLI entrypoints must be executable (`chmod +x`) and have `#!/usr/bin/env python3` shebang
- Library modules are not executable

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mikegilchrist)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/mikegilchrist)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
