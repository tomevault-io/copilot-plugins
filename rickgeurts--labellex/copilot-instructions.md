## labellex

> A legal-data labelling app for producing training data for a legal LLM.

# LabelLex

A legal-data labelling app for producing training data for a legal LLM.
First concrete use case: upload bank prospectuses (typically 200–400 pages,
templated by magic-circle law firms) and label clauses for MREL eligibility
under the EU bank-resolution framework (BRRD/SRMR).

The app must show prospectuses with **original PDF layout intact** —
columns, indentation, footnotes — because legal layout carries semantic
meaning. The model loop (a local Ollama LLM) will eventually pre-label
clauses; humans correct; corrections feed the next training signal.

## Stack

- **Backend:** Python 3.11+, FastAPI, SQLAlchemy 2 (sync), SQLite (v0; Postgres later), pymupdf for PDF parsing.
- **Frontend:** React 18 + TypeScript + Vite, pdfjs-dist for PDF rendering.
- **LLM (planned):** local Ollama on `localhost:11434`, target hardware RTX 5070 Ti / 16 GB VRAM.

## Repo layout

```
backend/
  pyproject.toml
  app/
    main.py            FastAPI entrypoint (lifespan: create_all + migrations + seed)
    config.py          Settings (DB url, storage dir, default user/project, Ollama, LoRA Forge)
    db.py              Engine, SessionLocal, Base, get_db, lightweight ALTER-TABLE shim
    models.py          ORM models
    schemas.py         Pydantic request/response models
    seed.py            Idempotent seed: default user, project, starter labels
    services/
      pdf_parser.py        pymupdf → ParsedDocument (canonical text + bboxes)
      storage.py           File store for uploaded PDFs
      attributes.py        collect_effective_attributes + value-type validation
      label_counts.py      group-by helper for LabelOut.annotation_count
      document_activity.py touch_document + attach_annotation_counts (per-doc)
      ollama.py            sync httpx client, status() + generate_structured()
      structure_detector.py TOC outline → MREL section-type classification
      clause_discovery.py  per-page Ollama call → re-anchored verbatim quotes
      strategies/          Strategy interface + zero_shot impl + router stub
    routers/
      projects.py      /api/projects (CRUD)
      labels.py        /api/projects/:id/labels (CRUD)
      attributes.py    /api/labels/:id/attributes (CRUD)
      categories.py    /api/projects/:id/categories (CRUD)
      documents.py     Upload, list, get, PATCH (category), /pdf, /pages/:n
      annotations.py   POST/GET/PATCH/DELETE /api/annotations
      search.py        /api/documents/:id/search
      structure.py     /api/ollama/status, /api/documents/:id/detect-structure
      suggestions.py   /api/labels/:id/suggest-attributes,
                       /api/documents/:id/prelabel  (NDJSON stream),
                       /api/documents/:id/suggestions,
                       /api/suggestions/:id/accept|reject
      relations.py     /api/projects/:id/relation-defs (CRUD),
                       /api/relations (CRUD),
                       /api/documents/:id/relations (list)
      publish.py       /api/projects/:id/publish-to-lora-forge
                       (POST: bundle docs+annotations → sibling LoRA Forge)
  scripts/
    parse_pdf.py       Standalone parser test harness (run on a fixture PDF)

frontend/
  package.json
  vite.config.ts       Dev proxy: /api → http://127.0.0.1:8000
  src/
    main.tsx, App.tsx  App owns only the route table; nested layout in ProjectShell
    api.ts             Typed API client
    types.ts           Shared types (mirror backend schemas)
    pages/
      ProjectsListPage.tsx   /projects — list + create + delete projects
      ProjectShell.tsx       /projects/:id layout (sidebar + Outlet)
      ProjectPage.tsx        /projects/:id — drag-drop upload + document table
      ProjectSettingsPage.tsx /projects/:id/settings — document categories CRUD
      LabelsPage.tsx         /projects/:id/labels — labels admin
      DocumentViewer.tsx     /projects/:id/documents/:docId — PDF viewer
    components/
      PdfPage.tsx            pdf.js canvas + word-bbox overlay + label picker
      AnnotationListPanel.tsx Sticky right-side annotation list
    utils/
      spans.ts               lineRects, pageAnnotationSlice, effectiveAttributes
    styles.css
    vite-env.d.ts

storage/uploads/{id}.pdf       Uploaded PDFs (gitignored)
backend/labellex.db[-wal|-shm] SQLite + WAL sidecars (gitignored)
example_*.pdf                  Test fixtures (gitignored)
```

