## paper-assistant

> AIRA analyzes research papers for integrity and reproducibility signals. Upload a PDF and it extracts text natively via pypdf (OCR via Tesseract + Poppler is available as a separate `/api/parse_pdf_ocr` endpoint, not an automatic fallback on the ingest path), verifies every citation against the Crossref database with confidence scoring, extracts statistical indicators (p-values, sample sizes, effect sizes, confidence intervals), detects in-text citation markers, and returns a structured `IntegrityReport` with a heuristic integrity score (0–100, letter-grade A–F) based on open-science transparency markers.

# AIRA — AI Research Assistant

AIRA analyzes research papers for integrity and reproducibility signals. Upload a PDF and it extracts text natively via pypdf (OCR via Tesseract + Poppler is available as a separate `/api/parse_pdf_ocr` endpoint, not an automatic fallback on the ingest path), verifies every citation against the Crossref database with confidence scoring, extracts statistical indicators (p-values, sample sizes, effect sizes, confidence intervals), detects in-text citation markers, and returns a structured `IntegrityReport` with a heuristic integrity score (0–100, letter-grade A–F) based on open-science transparency markers.

> **Product strategy:** the reframed product vision, market-research question list, UI-redesign scope, and rescoped roadmap live in [VISION.md](VISION.md) (2026-07-22). This file stays focused on technical architecture.

## Architecture

```
User → React frontend (Vite)   port 5173 ──┐
User → Streamlit app_v2.py     port 8501 ──┼──▶ FastAPI app/ package   port 8000 ──▶ Crossref API
                                           │      via backend.py (3-line shim)   ──▶ OpenAlex API
                                           └───────────────────────────────────────▶ SQLite data/aira.db + PDFs data/pdfs/
```

### Key files

| File | Role |
|------|------|
| `README.md` | Public-facing project README (description, features, architecture, setup, API reference, testing, design decisions, limitations) — the front door for external readers |
| `backend.py` | 3-line shim — `from app.main import app`; entry point for uvicorn and CI |
| `app/` | FastAPI package — `main.py` (lifespan + CORS), `config.py` (env vars + constants), `routers/` (health, pdf, analysis, papers, projects), `services/` (pdf, references, citations, statistics, integrity, embeddings, ingest), `schemas/models.py`, `utils/helpers.py`, `db/` (engine + 6 ORM models) |
| `app/db/models.py` | SQLAlchemy ORM: Paper, AnalysisReport, Reference, Project, ProjectPaper, Embedding |
| `frontend/` | React + Vite + Tailwind + shadcn/ui — primary UI |
| `app_v2.py` | Legacy Streamlit frontend — calls FastAPI backend; still functional, secondary path |
| `app_legacy.py` | Original standalone Streamlit app (pre-API architecture, OpenAI-dependent, no Crossref, no OCR). Superseded by `app_v2.py` and the React frontend; kept for reference. |

## Prerequisites (one-time)

```bash
# macOS
brew install tesseract poppler

# Ubuntu/Debian
sudo apt-get install tesseract-ocr poppler-utils
```

## Local Development

### Venv setup

```bash
python3.11 -m venv .venv --clear
source .venv/bin/activate
pip install -r requirements.lock.txt        # reproducible pinned install
# (or `pip install -r requirements.txt` for a loose latest-compatible install)
```

Lockfiles (`requirements.lock.txt`, `requirements-dev.lock.txt`) are compiled with pip-tools and are what CI installs. After editing `requirements*.txt`, regenerate:

```bash
pip-compile --strip-extras --no-header -o requirements.lock.txt requirements.txt
pip-compile --strip-extras --no-header -o requirements-dev.lock.txt requirements.txt requirements-dev.txt
```

### Backend

```bash
source .venv/bin/activate
uvicorn backend:app --reload --port 8000
# Verify: curl http://localhost:8000/healthz
```

### React frontend (primary)

```bash
cd frontend
npm install
npm run dev        # → http://localhost:5173
```

### Streamlit frontend (legacy)

```bash
source .venv/bin/activate
streamlit run app_v2.py    # → http://localhost:8501
```

### Tests

```bash
pip install -r requirements-dev.txt
pytest tests/ -v
```

### Docker (backend only)

