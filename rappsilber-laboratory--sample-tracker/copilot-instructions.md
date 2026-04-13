## sample-tracker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Install dependencies
pip install -r requirements.txt

# Initialize the database (run once before first use)
flask init-db

# Run the development server
flask run
```

There are no tests or linting tools configured in this project.

## Architecture

This is a Flask web application for laboratory sample tracking, focused on protein cross-linking and mass spectrometry experiments.

### Core Files

- **app.py** — Flask app, all routes, CRUD logic, and form/model wiring
- **models.py** — SQLAlchemy ORM models with relationships
- **forms.py** — WTForms form definitions, including a custom `MultiCheckboxField`
- **config.py** — Flask/SQLAlchemy config (SQLite by default)
- **schema.sql** — Raw DDL; used by `flask init-db` to create tables
- **fileInfoScript.py** — `SpectraAddressBook` utility that recursively scans directories for mass spectrometry data files (`.raw`, `.mgf`, `.mzml`, `.d`)

### Data Model

Five main entities: **Project → Experiment → Sample**, plus **Species**, **CellLine**, and **Virus** as reference data.

- Samples are polymorphic: `CrosslinkSample` vs `IdentificationSample`, each with a distinct set of type-specific fields. The type is stored in `crosslinking_type` and app.py nullifies irrelevant fields on save to maintain integrity.
- Species and CellLine relate to Sample via many-to-many junction tables (`sample_species`, `sample_cell_line`).
- CellLine relates to Virus via the `cell_line_virus` many-to-many junction table.
- Virus has an optional FK to Species and an optional `variant` field.
- Projects have an `active` flag for archival without data deletion.

### Request Flow

1. Route handlers in `app.py` instantiate forms (from `forms.py`) and query models (from `models.py`).
2. Form choices (dropdowns, multi-selects) are populated dynamically from the database at request time.
3. On POST, form data is validated, then mapped to SQLAlchemy model instances and committed.
4. Templates in `templates/<entity>/` render the response; `base.html` and `_form_helpers.html` are shared.

### File Browser

`/samples/<id>/files` instantiates `SpectraAddressBook` from `fileInfoScript.py` to scan a configured directory and display matching spectra files for a sample.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Rappsilber-Laboratory)
> This is a context snippet only. You'll also want the standalone SKILL.md file — [download at TomeVault](https://tomevault.io/claim/Rappsilber-Laboratory)
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