## Run

Backend (from repo root):

```
.venv\Scripts\python.exe -m uvicorn app.main:app --reload --app-dir backend --port 8000
```

Frontend:

```
cd frontend
npm install   # first time only
npm run dev   # opens on http://localhost:5173 (or 5174 if 5173 is taken)
```

Tables and the default user/project/labels are created on backend startup
(`lifespan` → `Base.metadata.create_all` → `seed`). Idempotent.

To re-parse the canonical fixture from the command line:

```
.venv\Scripts\python.exe backend\scripts\parse_pdf.py
```

## Architecture invariants

These are load-bearing decisions; check before reworking.

- **Backend is canonical for text + offsets.** pymupdf extracts per-word
  `(char_start, char_end, bbox, block, line)` and persists it. The frontend
  uses pdf.js for *visual rendering only* and pulls word data from
  `GET /api/documents/:id/pages/:n` so character offsets in annotations
  agree with what's stored.
- **Word bboxes are PDF native point coords (y-down from top-left).** The
  frontend overlays words at `bbox * scale` directly — no coordinate
  transform. Sample doc is A4 (595×842pt).
- **Annotations carry a span across pages.** Storage:
  `(start_page_num, start_char, end_page_num, end_char)`. Single-page
  spans set `start_page_num == end_page_num`. Char offsets are in each
  page's local text. The frontend resolves per-page slices via
  `pageAnnotationSlice` in `frontend/src/utils/spans.ts`. Re-labelling is
  still done by delete + create; span and attributes can be PATCHed (label
  is immutable post-create).
- **Single source of truth for labels.** A label's `project_id` must match
  the document's project — enforced in `routers/annotations.py`.
- **Selection model is drag-to-select on word overlays.** Cursor on word
  overlays is the I-beam (`cursor: text`). PointerDown on a word starts a
  cross-page drag tracked at viewer level; pointerEnter on subsequent words
  (across pages) extends it; document-level pointerup finalises and opens
  the picker. Selection / annotation / search highlights all render as
  one rectangle per visual line (book-marker style), built by
  `lineRects()` in `utils/spans.ts` — never per-word boxes.
- **Continuous-scroll viewer.** All pages stack vertically in
  `.viewer-pdf-area`. A single `IntersectionObserver` at the viewer level
  watches every page wrapper. Pages within a 2-page buffer of any visible
  page get the heavy render path (canvas + word overlays); other pages
  render only as a sized placeholder. Annotation / selection / search
  highlights render even on inactive pages so navigation feedback shows
  immediately.
- **Routing is project-scoped.** `/projects` is the picker; everything
  else nests under `/projects/:projectId/...` (`labels`, `settings`,
  `documents/:documentId`). `ProjectShell` reads `:projectId` from
  `useParams`, fetches the project, and renders the sidebar + nested
  `<Outlet>`. Pages read params themselves (no projectId props passed
  from App). The labels/settings pages read `refreshProject` from
  `useOutletContext` so sidebar state stays in sync after admin edits.
- **DB schema migrations are hand-rolled.** Until we adopt Alembic, new
  columns on existing tables go through `db.run_lightweight_migrations()`
  (called from lifespan). It checks `PRAGMA table_info` and runs ALTER
  TABLE + backfill if the column is missing. Idempotent. New tables come
  through `Base.metadata.create_all` as usual. Two columns are managed
  this way today: `Document.last_modified_at` and `Document.category_id`.
- **Document activity is tracked centrally.** `services/document_activity.touch_document()`
  bumps `Document.last_modified_at`; called from every annotation
  create/update/delete and from `accept_suggestion`. Single point of
  truth so the documents-table "last activity" column doesn't drift.

## Conventions