```bash
docker compose up --build
# First start downloads the SPECTER model (~500 MB) into ./data/hf-cache;
# DB, PDFs, and model cache all persist via the single ./data bind mount.
# Build is CPU-torch pinned to the lockfile version (see Dockerfile note).
```

## Environment Variables

Copy `.env.example` to `.env`:

| Variable | Required | Default | Notes |
|----------|----------|---------|-------|
| `CROSSREF_EMAIL` | **Yes** | `test@example.com` | Sent in Crossref User-Agent header for API politeness |
| `OPENAI_API_KEY` | No | — | AI summary in `app_v2.py` only; `backend.py` does not use it |
| `MAX_PDF_SIZE_MB` | No | `10` | |
| `MAX_PDF_PAGES` | No | `100` | |
| `MAX_OCR_PAGES` | No | `20` | |
| `CROSSREF_CONCURRENT_REQUESTS` | No | `3` | Semaphore limit for concurrent Crossref calls |
| `OPENALEX_MAILTO` | No | `CROSSREF_EMAIL` | Contact email for OpenAlex polite pool |
| `OPENALEX_CONCURRENT_REQUESTS` | No | `3` | Semaphore limit for concurrent OpenAlex calls |
| `POPPLER_PATH` | No | — | Windows only: path to poppler `bin/` directory |
| `ALLOWED_ORIGINS` | No | localhost 5173/3000/8501 | Comma-separated CORS origins; `*` allowed but disables credentials (browsers reject wildcard+credentials) |
| `USE_SEMANTIC_MATCHING` | No | `1` | Set to `0` to fall back to difflib `title_similarity()` for citation scoring |

## API Endpoints

