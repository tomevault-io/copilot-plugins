## knowledge-base-tool

> A multi-feature, file-backed web application for managing personal knowledge:

# Knowledge Base Tool — Project Guide

A multi-feature, file-backed web application for managing personal knowledge:
files, documentation, training, and utilities. FastAPI + Jinja2 server-rendered
pages, Markdown files as the source of truth, SQLite only for indexes.

---

## Getting Started

Docker (recommended — matches production):
```bash
docker compose -f dockerise/docker-compose.yml up --build -d
```
The app is served at [http://localhost:5050](http://localhost:5050).

Local dev:
```bash
python3 -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
python cli.py serve            # http://127.0.0.1:5050
```

**Python 3.10+ is required.** Several modules use PEP 604 annotations
(`Transcript | None`) without `from __future__ import annotations`, so 3.9
raises `TypeError` at import. The Docker image pins `python:3.12-slim`.

---

## Architecture

```
app/
├── main.py          # ALL FastAPI routes (~2.4k lines) + module-level manager singletons
├── config.py        # load_settings() → Settings dataclass; every path comes from here
├── models.py        # Pydantic Post schema
├── parser.py        # frontmatter ↔ Post
├── post_manager.py  # Post CRUD + slugify + auto-index
├── search.py        # SQLite FTS5 engine (posts)
├── tag_manager.py   # tag counts + index.json
├── markdown.py      # render_with_toc()
├── exporter.py      # static HTML site export
├── hooks.py         # plugin event system (on_post_created/updated/deleted)
├── books.py         # collections, chapters, imported resource books (PDF/EPUB/…)
├── diagrams.py      # Mermaid/PlantUML sources + markdown export
├── notes.py  tasks.py  emails.py  bookmarks.py  api_docs.py  music.py
├── toeic.py  transcripts.py  vault.py  dictionary_db.py  sync.py
templates/           # Jinja2 — base.html + one set per feature
static/              # style.css, app.js, diagrams.js, vendored jszip/epub/mermaid
tests/               # pytest suite
cli.py               # Typer CLI (serve, new, list, search, build-index, export)
launcher.py          # pystray desktop tray launcher
config.yaml          # data dir + UI + server config
```

**Core patterns:**

- **One manager class per feature.** `NoteManager`, `TaskManager`, `BookManager`,
  … each owns a directory and does its own file I/O. Managers are instantiated
  **once at import time** in `app/main.py` (`pm`, `tm`, `books`, `notes`,
  `tasks_mgr`, `email_mgr`, `transcripts_mgr`, `dict_db`, …). `vault_manager` is
  a singleton created inside `app/vault.py` itself.
- **Markdown + YAML frontmatter is the source of truth.** SQLite (`.kb/`) holds
  only rebuildable indexes: `search.db` (FTS5) and `dictionary.db`.
- **Paths always come from `Settings`** (`app/config.py`), derived from
  `config.yaml`. Never hardcode a data directory.
- **Routes are thin.** Parse the form, call the manager, redirect `303`. Domain
  logic belongs in the manager module, not in `main.py`.
- Some routes use module-level path constants (`NOTES_DIR`, `POSTS_DIR`,
  `BG_DIR`, `FONTS_DIR`, `RESUME_PATH`, `LANG_PATH`) rather than the manager —
  relevant when testing (see below).

---

## Testing

```bash
python -m pytest tests/ -q                                   # full suite
python -m pytest tests/test_tasks.py -q                      # one file
python -m pytest tests/ -q --cov=app --cov=cli --cov-report=term-missing
```

Current state: **593 tests, 96% coverage** of `app/` + `cli.py`. Keep it there —
every new feature or bugfix lands with tests.

- Tests live in `tests/test_<feature>.py`. Manager unit tests first, then route
  tests through `TestClient`.
- PyMuPDF-dependent tests (PDF page rendering) skip automatically when `fitz`
  is unavailable, so a plain `requirements.txt` install stays green.
- **Use the `write-unit-test` skill** when adding tests — it documents the
  fixture patterns and the module-level-singleton monkeypatch gotchas.

---

## Feature Overview

All 12 major features in the system:

### 1. 📝 Posts
* **Main Function**: Article and long-form note publisher.
* **Short Description**: Allows creating and editing markdown posts. Supports tag organization, categorization, image uploads, and SQLite-backed FTS5 full-text search.

### 2. 📚 Series
* **Main Function**: Multi-part article binder.
* **Short Description**: Chains multiple standalone posts into structured series, creating next/previous page pagination and ordering for chapter-by-chapter reading.

### 3. 📖 Books & Reader
* **Main Function**: Digital library manager & immersive reader.
* **Short Description**: Organize book collections of chapters. Features an uploaded files section supporting **PDF, EPUB, MOBI, CBZ, FB2, XPS** formats. Offers a realistic **3D Page-Flip Reader** (spread-view for PDFs/Comics, responsive typography + TOC sidebar for EPUBs/MOBI), page jump, and resource deletion.

### 4. 🎧 TOEIC Test Preparation
* **Main Function**: Interactive practice sets for TOEIC training.
* **Short Description**: Features custom-formatted listening and reading tests. Users can select answers via radio inputs, play synced audio transcripts, submit answers for grade calculation, and view comprehensive explanation logs.

### 5. 🎵 Music Manager
* **Main Function**: Personal audio player and library organizer.
* **Short Description**: Supports importing audio files, editing track metadata, organizing songs into playlists, and playing them in a persistent audio drawer.

### 6. 🗒️ Notes
* **Main Function**: Quick-access, board-style sticky notes with backup tools.
* **Short Description**: Simple grid layout for jotting down instant ideas. Notes can be tagged, pinned to the top of the feed, and rendered with custom background themes (plain, lines, dots, grid, sticky). Includes a **Manage** panel to **export all notes as a ZIP file**, **import notes from a ZIP archive**, or **import individual/multiple .md files directly**.

### 7. 📄 API Docs
* **Main Function**: REST API endpoint documentation.
* **Short Description**: Let developers document APIs. Groups API endpoints under projects, specifying request methods, URL paths, headers, query parameters, body schemas, and response formats in a beautiful schema view.

### 8. 🔖 Bookmarks
* **Main Function**: Link saver and cataloger.
* **Short Description**: Stores bookmarks and external links. Organize them with quick-filter tags and category folders.

### 9. ✅ Tasks
* **Main Function**: Task manager with version history.
* **Short Description**: Manage tasks with dynamic subtask checklists (status: to-do, in-progress, done) and notes. Updates are saved as individual historical files (`dd_mm_yyyy_user_name_updated_times.md`), letting users view previous versions or clear old histories.

### 10. ✉️ Email Composers
* **Main Function**: Template-based professional email compiler.
* **Short Description**: Drafting tool with 8 pre-seeded business templates (welcome emails, follow-ups, outreach). Custom variables (e.g. `{{name}}`) are extracted into interactive form inputs, rendering real-time draft previews with copy actions and direct `mailto` desktop client launch. Built-in templates cannot be edited or deleted.

### 11. 📓 Vault
* **Main Function**: Obsidian-like Markdown knowledge manager.
* **Short Description**: Manage interconnected Markdown files in a hierarchical folder structure. Features a drag-and-drop file tree, live Markdown preview, an auto-generated Table of Contents pane, and inline context actions (rename, delete, move) to quickly organize knowledge bases locally.

### 12. 🧩 Diagrams
* **Main Function**: Mermaid / PlantUML diagram editor with export.
* **Short Description**: Write diagram source (flowcharts, sequence, class, state, ER, gantt, pie…) with a **live side-by-side preview** and inline parse errors. Mermaid renders in-browser from a locally vendored library, so it works offline, and exports to **SVG** or **PNG** (2×, background flattened). Every diagram exports to **markdown** with a fenced ```mermaid / ```plantuml block, so it renders as a real diagram on GitHub, Obsidian, or VS Code. PlantUML is stored, previewed as source, and markdown-exported — image export is Mermaid-only (no pure-Python PlantUML renderer; see the note below).

### 13. 🔒 Lock Screen
* **Main Function**: Client-side privacy screen overlay.
* **Short Description**: Prevent shoulder-surfing by securely locking the application interface. Users can set a custom password in Settings and instantly trigger a beautiful frosted-glass lock screen via a floating button. Keeps the session locked across page reloads until unlocked.

**Also present** (not in the numbered list above but fully wired): **Dictionary**
(personal vocabulary store, SQLite-backed), **Resume** (frontmatter-driven CV
with markdown export), **Transcripts** (AI conversation JSONL logs with
full-text search), and **Sync/Backup** (mirror `notes/posts/tasks/vault` to a
sync folder; ZIP backups).

---

## Gotchas worth knowing

- **Never write a literal closing `script` tag inside inline JS in a template** —
  even in a comment or string, the HTML parser ends the script element there and
  silently drops the rest of your code (this broke Mermaid rendering once; a test
  in `test_diagrams.py` now guards it).
- **Pass data into inline JS with `| tojson`**, not a raw `text/plain` block:
  `tojson` escapes `<`, `>`, `'` as `\uXXXX`, whereas script elements don't
  entity-decode, so escaped markup would arrive as literal `&lt;`.
- **HTML textareas submit CRLF.** Normalise newlines before storing source-ish
  content (see `DiagramManager._norm_source`), or the CRLF leaks into files and
  exports.
- **`frontmatter` strips the body's trailing newline** on load — re-normalise on
  read if you want write→read symmetry.
- **PlantUML needs a rendering server** (Java jar or an HTTP PlantUML server);
  there is no pure-Python renderer. That's why image export is Mermaid-only. To
  add it later, encode the source (deflate + PlantUML base64) and point at a
  self-hosted `plantuml/plantuml-server`.

## Conventions

- **Data directories are gitignored** (`posts/`, `notes/`, `books/`, `img/`, …)
  with a `.gitkeep`. New feature dirs need matching `.gitignore` entries.
- **Slugs**: `slugify(title, allow_unicode=True)`, deduplicated with a `-2`,
  `-3` suffix. Reserved slugs (e.g. `new`) get a `-1` suffix so they can't
  shadow a route.
- **Route ordering matters** in `main.py`: specific paths must be registered
  before catch-alls like `/books/{coll}/{chapter}`.
- **User-facing writes are destructive-safe**: managers return `None`/`False`
  for missing targets, and routes turn that into a `404`.
- Reports of completed work go to `./ai-scratch/reports/` — use the `report`
  skill.

---
> Source: [phatnguyen9722/knowledge_base_tool](https://github.com/phatnguyen9722/knowledge_base_tool) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-31 -->