- **Imports:** stdlib → third-party → local, blank line between groups.
- **SQLAlchemy 2.0 style:** `Mapped[...]` + `mapped_column` everywhere.
- **Pydantic 2:** schemas inherit `_Base` with `from_attributes=True` so
  ORM models pass through directly.
- **Errors:** raise `HTTPException` from routers; let FastAPI surface them.
- **Frontend types** mirror backend schemas — keep `frontend/src/types.ts`
  and `backend/app/schemas.py` in sync.
- **TS strict mode is on** (incl. `noUnusedLocals`, `noUnusedParameters`).
- **No comments that describe what the code does.** Only WHY-comments where
  a hidden constraint or non-obvious choice would surprise a reader.

## Gotchas / quirks

- **Vite binds IPv6 by default on this Windows setup.** `localhost` and
  `[::1]` work; `127.0.0.1` does not unless you set `host: "127.0.0.1"`
  in `vite.config.ts`.
- **PowerShell 5.1 on Windows wraps native exe stderr in ErrorRecords**,
  which makes a clean `pip install` look like a failure. Verify with the
  follow-up import check, not with `$?`.
- **TOC dotted leaders** (e.g. `'PROGRAMME................................1'`) are
  captured by pymupdf as one giant "word." Filter when running structure
  detection — they're noise, not content.
- **PDF page index ≠ printed page number.** Front matter pushes the offset
  (cover + roman-numeralled TOC). `Page.printed_page_num` is reserved for
  the printed value but not yet populated.
- **The annotation overlay is `pointer-events: auto`** so users can click
  to delete; this means it can swallow clicks on words it covers. Keep an
  eye on this when adding selection features.
- **SQLite is in WAL mode.** Connect-time PRAGMAs in `db.py` set
  `journal_mode=WAL` and `busy_timeout=5000`. Two consequences: (1)
  sidecar files `labellex.db-wal` and `labellex.db-shm` appear on disk —
  gitignored as `*.db-wal` / `*.db-shm`; never delete them while the
  server is running, the journal contains uncommitted state. (2)
  Contended writes wait up to 5 s for the lock instead of erroring
  instantly. WAL was added because long-running streaming endpoints
  (pre-label scans) used to starve concurrent reads.
- **Don't put side effects inside `setState` updaters.** React StrictMode
  invokes `(prev) => ...` callbacks twice in dev to surface impure code.
  An `api.x(...)` call inside one fires the request twice — and since
  SQLite is single-writer, the second collides on the lock. Pattern: read
  state from closure, set running=true with a pure updater, then fire
  the side effect at the top level of the handler.
- **`uvicorn --reload` sometimes hangs on file changes** while a
  streaming request is in flight (or sometimes for no obvious reason).
  Symptom: log says "WatchFiles detected changes... Reloading" but no
  second "Started server process" appears, and the running worker still
  serves stale code. Fix: kill the parent reloader + worker PIDs and
  restart uvicorn cleanly. Do this any time a route looks like it didn't
  pick up a backend edit.

## Status

- ✅ v0 spike: upload PDF, render with original layout, click-words to
  label, persist, reload-restores. Validated on a 254-page real bank
  prospectus.
- ✅ Hierarchical labels (`parent_id` self-FK on `LabelDefinition`),
  cycle-safe parent validation, admin CRUD UI at `/labels`, hierarchical
  label picker in the viewer, indented sidebar tree.
- ✅ Typed attributes per label (`AttributeDefinition` + `AnnotationAttribute`).
  Types: string / enum / bool / number / date. Required-attribute
  enforcement on annotation create. Inheritance: an annotation tagged with
  a descendant label can carry values for any attribute defined on itself
  or any ancestor — backend resolves via `services/attributes.collect_effective_attributes`,
  frontend mirrors via `effectiveAttributes(label, labelById)` in
  `PdfPageView`. Admin authors attributes per label at `/labels`; the
  annotation picker becomes a two-step flow (label → attribute form) when
  the picked label has any effective attribute.
- ✅ Annotation editing in the viewer: click an existing annotation overlay
  → popover opens with editable typed-attribute inputs (own + inherited),
  delete / cancel / save buttons. Required attributes still gate save.
  Backend exposes `PATCH /api/annotations/{id}` accepting `{attributes}` —
  it replaces the annotation's attribute set wholesale (label and span are
  immutable post-create — re-label by deleting and creating). Annotation
  overlays only intercept clicks when no picker is in progress, so word
  selection still works.