Base URL: `http://localhost:8000`

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/healthz` | Health check |
| GET | `/version` | Dependency version info |
| POST | `/api/parse_pdf` | Extract text from PDF (native pypdf) |
| POST | `/api/parse_pdf_ocr` | Extract text via OCR (Tesseract, for scanned PDFs) |
| POST | `/api/verify_references` | Verify reference list against Crossref (fuzzy + DOI matching) |
| POST | `/api/find_intext_citations` | Extract in-text citation markers |
| POST | `/api/extract_statistics` | Extract p-values, sample sizes, effect sizes, CIs |
| POST | `/api/truthiness_score` | Compute heuristic integrity score |
| POST | `/api/analyze_pdf` | Full pipeline — combined endpoint used by the frontend |
| POST | `/api/papers` | Ingest PDF — SHA256 dedup, persist Paper + Report + References + Embedding; 201 new / 200 duplicate |
| GET | `/api/papers` | List papers — pagination envelope, optional `project_id` filter, latest report score, `has_embedding` |
| GET | `/api/papers/{id}` | Paper detail + stored `IntegrityReport` (parsed from latest report) |
| GET | `/api/papers/{id}/similar` | Cosine similarity over stored SPECTER vectors (`top_k`, optional `project_id` scope) |
| GET | `/api/papers/{id}/related` | OpenAlex `related_works` (cached 24h, polite `mailto`); 200+`reason` for data conditions, 503 when OpenAlex is down |
| GET | `/api/projects` | List all projects with paper counts |
| POST | `/api/projects` | Create project |
| GET | `/api/projects/{id}` | Get project with paper_ids |
| PUT | `/api/projects/{id}` | Update project name/description |
| DELETE | `/api/projects/{id}` | Delete project (cascades ProjectPaper links) |
| POST | `/api/projects/{id}/papers` | Add paper to project |
| DELETE | `/api/projects/{id}/papers/{paper_id}` | Remove paper from project |

## Git Conventions

- **No AI attribution in commits.** Do not add `Co-Authored-By: Claude` or any AI co-author trailer to commit messages.
- Commit messages should be imperative-mood, concise, and describe the *why* not just the *what*.
- Do not commit `.venv/`, `venv/`, `.env`, or `node_modules/`. These are in `.gitignore`; if they appear tracked, run `git rm -r --cached <path>`.

## Roadmap / Backlog

### Done
- **`asyncio.gather()` parallelization** — all four sub-analyses run concurrently in `app/services/ingest.py` and `app/routers/analysis.py`
- **Regex precompilation** — `SECTION_PATTERNS` compiled once at module level in `app/services/pdf.py`, not per request
- **`requirements.txt` security updates + venv rebuild** — pypdf → 6.6.0, python-multipart → 0.0.18, fastapi → 0.128.0. Installed and confirmed.
- **O(n²) string concat fix** in `parse_reference_list` (`app/utils/helpers.py`) — replaced loop concatenation with list accumulation + `" ".join()`
- **Test skeleton + CI** — `tests/test_backend.py` (17 tests) + `.github/workflows/test.yml`
- **`extract_title_from_ref()` fix** (`app/utils/helpers.py`) — old regex captured the author list (everything before the first period) instead of the paper title. New function handles APA-style `(YEAR). Title.` and Vancouver-style `Author. Title. Journal.` patterns.
- **SPECTER semantic embeddings** for citation matching — `app/services/embeddings.py`; replaced `difflib.SequenceMatcher` in `verify_references()` with `allenai-specter` (sentence-transformers). Encodes all Crossref candidates for a reference in a single batched forward pass. Controllable via `USE_SEMANTIC_MATCHING` env var (default `1`). Old `title_similarity()` kept as fallback. Model loads at startup via lifespan hook.
- **Week 1 Foundation** — package restructure into `app/`; SQLAlchemy 6-table schema + Alembic initial migration; `POST /api/papers` ingest pipeline (SHA256 dedup, PDF storage, Paper/Report/Reference/Embedding persistence; 768-dim SPECTER vector confirmed in DB); projects CRUD (7 endpoints). 50 tests green.
- **Track A / A0 artifacts (2026-07-22, awaiting owner checkpoint — do not start A1 until reviewed)** — `frontend/DESIGN.md` (references + direction + open questions), semantic status/grade tokens (CSS vars light+dark, Tailwind-mapped), ThemeProvider active (default light), TypeScript scaffold (strict tsconfig, `npm run typecheck`), standalone dev routes `/styleguide` (live tokens) and `/styleguide/paper` (the one mocked screen). Production pages untouched.
- **Pre-gate hardening (2026-07-22, VISION.md §3 shortlist)** — CORS defaults to explicit local origins (wildcard now disables credentials); pip-tools lockfiles pinned to the tested venv + dependabot (pip/npm/actions) + CI installs from lock; backend Dockerfile + compose (CPU torch, Alembic migrate on start, single `./data` mount — **not build-verified: no Docker on the dev machine**).
- **SPECTER encode moved off the event loop** — `model.encode()` calls in `app/services/ingest.py` (`_try_store_embedding`, now async) and `app/services/references.py` (`_title_sims` call sites) run via `asyncio.to_thread`; the DB session never crosses the thread boundary. Concurrency stays bounded by `CROSSREF_SEMAPHORE` on the verification path.
- **Week 2 Library UI + discovery backend** — paper read endpoints (`GET /api/papers` list with pagination envelope + latest-report join, `GET /api/papers/{id}` detail with parsed stored report); ingest metadata enrichment (title + DOI, see backlog note); `/similar` (cosine over stored SPECTER vectors) and `/related` (OpenAlex, cached, semaphore + retry mirroring the Crossref client); React Query frontend with `/library`, `/projects/:id`, `/papers/:id` reusing the dashboard components via the extracted `reportTransform.js`; **upload flow now ingests** (`POST /api/papers`) instead of the stateless `/api/analyze_pdf`, so analyzed papers persist to the library. Also fixed en route: the reference-verification DOI regex was broken (never matched — DOI branch was dead code) and the resurrected fast path is now guarded by title agreement. 88 tests green.

### Prioritized backlog (assessed 2026-08-03)

Ranked by value-to-effort (best first). Type ∈ correctness / scale / polish; effort in
minutes / hours / days. Verified first-hand against the code on 2026-08-03; supersedes the
old two-bullet "Planned next" (the title-heuristic and picker items are folded in as rows
6→#8 and 8→DOI-enrichment below).

> **Verified 2026-08-03 — already complete, do not re-raise:** the DOI-verification fast
> path is guarded by title agreement (`app/services/references.py:96-106`) and is exercised
> by tests (`tests/test_references.py::test_doi_match_with_title_agreement_is_verified` and
> `::test_wrong_but_resolvable_doi_is_not_verified`).

| # | Item | Type | Effort | What it buys (and rabbit-hole warnings) |
|---|------|------|--------|------------------------------------------|
| 1 | **Guard the integrity score on failed extraction.** When native extraction yields `<100` chars (`extraction_method=="failed_needs_ocr"`, `app/services/ingest.py:123-132`), emit a null/`needs_ocr` grade instead of the current **65/C**. Ingest never runs OCR (only the separate `/api/parse_pdf_ocr` endpoint does), so a scanned PDF is silently graded on empty text. | correctness | hours | Stops the tool silently mis-grading unreadable PDFs — the worst failure mode for an integrity tool. **Rabbit hole:** don't instead try to wire OCR into ingest (page caps, slow Tesseract path) — that's days; do the guard. |
| 2 | **Add `pytest-cov` + coverage in CI.** No coverage tooling today (not in dev lockfiles; no pytest config). | polish | 30–60 min | Reveals the untested core (row 3); a portfolio signal. |
| 3 | **Unit-test the scoring core.** `integrity.py`, `statistics.py`, `pdf.py`, `citations.py` and the pdf/analysis routers have no direct unit tests. | correctness | 2–3 h | Locks the scoring logic; a regression net; would have caught row 1. |
| 4 | **Fix the "AI Summary" mislabel.** `SummaryAccordion.jsx` is labelled "AI Summary" but renders parsed report data, not LLM output. | polish | minutes | Honesty in an integrity-branded UI. |
| 5 | **Cap ingest embedding concurrency.** `_try_store_embedding` (`ingest.py:219`) calls `asyncio.to_thread(model.encode,…)` with no semaphore (the verification path is correctly bounded by `CROSSREF_SEMAPHORE`; this is the ingest path). | scale | 30–60 min | Cheap OOM insurance under concurrent uploads; low probability on a single-user tool. |
| 6 | **Log instead of silently swallowing corrupt report JSON.** `ingest.py:81-84` does `except Exception: pass` on the dedup path (the router path logs a warning). | polish | minutes | Surfaces DB corruption instead of hiding it. |
| 7 | **Benchmark SPECTER vs a general-purpose embedding model.** No benchmark artifacts exist. | validation | hours–1 day | Defensibility of the model choice — a real talking point for ML/PI readers. |
| 8 | **DOI-based metadata enrichment** (folds in the old title-heuristic note). Look up title/authors/year via Crossref/OpenAlex by DOI. Today `_extract_title` (`ingest.py:35-58`) is a first-line/filename heuristic and `authors`/`year` are never populated at ingest. | correctness | hours (DOI lookup) | Real titles + populated authors/year on every paper. **Rabbit hole:** the GROBID variant is a new heavyweight service dependency — days. |
| 9 | **Validate the integrity score against ground truth.** No labelled set, no inter-rater comparison — the score is an unvalidated heuristic (as the disclaimer states). | correctness | days | Turns "toy heuristic" into a defensible signal — highest strategic value. **Rabbit hole:** unbounded; needs a labelled corpus + methodology, can eat weeks. |
| 10 | **Content-aware dedup.** SHA-256 hashes raw PDF bytes (`ingest.py:68`), so the same paper from two sources (re-encoded bytes) becomes two library entries. | correctness / scale | days | One library entry per paper regardless of source bytes. **Rabbit hole:** "just change the hash" hides a dedup-key migration, DOI-less-paper handling, and backfill of existing rows. |
| 11 | **Paginated / "papers-not-in-project" picker** (folds in the old picker note). `ProjectDetail.jsx` → `getAllPapers` loads the whole library; backend has only an in-project filter. | scale | half-day | Picker scales past a few hundred papers; low current impact. |
| 12 | **Resolve the Smart Summary parity gap.** `app_v2.py:462-543` has a GPT-3.5 map-reduce summary the React UI lacks; no backend LLM endpoint exists. **Recommendation: retire** the Streamlit-only feature. | polish | minutes to retire | Removes a confusing parity gap. **Rabbit hole:** *porting* it = a key-gated backend LLM endpoint + UI + cost controls — a project, not a task. |
| 13 | **Build-verify the Docker setup.** Authored without a Docker daemon on the dev machine. | polish | 1–2 h *if* Docker available | Confirms the reviewer setup path. **Rabbit hole:** no local Docker → can't verify without new infra. |

**If I had two hours next, I'd do row 1** (guard the score on failed extraction). It removes
a live, demoable, user-facing wrong output — a confident "C" on a scanned PDF AIRA never
read — which for an *integrity* tool costs more credibility than the missing coverage number
(row 2, the runner-up). Row 3 then locks the fix with a test.

### Deferred (documented — stubbed as "Coming Soon" in the UI)

The following are already wired into the frontend as disabled/placeholder elements (`FeatureGrid.jsx`, `ContentTypeSelector.jsx`, `UrlInput.jsx`). Captured here so they don't get re-discovered from scratch:

| Feature | Where stubbed | Notes |
|---------|---------------|-------|
| Frontend memoization / React Query / virtualization | `CitationTable.jsx`, `airaApi.js` | `useMemo`/`useCallback`, `@tanstack/react-query`, `@tanstack/react-virtual` |
| OpenAlex related-papers | `app_v2.py:563` | Natural follow-on to SPECTER2; OpenAlex has 250M+ scholarly works via free API |
| AI Research Chat | `FeatureGrid.jsx` (`comingSoon: true`) | Ask questions about the analyzed paper |
| Synthesis Matrix | `FeatureGrid.jsx` (`comingSoon: true`) | Multi-paper side-by-side comparison + literature review |
| URL-based input (arXiv links) | `UrlInput.jsx` | Analyze by URL rather than file upload |
| News article analysis | `ContentTypeSelector.jsx`, `Home.jsx:200` | Different content-type path, UI button disabled |
| Citation network visualization | `app_v2.py:563` | |
| Paper library with search | `app_v2.py:563` | |

## Long-Term Vision / Roadmap

> **Superseded 2026-07-22:** this section is historical. [VISION.md](VISION.md) now governs
> product strategy and sequencing — the Week 3–5 plan below is paused (Week 3 Discovery UI
> explicitly skipped). The tactical "Roadmap / Backlog" section above remains valid.

> Added 2026-07-09 after a planning session. This section is the product trajectory; the
> "Roadmap / Backlog" section above remains the near-term tactical list. Full plan detail
> lives in the session plan; this is the durable summary future sessions should work from.

### Vision

Evolve AIRA from a stateless citation checker into a **lightweight research workspace**
(think Elicit/Zotero/Scite hybrid) where paper integrity checking is one intelligence
layer, not the whole product. Papers persist in a library, are organized into projects,
and cross-paper intelligence (similarity, conflicting findings, later chat/takeaways)
operates over that library.

**Context driving scope:** ~4–5 focused weeks available (until early-to-mid August 2026),
then development continues at a much slower pace. This is a portfolio piece for
bioinformatics/AI-ML roles — technical depth and defensibility beat feature count.
2–3 things built solidly beat 8 things half-built.

### Locked decisions (with rationale)

1. **Persistence: SQLite via SQLAlchemy + Alembic.** Single-file DB (`data/aira.db`),
   uploaded PDFs at `data/pdfs/<sha256>.pdf`, both gitignored. Zero-setup for reviewers;
   SQLAlchemy keeps a Postgres migration cheap if ever needed. Alembic from day one
   because the schema will keep evolving during the slow-pace phase.
2. **Vector search: brute-force numpy cosine over BLOB-stored vectors.** At library scale
   (10²–10³ papers, 768-dim SPECTER) this is milliseconds and dependency-free.
   sqlite-vec/FAISS is the documented upgrade path past ~10k papers. Deliberate,
   defensible engineering choice.
3. **Local-first AI.** Core intelligence (SPECTER embeddings, NLI conflict flagging) runs
   keyless on local models. LLM-API features (takeaways, chat, writing help) are optional,
   activate only when a key is configured, and land post-window.
4. **Single-user local tool.** No auth/multi-user, indefinitely. News analysis stays stubbed.

### Core data model

| Entity | Key fields | Built when |
|--------|-----------|------------|
| `Paper` | sha256 (dedup), title, authors, year, doi, pdf_path, full_text, sections, counts, extraction_method | Foundation |
| `AnalysisReport` | paper_id, report_json (full IntegrityReport), integrity_score/grade (promoted columns), **analyzer_version** | Foundation |
| `Reference` | paper_id, raw_text, doi, matched_title, status, confidence — normalized out of the report (enables citation graph later) | Foundation |
| `Project` + `ProjectPaper` m2m | name, description; papers can be in many projects | Foundation |
| `Embedding` | paper_id, kind (`title_abstract` now; `chunk` later for RAG), vector BLOB | Foundation |
| `StatClaim` | paper_id, claim_type (p_value/effect_size/sample_size/ci), raw_text, parsed_value, **context_sentence**, section | Conflict detection |
| `ConflictFlag` | project_id, claim_a/claim_b, relation, nli_confidence, status (open/reviewed/dismissed) | Conflict detection |
| `Annotation` | paper_id, anchor (page + text quote/offsets), content, color | Future |
| `Finding` | curated note linking papers/claims within a project | Future |

### Hard forks (done in foundation) vs additive (later, no rework)

**Forks:** stateless→persistent ingest pipeline; `backend.py` (1,013 lines) → `app/`
package (routers/ + services/ + db/models/schemas, with `backend.py` kept as a 3-line
shim so `uvicorn backend:app`, CI, and `app_v2.py` keep working); `analyzer_version` on
reports from day one; `extract_statistics` context-sentence capture (touches the core
response contract — additive `claims` field alongside existing string lists).

**Additive later:** annotations + PDF viewer (react-pdf), RAG chat (chunk embeddings),
LLM takeaways/writing help, citation-network viz (enabled by normalized `Reference`),
URL/arXiv ingestion, BibTeX export (port from `app_v2.py:240`), FTS5 full-text search,
tags/reading status, Zotero import.

**Minimum foundational layer that unlocks the most:** DB + ingest persistence + package
split + stored embeddings + projects. Every wishlist feature then becomes "add a table
and a router."

### Phased roadmap — the 4–5 week window

- **[DONE] Week 1 — Foundation.** Package restructure (tests stay green); SQLAlchemy models +
  Alembic; ingest pipeline (`POST /api/papers`: dedup → store PDF → extract → persist
  Paper/Report/References/Embedding); projects CRUD. Tests: ingest, dedup, model
  round-trips against tmp SQLite.
- **[DONE] Week 2 — Library UI + discovery backend.** React Query (from deferred backlog);
  routes `/library`, `/projects/:id`, `/papers/:id` reusing existing dashboard components
  against stored reports. `GET /api/papers/{id}/similar` (cosine over stored SPECTER
  vectors, library/project scope) and `GET /api/papers/{id}/related` (OpenAlex
  `related_works`, polite `mailto`, cached). Plus (scope added during build): paper read
  endpoints, ingest title/DOI enrichment, upload flow wired to ingest.
- **Week 3 — Discovery UI + conflict groundwork.** "Discover" tab (In your library /
  From OpenAlex; add-to-project for library hits; metadata + link-out for OpenAlex
  results — importing external papers is stretch, not scope). Extend `extract_statistics`
  with `StatClaim` context capture + tests; persist claims at ingest.
- **Week 4 — Claim/conflict detection. TIME-BOXED: 5 working days hard cap.**
  Scope: within a single project only; anchored on StatClaims (never general free-text
  claim extraction); off-the-shelf NLI cross-encoder (`cross-encoder/nli-deberta-v3-base`),
  no fine-tuning. Pipeline: pair same-type claims across papers → SPECTER-similarity
  screen on context sentences → NLI entailment/contradiction scoring → numeric heuristic
  (opposite-direction effects, p<0.05 vs p>0.05). Output is always **"flagged for
  review" + confidence — never a verdict.**
  - **Day-3 go/no-go — EXPLICIT USER GATE, not self-assessed.** Run the pipeline on a
    hand-built set of 3–5 paper pairs with known agreements/conflicts, then STOP:
    present the flagged examples to the user and wait for their review. The user
    decides continue vs fall back; the working session must not make that call itself.
  - **Fallback:** cut the NLI stage, keep StatClaim extraction (valuable regardless),
    ship semantic discovery alone.
- **Week 5 — Buffer + portfolio polish.** README rewrite with architecture diagram +
  demo GIF/screenshots. Stretch only if clean: BibTeX export port, FTS5 search.

### Post-window direction (documented, deliberately not started)

PDF viewer + annotations → LLM-optional layer (project-aware takeaways, RAG chat,
writing help — all key-gated) → citation network visualization → URL/arXiv ingestion →
library QoL (tags, full-text search, Zotero import).

---
> Source: [sameermogallur/paper-assistant](https://github.com/sameermogallur/paper-assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-13 -->
