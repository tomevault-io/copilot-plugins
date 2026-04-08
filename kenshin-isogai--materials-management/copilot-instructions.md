## materials-management

> Please use `uv run` instead of `python`.



## Rule

### Executing Python code

Please use `uv run` instead of `python`. 

The python environment in this project folder is prepared using `uv`.

### Required Context Documents (Read Before Implementing)

When implementing or modifying code, consult these documents first:

1. `specification.md` (source of truth for requirements and contracts)
2. `documents/technical_documentation.md` (architecture and maintenance guidance)
3. `documents/source_current_state.md` (current implementation snapshot)
4. `documents/change_log.md` (recent behavior and design changes)

If any conflict appears, follow precedence:

1. `specification.md`
2. `documents/technical_documentation.md`
3. current code behavior

### Tools During PDF/CSV Task

#### System tool

`Tesseract OCR` (installed via winget)
- Default path: `C:\Program Files\Tesseract-OCR\tesseract.exe`
- Installed language data: `eng` and `osd` only.

#### How to use (examples)

List available OCR languages:

```powershell
& "C:\Program Files\Tesseract-OCR\tesseract.exe" --list-langs
```

Note: If you need Japanese OCR, install `jpn` traineddata for Tesseract and set `TESSDATA_PREFIX` as needed.

### Application Update Workflow (Required)

When updating this application, follow this order:

1. Confirm scope and impacted areas (`backend`, `frontend`, DB schema, quotation file flow).
2. Implement the change in the domain layer first (`backend/app/service.py`), then wire adapters (`backend/app/api.py`, `backend/main.py`) and UI (`frontend/src`) as needed.
3. If schema/constraints change, update `backend/app/db.py` with idempotent migration-safe changes.
4. Run tests with `uv run`:
   - Full backend tests: `uv run python -m pytest`
   - Or targeted tests for touched behavior (for faster iteration), then run full suite before completion.
5. Validate runtime behavior for changed user flows (API/CLI/UI path that was modified).
6. Update documentation in the same change set:
   - User-facing setup/usage: `README.md` (root and/or `backend/README.md`, `frontend/README.md` if relevant)
   - Architecture/data model/maintenance notes: `documents/technical_documentation.md`
   - Current-state snapshot updates: `documents/source_current_state.md` when structure/behavior changed
   - Change history updates: `documents/change_log.md` for meaningful changes
   - Requirements/spec updates: `specification.md` when behavior or contract changed
7. In your final report, always include:
   - What changed
   - What tests were run (and results)
   - What documentation was updated

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Kenshin-Isogai)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Kenshin-Isogai)
<!-- tomevault:4.0:copilot_instructions:2026-04-08 -->