- ✅ Drag-to-select on word overlays replaces the old click-first /
  click-last model. Selection state lives at the viewer level so drags
  span pages; pages activate via IntersectionObserver as the user scrolls,
  word overlays come online and pointerEnter continues to extend the
  selection. Pointer-up anywhere finalises and opens the picker at the
  release position. Highlights are line-based rectangles (book-marker),
  yellow during drag, label-coloured for annotations, yellow + outline
  for search hits.
- ✅ Continuous-scroll PDF viewer with toolbar slider. Bulk
  `GET /api/documents/:id/pages` loads all pages with words in one shot
  (~20 MB raw / ~3 MB gzipped for the 254-page reference) so cross-page
  selection has the data it needs. Page-input, slider, prev/next, and the
  arrow keys all call into `scrollToPage(n)` which scrolls the page
  wrapper into view. Most-visible page drives `currentPage` state.
- ✅ Edit-span on existing annotations: the editor popover gains an "edit
  span" button that closes the editor and enters resize mode (yellow
  banner across the top). The next drag PATCHes the annotation's span
  fields preserving label + attributes. Esc cancels.
- ✅ Arrow-key page navigation (`←` / `→`) on the document viewer.
  Suppressed when focus is in any `INPUT` / `TEXTAREA` / `SELECT` /
  contenteditable element, and when modifier keys are held.
- ✅ Document-wide text search (backend: `GET /api/documents/:id/search?q=…`,
  case-insensitive substring on `Page.text`, returns `SearchHit { page_num,
  char_start, char_end, snippet, match_in_snippet }` — frontend wraps the
  match in `<mark>`). UI is a magnifying-glass toggle in the toolbar; query
  is debounced 250 ms; clicking a hit navigates and centres a transient
  yellow highlight on the matched words via the same scroll mechanism as
  annotation jump (separate `[data-search-anchor]` element). The highlight
  persists until the user closes the search bar.
- ✅ Annotation list panel on the document viewer: a sticky right-side
  panel listing every annotation in the document, with text search,
  filter-by-label, sort by page / label / newest, click-to-jump. The
  current page's items are highlighted. Toggle via "Hide list" / "Show
  list" in the toolbar; preference persists in `localStorage` under
  `labellex.viewer.showPanel`. Click-jump also **scrolls the bbox into
  view**: `PdfPageView` accepts a `scrollToAnnotationId` prop and centres
  the matching annotation via `scrollIntoView` once it's rendered on the
  current page (the first overlay div of each annotation carries
  `data-annotation-id` for the lookup). Caller clears the request via
  `onScrollHandled`.
- ✅ `/labels` admin polish: edit-as-modal with embedded attribute manager
  (own + read-only inherited shown), per-row checkbox + bulk delete
  (post-order so parent+children selections work), search box (filters
  tree, keeps ancestors of matches), collapse/expand chevrons persisted in
  `localStorage` under `labellex.labels.collapsed`, 14-colour preset
  palette in `ColorInput`, and inline rename via double-click on the row's
  name (Enter saves, Esc cancels, blur saves if non-empty). Annotation
  usage counts surfaced from backend (`LabelOut.annotation_count`,
  computed via group-by in `services/label_counts.attach_annotation_counts`).
- ✅ Multi-project (single-user). `/projects` lists, creates, deletes
  projects; `Project.name` is unique per user. New projects start empty —
  the admin defines labels (under `/projects/:id/labels`) and document
  categories (under `/projects/:id/settings`) before uploading. Sidebar
  shows the current project name with a `← Switch project` link.
- ✅ Document table view at `/projects/:id` replacing the v0 list.
  Columns: filename + page count, category (inline `<select>` to
  assign/change), status pill (labelled / unlabelled / parsing / parse
  failed), annotation count, last modified. Default sort is server-side
  by `last_modified_at` desc (so docs you just touched bubble up).
  `attach_annotation_counts` in `services/document_activity` powers the
  annotation column via a single GROUP BY per request.
