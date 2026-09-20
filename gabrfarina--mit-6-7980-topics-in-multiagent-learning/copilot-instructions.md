## mit-6-7980-topics-in-multiagent-learning

> Reflect every user-requested course change in both the current index page and the syllabus PDF in the same update.

# Course materials

Reflect every user-requested course change in both the current index page and the syllabus PDF in the same update.

- The editable syllabus and schedule are in `syllabus/6.7980 F26 Syllabus.typ`.
- Generate the current `html/index.html` with `scripts/course_index.py` and `html-export.json`.
- Rebuild `syllabus/6.7980 Fall 2026 Syllabus.pdf` and synchronize the copy at `html/syllabus.pdf`, which is linked from the index page.
- Edit shared facts in the syllabus's `course` dictionary and prose in its `item(...)` / `course-text(...)` blocks. The index reads these via `scripts/course_data.py`; do not duplicate course text in Python.
- `html-export.json` uses `notes` with stable `syllabus_ids` and a separate `slides` map keyed by lecture ID. Numbers, dates, course facts, and citation metadata are generated in `.build/html-export.json`; do not author redundant numbers or dates. Use `scripts/public_files.py` for export paths and validation shared with deployment.
- Verify schedule consistency and visually check the rebuilt PDF after changes.
- Reorder the syllabus's date-free `lecture(...)`, `no-class(...)`, and `module[...]` outline. `schedule(class-dates, outline)` assigns dates and lecture numbers; verified class dates and fixed academic-calendar exceptions are in `syllabus/fall-2026-calendar.typ`. Keep stable lecture IDs with their topics and map notes through `syllabus_ids` in `html-export.json`.
- Use Frutiger only for bold text and headings in the syllabus PDF. Use New Computer Modern for regular and italic body text. Do not use PT Sans, and check the embedded fonts when changing typography.
- Load the bundled regular and bold Frutiger faces with `--font-path html-exporter/assets/fonts` when compiling the syllabus. `make syllabus` rebuilds and synchronizes both PDF copies with this setting; the full site build uses it too.

# FoW arena connection settings

- The user has approved hardcoding `https://6s890.lids.mit.edu` as the default FoW arena. It may appear in the public frontend and documentation.
- An explicit arena link takes precedence over the browser's remembered successful connection, which takes precedence over the default. Remember only the validated backend origin after a successful connection.
- Keep team tokens in memory only and scrub them from the URL before making requests. A token-bearing link must provide its own explicit arena address; never send its token to a remembered or default destination implicitly.
- Label ratings as `ELO` in the FoW interface; keep the BayesElo calculation and backend API identifier unchanged.

---
> Source: [gabrfarina/MIT-6.7980-Topics-in-Multiagent-Learning](https://github.com/gabrfarina/MIT-6.7980-Topics-in-Multiagent-Learning) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-09-20 -->
