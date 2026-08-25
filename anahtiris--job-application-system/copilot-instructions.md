## job-application-system

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this system does

An AI-assisted job application pipeline with a two-stage funnel:

1. **Lead capture → triage**: Browser extension sends raw job board text to the backend instantly. Batch extraction (`POST /api/leads/extract-captured`) parses company/title/JD via LLM. Each lead is then analyzed (fit score + company tone) and either approved (→ Application) or rejected.
2. **Application pipeline**: Given a job description, produces a tailored one-page CV and cover letter (DOCX + PDF) stored under `applications/[Company]/`.

Initial setup: Parse a raw resume into `resume_master.md` / `resume_master_de.md`.

The full step-by-step orchestration is in `workflows/end-to-end.md`.

## Workflow permission protocol

Before starting any multi-step workflow (process captured jobs, generate CV/cover letter, interview prep), list every command you plan to run — reads AND writes — grouped by type, and ask once before doing anything. Example format:

**Read (auto-approved):** GET /api/leads/, resume_master.md, data/skills.json  
**Write (will prompt):** PUT /api/leads/{id}/processed ×3, WebSearch ×3

After the user says proceed, run everything. Write-operation prompts will still appear from the permission system — the user can click "Allow for this session" on the first occurrence of each pattern to avoid repeated prompts.

## Session-bounding convention

Long repetitive workflows degrade as the context window fills — the agent drifts from the rules it read at the top of the session. Bound them. For **any** workflow that processes many similar items (captured-job triage, skills extraction, multi-application generation), do this:

1. Process at most **N items per pass** (default N=4; the count is per-workflow and may be tuned).
2. After the batch, **STOP** — do not continue past N in the same pass. Report which items were processed and how many remain.
3. Tell the user to `/clear` and re-paste the prompt to continue the next batch, so each batch runs in a fresh context that re-reads these rules.

The point is to bound context/token usage per run and keep the last item's output as faithful as the first. For genuinely small counts a single all-at-once pass is fine (e.g. plain "process my captured jobs" with only a few pending) — the bound matters when the item count is large. Script-driven workflows (`scripts/batch_generate.py`) express the same bound as a `--limit` flag plus a printed resume cursor rather than a `/clear` instruction.

This convention generalizes the captured-jobs batch mode (see "Batch mode" under the leads pipeline below) to every repetitive workflow.

## Key commands

```bash
# Start the web app (both must run simultaneously)
cd app/backend && source ../../.venv/bin/activate && uvicorn main:app --reload   # :8000
cd app/frontend && npm run dev                                                   # :3000

# Install backend dependencies (first time)
source .venv/bin/activate && pip install -r app/backend/requirements.txt

# Unpack/pack DOCX for XML editing
python app/backend/office/unpack.py <template.docx> <output_dir>/
python app/backend/office/pack.py <unpacked_dir>/ <output.docx> --original <template.docx>

# Convert DOCX to PDF (requires LibreOffice)
libreoffice --headless --convert-to pdf <file.docx> --outdir <outdir>/

# Check page count of a PDF
pdfinfo <file.pdf> | grep Pages
```

## Web App (`app/`)

A local Next.js + FastAPI app that replaces the manual Claude Code workflow with a structured UI. It enforces all guardrails in code so the LLM cannot exaggerate.