- ✅ Drag-and-drop bulk PDF upload above the document table. Drop zone
  highlights on dragover; non-PDF files are filtered with a count of
  skipped files surfaced. Uploads run **sequentially** (pymupdf parsing
  is CPU-bound; parallel hurts) with per-file status queued → uploading
  → done | failed. Failed uploads show the error inline; "clear" wipes
  the finished-uploads list once all are settled.
- ✅ Document categories (per-project). `DocumentCategory` table scoped
  to a project; documents carry an optional `category_id` FK that nulls
  on category delete (handled in router code, since SQLite FK
  enforcement is off by default). Admin authors categories at
  `/projects/:id/settings` (name, description, color preset). Inline
  picker on the documents table assigns/changes per row, persisting via
  `PATCH /api/documents/:id`.
- ✅ Inter-annotation relations. Two tables: `RelationDefinition`
  (admin-authored type per project) and `AnnotationRelation` (directed
  link between two annotations on the **same** document). Same-document,
  no-self-loops, unique-(from, to, type) all enforced in
  `routers/relations.py`. Annotation delete wipes its relations
  explicitly (SQLite FK enforcement is off on this engine). Endpoints:
  `/api/projects/:id/relation-defs` CRUD, `/api/relations` CRUD,
  `/api/documents/:id/relations` list. `/projects/:id/settings` is
  refactored around a generic `TaxonomySection` component composing the
  categories card and the new relation-types card. Document viewer:
  editor popover gains a Relations section (outgoing + incoming) with
  a clickable target snippet that jumps + scrolls, and an × to remove.
  "+ Link" toolbar button enters linking mode (top banner explains the
  flow, word-dragging suppressed) — clicking another annotation opens a
  relation-type picker at the click position. Esc cancels.
- ✅ Publish to sibling **LoRA Forge** instance (`routers/publish.py`).
  `POST /api/projects/:id/publish-to-lora-forge` bundles every document
  in the project — full page-joined text plus `{label, text, startPage,
  endPage, attributes{name:value}}` per annotation — and POSTs the
  payload (`{source:"labellex", schemaVersion:1, project, exportedAt,
  taskType:"clause_extractor", documents}`) to LoRA Forge's webhook.
  Returns LoRA Forge's dataset response plus a summary
  (`totalDocuments`, `documentsWithLabels`, `annotations`) for the UI.
  `httpx.RequestError` → 502 with a hint pointing at
  `LABELLEX_LORA_FORGE_WEBHOOK_URL`; any 4xx/5xx from LoRA Forge → 502
  with the upstream body (truncated to 500 chars). Documents with zero
  annotations are still included in the payload so LoRA Forge can report
  coverage gaps. Frontend: "Publish to LoRA Forge" button on
  `/projects/:id` next to the document count, disabled until ≥1
  annotation exists; success renders a green banner with the new
  dataset id and row count, failure renders `error-banner`.
  Settings: `LABELLEX_LORA_FORGE_WEBHOOK_URL` (default
  `http://127.0.0.1:8001/datasets/labellex-webhook`),
  `LABELLEX_LORA_FORGE_TIMEOUT_SECONDS` (default 30). The webhook
  contract is one-way and idempotent on the LoRA Forge side —
  re-publishing the same project replaces the previous dataset row.
- ⬜ Multi-user (auth, sessions, roles, per-document checkout). Projects
  exist but everything still attributes to `settings.default_user_id`.