- **Backend**: `app/backend/` — FastAPI + SQLite. Routers: `/api/resume`, `/api/application`, `/api/tracker`, `/api/settings`, `/api/leads`, `/api/trash`. Services: `generator.py` (locked tailoring + streaming), `reviewer.py` (persona + 2 random reviewers), `researcher.py` (company scraper + tone classifier), `analyzer.py` (JD gap analysis), `interview.py` (prep + skills debrief), `pdf.py` (LibreOffice + 1-page check).
- **LLM layer** (`services/llm.py` + `services/providers/`): `generate(model, prompt, system, fmt)` dispatches on a `provider/model` slug. `fmt` is an optional JSON schema for structured output, translated per provider: Ollama → `format` (full schema); Anthropic → forced tool call (`input_schema=fmt`, guaranteed schema-valid); OpenAI → `response_format` json_schema; Gemini → `responseMimeType: application/json` (valid-JSON floor only — Gemini's responseSchema dialect rejects the `$defs`/`additionalProperties` the Pydantic schemas use). Perplexity accepts `fmt` but ignores it (json_schema is gated/beta). Because not every provider enforces the schema, callers using `fmt` must still keep their JSON-sanitizing / graceful-fallback path. Output schemas live next to their service (`analyzer_schema.py`, `researcher_schema.py`, `reviewer_schema.py`, `skill_extractor_schema.py`, `interview_schema.py`). Streaming (`stream()`) does not support `fmt`.
- **Frontend**: `app/frontend/` — Next.js 16. Pages: dashboard (`/`), setup (`/setup`), skills inventory (`/skills`), trash (`/trash`), leads list (`/leads`), lead detail (`/leads/[id]`), 5-step wizard (`/apply/new`), detail (`/apply/[id]`), settings (`/settings`).
- **Ollama models** (configured in `app/backend/config.toml`): all four roles (parser, writer, reviewer, research) are set there. Restart the backend after editing `config.toml` — it is read once at startup.
- **Persona**: `data/persona.md` (gitignored) — the obligated personal reviewer. Edit via `/settings`.
- **PDF gate**: the "Finalize & Generate PDFs" button is gated until review completes. The backend enforces a 1-page check and returns HTTP 422 if the CV overflows.

## Architecture

### Source of truth
`resume_master.md` (EN) and `resume_master_de.md` (DE) are the canonical resume data. **Never modify them.** All tailoring happens inside the DOCX XML.

### DOCX editing approach
DOCX files are ZIP archives. The workflow unpacks them (`unzip`-equivalent via `app/backend/office/unpack.py`), edits `word/document.xml` with string replacement, then repacks. The accent colour `1a56a4` in the CV template can be replaced wholesale with a company brand colour.

### Leads pipeline (`/api/leads/`)

Two-table funnel: `JobLead` → `Application`. Statuses: `captured → new → analyzing → analyzed → approved → applied → rejected`.

- **`POST /from-text`** — browser extension posts `{text: body.innerText, url}`. Saved instantly with `status="captured"`, no LLM blocking. Deduplicates by `source_url`.
- **`POST /extract-captured`** — batch LLM extraction for all `captured` leads. Parses company/job_title/language/job_description from raw text; sets `status="new"`.
- **`POST /{id}/analyze`** — runs `analyzer.analyze_jd` + `researcher.research_company` in parallel. Stores fit score (0–100), verdict (`strong`/`maybe`/`skip`), company tone. Score normalization guard: float 0–1 is multiplied by 100.
- **`POST /{id}/approve`** — creates an `Application` record (copies company, job_title, language, JD, tone, source_url from lead). Sets `lead.status="approved"`, `lead.application_id=app.id`. New application starts with `status="New"`.
- **`POST /{id}/reject`** — sets `status="rejected"`.
- **`PUT /{id}/processed`** — write-back endpoint for the Claude flow below. Accepts `{company, job_title, language, job_description, company_tone, company_research, fit_analysis}`, stores `fit_analysis` as `fit_analysis_json`, derives `fit_score`/`fit_verdict` from it (reusing `_verdict`), and sets `status="analyzed"`. Mirrors what the Ollama `/analyze` path writes.

### Processing captured jobs — "process my captured jobs"

When the user says **"process my captured jobs"**, Claude Code does extraction **and** analysis itself in one pass (do NOT call the Ollama `/extract-captured` or `/{id}/analyze` endpoints — those are the offline fallback):

1. `GET /api/leads/` and select leads whose `status` is `captured` or `new`.
2. Read `resume_master.md` (or `resume_master_de.md` for German), `data/skills.json`, and `data/career_goal.md` from disk.
3. For each lead: extract `company`/`job_title`/`language`/`job_description` from `raw_text`; analyze fit against the resume + skills inventory + career goal; research the company on the web for tone + sentiment. Say "no reliable data found" when a company is thin — **never fabricate** reviews, salary, or facts.
   - **`job_description` must be the verbatim role content, not a summary.** Copy the actual posting sections (intro/Einleitung, responsibilities/Aufgaben, requirements/Profil, benefits/Wir bieten, and any salary figure) word-for-word from `raw_text`. Only strip job-board chrome — site nav, "related jobs" lists, SEO link spam, footers, and the board's own auto-match widgets (e.g. Stepstone's "Passt hervorragend / Du erfüllst alle Anforderungen", which is scored against the user's uploaded CV, not employer text). Keep the original wording and language of each section (postings are often bilingual). Do not paraphrase or compress — downstream steps (Job Analysis, generation, **interview prep**) depend on full-fidelity JD text, and `raw_text` is not copied onto the `Application`, so a lossy `job_description` is unrecoverable after approve. Flag board-estimated salaries as such (e.g. "von Stepstone geschätzt, nicht vom Arbeitgeber angegeben"), never as the employer's stated number.
   - **Check for a "no cover letter" tag.** If `raw_text` contains a phrase like "Anschreiben nicht erforderlich", "kein Anschreiben erforderlich/nötig", "cover letter not required", or "no cover letter needed", set `cover_letter_required: false` in the payload below. This is a default, not a hard rule — the wizard has a manual checkbox override, so a false positive/negative is recoverable.
4. `PUT /api/leads/{id}/processed` with this body (the `fit_analysis` shape must match exactly — it is what `/leads/[id]` renders):
   ```json
   {
     "company": "...", "job_title": "...", "language": "en|de", "job_description": "...",
     "company_tone": "direct|startup|contractor|agency",
     "company_research": "one-line tone reasoning",
     "cover_letter_required": true,
     "fit_analysis": {
       "core_theme": "...",
       "must_haves":    [{"skill": "...", "status": "STRONG|HONEST|GAP|UNKNOWN", "tier": 1, "evidence": "..."}],
       "nice_to_haves": [{"skill": "...", "status": "STRONG|HONEST|GAP|UNKNOWN", "tier": null, "evidence": "..."}],
       "ats_keywords": ["..."],
       "match_score": 0,
       "strongest_angle": "...",
       "weakest_point": "...",
       "is_poor_match": false,
       "goal_alignment": "aligns|neutral|detours",
       "goal_alignment_note": "..."
     }
   }
   ```

**Batch mode — "process my captured jobs in batches of N" (default N=4).** The extension's batch button copies this phrase when more than 4 leads are pending. Process only the **next N** captured/`new` leads (oldest first), exactly as above, then **STOP** — do not continue past N in the same pass. The point is to bound context/token usage per run. After the batch, report which leads were processed and how many remain, then tell the user to `/clear` and paste the prompt again to continue the next batch. (Plain "process my captured jobs" still processes all pending leads in one pass — fine for small counts.)

The `/leads` page shows color-coded status badges and fit verdict chips. The `/leads/[id]` detail page shows must-haves/nice-to-haves/ATS keywords from `fit_analysis_json`; all three arrays default to `[]` if missing (LLM sometimes uses non-standard field names).

### Processing an uploaded profile — "process my profile"

When the user says **"process my profile"** (or clicks the parse-stage "Copy
prompt for Claude" button on the Setup page), Claude Code structures the uploaded
résumé itself — do NOT call the Ollama `/api/resume/parse` LLM path (that is the
offline fallback, which runs the configured parser model automatically on upload):

1. The upload already saved the raw extracted text to `data/resume_raw_en.txt`
   (English) or `data/resume_raw_de.txt` (German) — no LLM/key needed for that
   step. Read the file for the active language.
2. Structure it into clean markdown using the same sections as the parser
   (`# Profile`, `# Contact`, `# Skills`, `# Experience`, `# Projects`,
   `# Education`). **Never fabricate** — extract only what is in the raw text,
   dates exact. Save via `PUT /api/resume/master` with `{language, content}`.
3. Then build the skills inventory with the existing "process my skills" steps
   and `PUT /api/resume/skills/merge`.

The offline fallback is the upload's automatic LLM structuring path: `POST
/api/resume/parse` extracts the text (always, keyless) and tries the configured
parser model. If no key/model is available it saves the raw text to disk and returns a clear
`parse_error` instead of failing, steering the user to this Claude path.

### Building the skills inventory — "process my skills"

When the user says **"process my skills"** (or clicks "Copy prompt for Claude" on the
Setup or Skills page), Claude Code builds the tiered skills inventory itself — do NOT
call the Ollama `/api/resume/skills/extract` endpoint (that is the offline fallback):

1. Read the master résumé (`resume_master.md` / `resume_master_de.md`),
   `data/skills.json` (existing inventory), and `data/career_goal.md` from disk.
2. Follow `skills/skill-assessment/SKILL.md`: draft a tier (1–4) + evidence per skill
   from concrete résumé evidence; **never fabricate**.
3. **Interview the user** (batched questions) about any skill whose tier the résumé
   cannot settle. Surface skills mentioned in answers that weren't on the résumé.
4. Save via `PUT`/`POST http://localhost:8000/api/resume/skills/merge` with
   `{"skills": {"Name": {"tier": 1, "evidence": "...", "needs_review": false}}}`. The
   backend merges keep-my-edits (existing entries win on collision) and persists.

The Ollama fallback (`POST /api/resume/skills/extract`) does the same extraction in
one offline pass with no interview, flagging low-confidence guesses `needs_review`.

### Browser extension (`browser-extension/`)

Manifest V3. Click → `chrome.scripting.executeScript` grabs `{text: body.innerText, url: location.href}` from the active tab → `POST /api/leads/from-text` → opens `localhost:3000/leads/{id}`. Returns in ~1 second (no LLM on the capture path). Deduplication: if a non-rejected lead already exists for the URL, shows "Already captured" and opens the existing lead.

### Application status flow

`New → Draft → Finalized → Applied → Interview → Offer / Rejected`

- **New**: application created from an approved lead, no documents yet.
- **Draft**: markdown drafts generated, not yet exported. Set via the wizard Generate step's `PUT /api/application/drafts` (auto-promotes `New → Draft`) or `PUT /api/application/finals`.
- **Finalized**: PDFs/DOCX exported via `POST /api/application/pdf` (the Finalize step), auto-promotes `New|Draft → Finalized`. If drafts are regenerated afterwards, `PUT /api/application/drafts` demotes `Finalized → Draft` since the exported files are now stale.
- Further transitions (`Draft|Finalized → Applied`, etc.) are manual via `PATCH /api/tracker/{id}/status`. Two side-effects on that endpoint keep the source `JobLead` in step: `→ Applied` sets a still-`approved` linked lead to `status="applied"`; `→ Rejected` soft-deletes the linked lead.

### Trash / soft delete (`/api/trash/`)

`DELETE /api/tracker/{id}` and `DELETE /api/leads/{id}` are **soft deletes** — they set `deleted_at` rather than removing the row. `GET /api/tracker/` and `GET /api/leads/` exclude rows where `deleted_at` is set, so deleted items vanish from the normal lists immediately but remain recoverable. The `/trash` page (sidebar icon, under Skills) lists everything soft-deleted, newest first, with:
- **Restore** — `POST /api/trash/applications/{id}/restore` or `POST /api/trash/leads/{id}/restore`, clears `deleted_at`.
- **Delete forever** — `DELETE /api/trash/applications/{id}` or `DELETE /api/trash/leads/{id}`, a real hard delete (same as the old pre-soft-delete behavior; files under `applications/[Company]/` are not touched either way).

Soft-deleting an `Application` does not affect a `JobLead` that references it via `application_id` (and vice versa) — each is restored/purged independently.

### Person name / file naming

`person.name` is stored in the settings DB table and used for PDF/DOCX filenames. It is auto-extracted from the `# Contact` section when a resume is first parsed via `POST /api/resume/parse`. It can also be set manually via `PUT /api/settings/profile`. Files are named `[FirstName_LastName]_CV.docx` (EN) / `[FirstName_LastName]_Lebenslauf.docx` (DE), etc.

### Wizard flow (5 steps)
1. **Job Details** — company, company website, job posting URL (`source_url`, carried over from the lead on approve), job title, language, job description, cover letter notes. Creating saves to DB; revisiting patches via `PATCH /api/tracker/{id}/details`.
2. **Job Analysis** — auto-runs `POST /api/application/analyze-jd` on entry; shows STRONG/HONEST/GAP classification per JD skill against the skills inventory, ATS keywords, match score, and overclaim flags. Uses the `research` model. The analyzer requests structured output (`JDAnalysis.model_json_schema()` via `generate(fmt=...)`); the result is then validated with `JDAnalysis.model_validate` (raises on schema drift). The markdown-fence / trailing-comma sanitizer is kept as a fallback for providers that don't enforce the schema. Clears result on Back so re-entry re-runs fresh.
3. **Generate** — auto-runs company research (tone + address) on entry; streams resume tailoring + cover letter via SSE. Tone can be overridden before generating.
4. **Review** — persona + 2 random reviewers score and rewrite both documents. Side-by-side panel: left = highlighted document, right = rewrite queue. Items sorted by document position.
5. **Finalize** — editable company address, side-by-side markdown editors, word count indicator on cover letter, "Finalize & Generate PDFs" exports DOCX + PDF.

When resuming an existing application, `inferStep` skips Job Analysis (returns step 3/4/5 based on progress).

## Writing Style
- Never use the em-dash (—) symbol in writing documents. Use commas or periods instead.
- No corporate buzzwords ("leverage", "passionate", "drive innovation") unless quoting source material.
- For application materials specifically: no contractions, consistent tense, no fabricated enthusiasm.

## Honest Representation
- Never claim production/deployed status for prototype or personal projects unless explicitly confirmed.
- Skill tier claims must match the 4-tier framework (daily-driver / solid applied / AI-assisted / exposure only) — never round up.
- Flag if a generated claim sounds stronger than the underlying evidence.

### Company tone classification
`researcher.py` classifies companies into 4 tones used to adapt the cover letter opening strategy:
- `direct` — established product/service company
- `startup` — small/early-stage; triggers calm, hands-on builder voice with no corporate language
- `contractor` — IT consulting/contracting firm
- `agency` — recruiting or staffing agency

### Cover letter guardrails (enforced in `COVER_LETTER_SYSTEM`)
- Start date is computed in Python (`01.MM.YYYY` format, never "ab sofort") and injected as `START_DATE` — the LLM must not compute or guess it.
- Contact email is injected as `CONTACT_EMAIL` and enforced post-generation via regex replacement to prevent LLM hallucination.
- Banned phrases (case-insensitive): "leverage", "drive successful", "demonstrate(s) success", "track record of", "proven ability", "deep understanding", "extensive experience in", "well-versed in", "synergy", "best-in-class", and others — see `COVER_LETTER_SYSTEM` in `generator.py`.
- No overclaiming: skills that are recent (< 12 months) or from a side project must be described as "recent" / "exploring" / "in a side project".

### Skills (manual Claude Code workflow)
Each `skills/*/SKILL.md` is a self-contained instruction set consumed during a manual application run:
- `job-analysis` — extract keywords/priorities from a JD
- `company-research` — classify employer type and adapt opening strategy accordingly
- `resume-tailoring` — XML-level CV tailoring with mandatory 1-page check before finalising
- `cover-letter-aida` — AIDA-structured letter with specific closing formula (availability date + clickable mailto hyperlink in DOCX XML)
- `cv-review` — multi-persona review of both documents; 2 randomly selected personas each score 10 criteria 1–10 and provide concrete rewrites; produces a Revised Draft for each document

### Skills inventory
`data/skills.json` (gitignored) stores skill tiers (1=Core, 2=Proficient, 3=Familiar, 4=Exposure) + evidence. Managed via `/skills` page. Injected into cover letter generation as `SKILLS_INVENTORY` block to enforce honest tier-appropriate language, and used by Job Analysis to classify JD requirements.

### Interview preparation (detail page, Interview tab)
An application in the Interview stage can track multiple **rounds** (Screening, Technical, Final, or a custom label), each with its own prep, notes, and date — added explicitly via an "+ Add Round" action in the workspace (never implicitly on first Generate). Stored as `Application.interview_rounds_json`, a JSON array of `{id, round_type, date, prep, notes, created_at}` objects. A round's `round_type` cannot be changed after creation — delete and re-add instead. `Application.interview_date` still exists but is now a **derived, read-only** field: the backend recomputes it on every round add/edit/delete as the soonest upcoming round date, else the most recent past date, else `null` — clients never set it directly. This keeps the dashboard's upcoming/past-interview sorting (`app/frontend/app/page.tsx`) working unmodified. Legacy single-round data (the old `interview_prep_json`/`interview_date`/`interview_notes_json` fields) lazily migrates into one round named `"Technical"` on first read (`services/interview_rounds.py: load_rounds`).

Round CRUD lives in `routers/tracker.py`: `POST /api/tracker/{app_id}/interview-rounds` (body `{round_type, date}`), `PATCH /api/tracker/{app_id}/interview-rounds/{round_id}` (body `{round_type?, date?}`), `PATCH /api/tracker/{app_id}/interview-rounds/{round_id}/notes` (body `{notes}`), `DELETE /api/tracker/{app_id}/interview-rounds/{round_id}`. Each returns `{"rounds": [...], "interview_date": ...}`.

Two independent generation blocks, both only active when `app.status === "Interview"`:
- **Interview Prep** — 7 fields (`company_analysis`, `introduction_script`, `common_questions[]`, `job_specific_questions[]`, `weak_spots[]`, `questions_to_ask[]`, `salary`), parameterised by round/interviewer/focus skills and scoped to one round's `prep`. The Introduction Script is fed the tailored CV + cover letter, not just the master resume. Q&A items are `{id, q, a}` (job-specific `a` is talking-point bullets; weak-spot `q` is the likely probe, `a` the honest answer), ask items are `{id, text}`, the rest are plain strings. Rendered by the tabbed company-prep panel (`app/frontend/app/interview/company-prep/`) as add/edit/delete-able rows, with a round switcher above the tabs. Two generation paths (below).
- **Skills Debrief** — tier-aware per-skill coaching: STAR prompts for Tier 1/2, honest answer templates for Tier 3, overclaim flags. No parameters, application-wide (not round-scoped). Stored in `interview_debrief_md`.

**Two ways to generate Interview Prep** (mirrors the captured-jobs pattern):
- **Generate with Ollama** — `POST /api/application/interview-prep` (body now requires `round_id` alongside `application_id`, `interview_round`, `interviewer_type`, `focus_skills`) runs `interview.generate_interview_prep` offline, using structured output (`GenInterviewPrep.model_json_schema()` via `generate(fmt=...)`, see the LLM layer note above) to return the JSON object directly, written into that round's `prep`. Company Analysis is inferred from the JD (no web), labelled "(inferred — verify)".
- **Copy prompt for Claude** — the button copies an instruction referencing the application id, round id, interviewer type, and focus skills. When the user pastes it, Claude Code follows `skills/interview-prep/SKILL.md`: gathers context from `/api/tracker/{id}` and disk, **web-researches the company** (never fabricate), builds all seven fields in the page's language, and saves via `PUT /api/application/{id}/interview-prep` with body `{round_id, prep}` (the JSON object, not markdown). This is the **Interview prep — Claude path**.

**Export to PDF** — the `/interview` workspace's `CompanyPrepPanel` topbar has an "Export PDF" button that hits `GET /api/application/{id}/interview-export.pdf?round_id=`, defaulting to the most recently created round if `round_id` is omitted. The endpoint assembles a standalone HTML packet in `services/interview_export.py` (`build_interview_html(app, round)`: the seven Interview Prep fields + that round's working notes + the verbatim `job_description`, with headings per `app.language`), converts it via the LibreOffice pipeline (`render_interview_pdf`, reusing `pdf._find_soffice`), and streams it as a download. The Skills Debrief is excluded and nothing is persisted under `applications/`.

### Rejection analysis (`/applications/analysis`)
Surfaces patterns across all closed applications (`status IN ("Rejected", "Rejected after interview", "Ghosted after interview")`, not soft-deleted) — skill gaps, company-type breakdown, match-score distribution, goal alignment, and outcome stage — plus an LLM-written narrative. Requires at least 3 closed applications; otherwise `GET /api/tracker/analysis/rejected` returns `{"insufficient_data": true}`. Aggregation logic lives in `services/rejection_analysis.py` (`_aggregate`); the narrative (whichever path wrote it last) is persisted as JSON under the `rejection_analysis_json` key in the `Setting` table, so `GET /api/tracker/analysis/rejected` always returns fresh aggregates merged with the last-saved narrative.

**Two ways to generate the narrative** (mirrors the Interview Prep pattern):
- **Generate with Ollama** — `POST /api/tracker/analysis/rejected/generate` runs `rejection_analysis.generate_narrative` offline against the `research` model, using the pre-computed aggregates + `data/career_goal.md`, and persists the result.
- **Copy prompt for Claude** — the button copies an instruction to fetch stats from `GET /api/tracker/analysis/rejected`, read `data/career_goal.md`, write a 2-3 paragraph narrative (specific, honest, actionable, no corporate language — same bar as `SYNTHESIS_SYSTEM`), and save via `PUT /api/tracker/analysis/rejected` with `{"narrative": "..."}`. This is the **Rejection analysis — Claude path**.

### Templates
`templates/resume/resume_en.docx` and `templates/resume/resume_de.docx` are the base CV DOCX files. `templates/cover-letter/cover_letter.docx` is the cover letter base. All are gitignored — `.gitkeep` preserves the folders.

### Output structure
Every application lives in `applications/[Company]/` and is gitignored. Detail page (`/apply/[id]`) provides PDF and DOCX download links for both documents.

### Database migrations
`db.py` runs safe `ALTER TABLE` migrations on startup for new nullable columns (try/except loop). Current extra columns on `Application`: `resume_docx_path`, `cover_letter_docx_path`, `cover_letter_notes`, `interview_prep_json`, `interview_debrief_md`, `interview_date`, `interview_notes_json`, `interview_rounds_json`, `deleted_at`. `JobLead` table: `raw_text`, `deleted_at` added via migration. For constraint changes (e.g. making a column nullable), use the SQLite table-recreation pattern: CREATE new → INSERT SELECT → DROP old → RENAME.

## Critical constraints

- **CV must be exactly 1 page.** Always run the `pdfinfo` page count check before copying to the applications folder. Profile summary must be under 200 characters.
- **Never fabricate resume content.** Only surface, reorder, or rephrase what exists in the master file.
- **Cover letter closing is strictly formatted**: must include an `01.MM.YYYY` availability date (never "ab sofort") and a clickable `mailto:` hyperlink embedded in the DOCX XML (see `skills/cover-letter-aida/SKILL.md` for the exact XML snippet).
- **Language matching**: German JD → `resume_master_de.md` + German template + German filenames (`Lebenslauf`, `Anschreiben`). English JD → `resume_master.md` + English template + English filenames (`Resume`, `Cover_Letter`).

## File naming convention

| Language | Type | Filename |
|---|---|---|
| DE | Resume | `[FirstName_LastName]_Lebenslauf.docx/.pdf` |
| DE | Cover letter | `[FirstName_LastName]_Anschreiben.docx/.pdf` |
| EN | Resume | `[FirstName_LastName]_CV.docx/.pdf` |
| EN | Cover letter | `[FirstName_LastName]_Cover_Letter.docx/.pdf` |

---
> Source: [anahtiris/job-application-system](https://github.com/anahtiris/job-application-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-22 -->