- ⬜ Postgres + docker-compose.
- ✅ Ollama integration v0: structure detection.
  - `app/services/ollama.py` — sync httpx client. `status()` is non-raising
    (returns `{"reachable": false, "error": ...}` on failure so the UI can
    show it). `generate_structured(prompt, schema)` posts to
    `/api/generate` with the JSON schema in `format=` and parses the
    response (Ollama 0.5+ structured output).
  - `app/services/structure_detector.py` — extracts the document outline
    via `pymupdf` (`doc.get_toc`), falls back to a regex-driven parse of
    the printed "TABLE OF CONTENTS" page; classifies each entry into
    MREL-relevant section types (`tc_senior_preferred`,
    `tc_senior_non_preferred`, `tc_subordinated`, `risk_factors`, etc.)
    via Ollama with a fixed enum schema.
  - `GET /api/ollama/status` — reachability + configured model + locally
    pulled models.
  - `POST /api/documents/:id/detect-structure` — runs the pipeline and
    returns `{model, sections: [{title, page_num, section_type,
    confidence}]}`. 503 if Ollama unreachable or the configured model
    isn't pulled (with a one-line `ollama pull …` hint).
  - Frontend: 🪄 Detect structure button in the viewer toolbar opens a
    modal showing the classified outline with jump-to-page actions.
  - Settings: `LABELLEX_OLLAMA_BASE_URL` (default
    `http://localhost:11434`), `LABELLEX_OLLAMA_MODEL` (default
    `qwen2.5:14b-instruct`), `LABELLEX_OLLAMA_TIMEOUT_SECONDS` (default
    180). Set up Ollama with: install (`winget install Ollama.Ollama`),
    pull the model (`ollama pull qwen2.5:14b-instruct`), and the daemon
    auto-starts on Windows.
- ✅ Ollama: pre-labelling clauses for MREL features (clause discovery v1).
  - `app/services/clause_discovery.py` — per-page Ollama call with a JSON
    schema constraining the response to `{candidates: [{label, quote}]}`,
    where `label` is enum-bound to the names of the labels in scope. Each
    returned quote is re-anchored to the page's text via exact substring
    match → whitespace-tolerant fallback (pages have hard line breaks
    the model often normalises) → fuzzy head-and-tail anchor that
    recovers candidates the model paraphrased in the middle.
    Punctuation-trim + lowercase comparison absorbs small wrappings
    (smart quotes, trailing periods).
  - System prompt is explicit about **what counts as a clause** and
    **what to skip** (headings, definitions, Risk-Factor /
    Use-of-Proceeds / Taxation boilerplate, cover-page summary tables)
    — empirically the biggest source of false positives.
  - Seeded MREL labels in the default project carry richer descriptions
    referencing concrete legal markers (rank pari passu, Article 108
    BRRD, etc.) and explicit negatives. The seed upgrader replaces v0
    stub descriptions only when the stored description still matches
    the v0 literal — admin edits are preserved.
  - `POST /api/documents/{id}/prelabel` is a **streaming NDJSON endpoint**.
    Body `{start_page_num, end_page_num, label_definition_ids?}`. Yields
    one event per line:
    `{"type":"started","model","total_pages"}`,
    `{"type":"page_done","page_num","pages_done","pages_total","candidates":[…]}`,
    `{"type":"done"}` or `{"type":"error","message"}`. Per-page commits
    mean candidates from completed pages stay durable even if the scan
    bails mid-way. Pre-flight failures (404/400/503) still raise as
    HTTPException before the stream starts.
  - `GET /api/documents/{id}/suggestions?status=pending` — re-fetch
    pending candidates so the modal is resumable across sessions.
  - `POST /api/suggestions/{id}/accept` — promotes the suggestion to a
    real `Annotation` (deliberately bypasses required-attribute
    validation; the labeller fills attrs in the editor after accepting).
    Bumps `Document.last_modified_at`.
  - `POST /api/suggestions/{id}/reject` — flips status to `rejected`.
  - Frontend: 🪄 Pre-label toolbar button → modal with page-range inputs
    + label checkboxes (default scope = leaf labels). While scanning, a
    **live progress bar** (`<progress>` driven by `pages_done/pages_total`)
    updates per page; new candidates stream into the review list as they
    arrive. Each candidate row: snippet, label badge, page, jump / reject
    / accept. Jump uses the existing search-highlight mechanism (yellow
    line rectangles) and keeps the modal open. Pending suggestions are
    seeded from the server so review can resume across sessions.
  - Per-page chunking trade-off: clauses straddling a page break get
    split into per-page candidates — labeller can extend via "edit
    span" once accepted.
- ⬜ Ollama: few-shot strategy (use accepted annotations as in-context
  examples) and a per-(label, attribute) accuracy scoreboard surfacing
  what zero-shot vs. few-shot is doing on each slot.
- ⬜ Printed-page-number extraction.

---
> Source: [RickGeurts/LabelLex](https://github.com/RickGeurts/LabelLex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-07 -->
