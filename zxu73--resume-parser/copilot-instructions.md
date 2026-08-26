## resume-parser

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

BetterCV — an AI-powered resume optimization platform. Users upload a `.docx` resume and paste a job description, then a multi-agent pipeline evaluates fit, rates bullets, suggests rewrites, and optionally recommends experience swaps from a pool.

## Architecture

- **Frontend**: React 19 + TypeScript + Vite, styled with Tailwind CSS and Radix UI components. Path alias `@/*` maps to `./src/*`.
- **Backend**: FastAPI + LangGraph for multi-agent orchestration. Agent nodes call OpenAI via `langchain_openai.ChatOpenAI` with `.with_structured_output(...)`; the `/chat` and `/job-search` endpoints instead call LiteLLM directly with `acompletion` for lightweight, non-graph completions (freeform chat, JD query extraction). Model is configurable via the `REASONING_MODEL` env var (defaults to `gpt-4o-mini`).
- **Document handling**: python-docx for `.docx` manipulation, PyPDF2/pymupdf for PDFs.

### Agent Pipeline (sequential)

1. **Evaluation** (`evaluate_only_graph`, single node) — exhaustive keyword extraction, STAR analysis, produces `EvaluationResponse` (scores, strengths, weaknesses, missing skills) plus **anchored clarifying questions**.

   Question count is capped by `max_length=8` on the schema field, and the cap is load-bearing: the model sits at whatever ceiling it is given (at the old 3-5 it almost always returned 3). But count and pairing quality are directly opposed, and the prompt cannot hold both. Measured on a 12-bullet resume with no ML experience against an ML-heavy JD: at "aim for 8" it hit 8 nearly every run with only ~40% of pairings plausible — it runs out of honest anchors and starts attaching domain skills to unrelated bullets (`embedding model selection` ← "Wrote unit tests for the payments module"). Removing the floor entirely swung the other way, to 1-5 questions. The current wording (target 6, ceiling 8, floor 5) lands at 3-7 with ~60% plausible; the runs that come back with 3-4 are the ones where nearly every pairing is good.

   A bad pairing is not cosmetic. A "yes" on one is treated as ground truth by `_confirmed_block`, which validates only that the anchor exists in the resume, never that the skill fits it — so it goes straight into a rewrite the candidate has to defend in an interview. The `PLAUSIBILITY GATE` section of `EVALUATION_INSTRUCTION` is what pushes back, including a ban on the hedging phrasings ("did you consider…", "any techniques related to…") that are the tell that a pairing failed. It helps and does not fully hold.

   Two rules are enforced in `evaluate_node` rather than the prompt, because the model ignored both in roughly half of runs and neither needs judgement: `skill_targeted` must be one of the model's own `missing_skills` (a coined phrasing like "shipping LLM-backed features" is not a JD keyword, so a "yes" buys no ATS hit while spending a question slot — and `remaining` filters `missing_skills` by exact string, so the coined skill never leaves the pool and gets placed a second time by Rule A), and at most one question may anchor to a given bullet (a second is waste: `_confirmed_block` groups by `target_bullet`, so both skills land in the one rewrite anyway). Both are at 100% after the change, from ~50%. Every `ClarifyingQuestion` carries a `target_bullet`: the verbatim resume line the question is about ("when you did X, did you use <missing skill>?"). `evaluate_node` runs each anchor through `_snap_to_resume`, which fuzzy-snaps a lightly-paraphrased anchor back to real resume text and blanks it otherwise — a near-miss anchor would produce a `current_text` the document editor can never match.
2. **Rating** (`rate_only_graph`, fanned out) — takes the user's answers and produces:
   - `keyword_suggestions`: rewrites that insert missing JD skills in STAR format. `keywords_added` must be non-empty, each keyword a literal substring of `suggested_text`.
   - `star_suggestions` (target 5-10): STAR-format improvements only. `keywords_added` must be `[]`.
   - Each bullet appears in at most one section. All section invariants are enforced in one place — `RatingResponse.normalize_sections` in `agent.py` — at pydantic validation time; `app.py` does no post-processing. Governed by anti-hallucination rules in `backend/src/agent/guidelines.md`.
3. **Experience Optimizer** (`optimizer_graph`, single node, optional) — scores resume vs pool experiences on JD fit, recommends 1-for-1 swaps only if pool score exceeds resume by 20+ points.

`/apply-swaps-docx` finds the end of an experience block via `starts_new_block`, which checks for a `Heading` style, bold text, a date range (`2021-2024`, `(2020)`, `Jan 2020 - Present`), or the transition out of a bullet list. The last two exist because job sub-headings are often plain text — with only the bold/Heading check the block ran on to the next real heading (`Skills`) and blanked every job in between. The date and list-transition signals are deliberately skipped for list paragraphs, so a bullet that mentions a year or bolds a word is not mistaken for a heading. The endpoint returns `swaps_applied` / `swaps_failed`.

Graphs are invoked from `app.py` with `await graph.ainvoke(state)`. Evaluation and rating are split into separate graphs (and separate endpoints) so the clarifying-questions step can run in between — that HTTP round-trip is the human-in-the-loop step, which is why no checkpointer is needed. Nothing persists across requests: all context must be passed in the state dict (`ResumeState` / `OptimizerState`).

### Job Search + Auto-Apply (backend-only, no frontend yet)

`/job-search` and `/auto-apply` in `app.py` are a separate, ungraphed feature: given a JD, LiteLLM extracts a short search query, JSearch (RapidAPI) returns matching listings, and `_parse_greenhouse_url` flags which ones are Greenhouse-hosted. `/auto-apply` submits the stored resume `.docx` straight to a Greenhouse job board's public Job Board API. Requires `JSEARCH_API_KEY`; `GREENHOUSE_API_KEY` is optional (only some boards require auth). There is currently no frontend consumer for these endpoints.

#### Rating prompt structure

All three graphs are **single-node** — no fan-out, no `Send`, no reducers. The structure lives in the prompts, not in the graph. `evaluate_node` and `optimize_node` are one LLM call each; `rate_node` is 1-3 calls in sequence inside its single node (main rating, optional landing retry, optional coverage sweep).

`rate_node` builds its prompt in sections:

- **CONFIRMED block** — answers grouped **by `target_bullet`**, one entry per bullet listing every skill confirmed for it. The prompt requires exactly one `keyword_suggestions` entry per confirmed bullet, embedding each skill verbatim. Grouping by bullet (not by skill) is what keeps two skills on one line from producing two competing rewrites — only one could ever be applied, since DOCX replacement matches on pre-edit text.
- **Answers denying experience** (`_is_denial`) are collected as `denied` and removed from `remaining` as well as from the CONFIRMED block. Dropping them from CONFIRMED alone is not enough: the skill stays in `missing_skills`, and Rule A then places it as a guess — observed landing right back on the same bullet the user was asked about, so "No, we never used that" produced a rewrite claiming they had. A skill confirmed on one bullet and denied on another counts as confirmed.
- **Unanchored answers** (missing `target_bullet`, or an anchor no longer in the resume) go into an "ADDITIONAL USER CONTEXT" section instead of being lost, keeping older clients working.
- **The must-add list, and how it is enforced.** `_confirmed_block` derives the authoritative list: every answer that is not a denial and whose `target_bullet` is still findable in the resume, grouped by bullet. Its skills come from `missing_skills` (the clarifying questions were generated per missing skill), and `remaining` is everything in `missing_skills` the user did not confirm. Telling the model "these MUST appear" is not enough on its own — roughly three quarters of confirmed skills are missing after the first call, because it reworks long JD phrases (`observability tooling (Prometheus, Grafana, distributed tracing)` → `(Prometheus, Grafana)`). `strip_missing_keywords` then drops the non-literal keyword and `normalize_sections` moves the emptied entry into `star_suggestions`, so the answer disappears with no error anywhere. Two mechanisms close that gap:
  1. **Retry** (`_MAX_LANDING_RETRIES`, currently 1) — re-asks naming the exact strings that must reappear. Accepted only if it recovers ground, counted **in skills, not bullets**, since fixing two of three skills on one line leaves that bullet in the miss list either way. A second retry recovered nothing the first had not over six runs, hence 1.
  2. **Splice** (`_splice_keyword`) — appends `, using <skill>` to whatever is still missing, synthesising an entry if the model dropped the bullet entirely. This is the tier that actually guarantees the skill survives; the retry exists to keep the prose readable so the splice runs less often. Measured: 0 skills lost, 17% spliced.
- `_refile_confirmed` runs after every LLM call to move confirmed bullets back out of `star_suggestions` and rebuild their `keywords_added` from `grouped`. Without it the splice would not find a demoted bullet in `keyword_suggestions` and would rebuild it from the raw resume line, discarding a rewrite that was fine apart from one mangled phrase.
- A placeholder-token scheme (ask for `[[SKILL_1]]`, substitute the real phrase back) was tried and removed: it cut first-call misses only from 15/20 to 11/20 — the model often ignores the token and writes the phrase anyway — and lowered the splice rate from 17% to about 12%, which did not justify ~50 lines. Do not re-add it without measuring against those numbers.
- None of this fires when there are no confirmed answers; that path skips straight to the sweep. See the latency note below — the whole endpoint is 1-3 calls, 16-18s typical and 37s worst observed, inside `_GRAPH_TIMEOUT` (120s), though one run has 504'd, so an upstream stall can still exceed it.

**Section balance and coverage are enforced by a second LLM call, not by the prompt.** Rule A (`keyword_suggestions`) and Rule B (`star_suggestions`) compete for the same bullets and both defer: Rule A omits whenever a skill is not clearly supported, Rule B fires only once Rule A has spent its budget, and neither treats an unemitted bullet as a failure. The result was that most of the resume came back untouched. Measured on a 12-bullet resume against a 30-50 skill JD, the main call alone covered 5-7 bullets and returned an empty `star_suggestions` in a third of runs.

Tightening `RATING_INSTRUCTION` did **not** fix this. An explicit coverage requirement ("every non-strong bullet must appear in one of the two lists", with a three-part definition of strong) plus a two-tier support test for Rule A moved coverage from 5-7/12 to 5-7/12 — no change at all across four runs. Do not try to solve this with more prompt wording; it has been tried.

`_sweep_uncovered` in `agent.py` is what actually works. After the main call (and the landing retry and splice), it diffs `_resume_bullets(resume)` against the bullets already covered and, if any are left, makes one more `with_structured_output(SweepResponse)` call scoped to exactly those, with the still-unplaced skills and the denied list. `SweepResponse` deliberately omits `detailed_ratings` — re-asking for scores only invites the model to contradict the ones the user has already been shown. Measured after: coverage **11-12/12** every run, `star_suggestions` **6-7** with no answers and 2-6 with answers, never empty in ten runs. It only fires when something was left behind, so a fully-covered first pass still costs one call.

Two things the sweep merge has to do itself, because it appends **after** pydantic validation and so `normalize_sections` never re-runs on its output: dedupe against the already-covered bullets (the model gets the full resume for context and will otherwise re-suggest handled lines, surfacing as two competing rewrites of one bullet, only one of which can ever be applied), and file each entry by whether `keywords_added` is non-empty. Matching is on `_norm_bullet`, not exact text — see below.

Rule A's budget cap (two thirds of remaining bullets) still exists and is still only loosely followed, but it now matters much less: whatever Rule A does not claim, the sweep picks up as a STAR rewrite.

**The model drops the leading bullet glyph from `current_text`** despite `guidelines.md` demanding a character-for-character copy. The docx matcher tolerates it (it strips bullet glyphs), but only by spending fuzz budget that should be held in reserve for real near-misses — and a plain string compare against the resume under-reports coverage, which would make the sweep re-suggest handled bullets. `_norm_bullet` is the comparison key everywhere; `_restore_bullet_prefix` puts the real resume line back on every suggestion before `rate_node` returns. Measured after: `current_text` verbatim in the resume for 12/12 suggestions, where it was 0/12.

**Latency.** `/finalize-analysis` now runs 16-18s typical, 37s worst observed over six runs, against `_GRAPH_TIMEOUT` of 120s. It was ~7s with no answers before the sweep. The floor rose because the no-answer path is no longer a single call.

**CONFIRMED bullets are exempt from that budget**, and must stay in `keyword_suggestions` — demoting one throws away the answer the user gave. Note the interaction with the verbatim-keyword rule: the prompt tells the model to keep each keyword frozen (parentheticals, plurals and all) and fix grammar in the carrier clause around it. Advice to "trim the JD's framing" is actively harmful here — it breaks the literal-substring check, `strip_missing_keywords` drops the keyword, and `normalize_sections` then migrates the whole entry to `star_suggestions`, silently losing the user's answer.

Confirmed-skill loss, measured on the same 4-skill fixture: prompt wording alone dropped 3 of 4 skills in **5/5** runs; adding the retry left 1 of 4 dropped in **1-3 of 5** runs; retry + splice drops **0 in 14/14** runs (17% of skills reaching the user via the splice rather than a natural rewrite). **Do not verify a change here with a single run** — an earlier single-run check of this exact code path looked clean and was not, which is how the loss survived a round of "fixed and tested".

### Key Backend Files

- `backend/src/agent/app.py` — FastAPI app, all API routes, graph invocation. `/upload-resume` keys off the filename extension and falls back to `content_type` only when there is no extension: many clients send `.docx` as `application/octet-stream`, and since the resolved type decides whether a `doc_id` is issued at all, trusting the header alone rejected valid uploads and broke the download flow downstream.
- `backend/src/agent/agent.py` — prompts, node functions, state TypedDicts, compiled graphs, pydantic output schemas (`RatingResponse` with `keyword_suggestions`/`star_suggestions`)
- `backend/src/agent/tools.py` — document extraction and AI analysis helpers
- `backend/src/agent/guidelines.md` — bullet rewriting rules, shared by all three rating prompts via `_STAR_AND_KEYWORD_BASE`

### Key Frontend Files

- `frontend/src/App.tsx` — main workflow state machine (upload → analyze → dashboard)
- `frontend/src/components/AnalysisDashboard.tsx` — scores, recommendations, preview; BetterCV Score = 40% keyword match + 35% overall quality + 25% experience relevance
- `frontend/src/components/ResumePreview.tsx` — three-pane change review UI (current bullet / suggestion / live doc preview), approve/skip per suggestion, docx download
- `frontend/src/components/ExperienceManager.tsx` — manage experience pool for swaps
- `frontend/src/components/SwapReview.tsx` — table of resume-vs-pool comparisons, accept/reject swaps
- `frontend/src/components/ClarifyingQuestions.tsx` — shows each question with the resume bullet it is anchored to; submits `{question, answer, skill_targeted, target_bullet}` so the rewriter knows which line to change
- `frontend/src/components/ChatBot.tsx` — floating assistant (bottom-right); sends message + history + resume + JD to `/chat`; returned suggestions join the ResumePreview queue
- `frontend/src/hooks/` — `useResumeUpload.ts`, `useResumeEvaluation.ts`, `useExperienceSwap.ts` encapsulate all API calls
- `frontend/src/types/analysis.ts` — shared TypeScript types (`StructuredRating` has `keyword_suggestions`/`star_suggestions`)

**The main rating call invents resume bullets, and nothing upstream catches it.** Observed on a resume with genuine JD overlap: a suggestion whose `current_text` was `• Participated in a light on-call rotation for the inference tier` — a line lifted verbatim from the JOB DESCRIPTION, not the resume — plus one wholly invented line. Neither validator sees it: `strip_missing_keywords` checks only keywords, `normalize_sections` only files and dedupes, and `_restore_bullet_prefix` repairs a missing glyph but returns anything it cannot match untouched. `_sweep_uncovered` is immune (it filters against a whitelist of real uncovered bullets), which is why this surfaces only from the first call. The damage is not a corrupt document — the docx matcher just fails and reports it in `X-Replacements-Failed` — it is that the UI shows the user a "current bullet" that is not theirs, and approving it leaves them believing their resume claims experience it does not. `_is_resume_line` gates every suggestion at the end of `rate_node`, after the glyph restore (dropping earlier would delete entries whose only defect was the missing `•`), and logs what it drops. It is a backstop: the model still generates them.

**Rewrite-quality measurements, and their variance.** Audited over two fixtures (a weak 12-bullet resume with no ML experience, and one with real JD overlap), ~50 suggestions per 4-6 runs. Reliably clean at 0%: invented numbers, non-literal keywords, the `while <bare noun phrase>` grammar failure, first-person pronouns, gerund openers. Two real defects: **~36% of `keyword_suggestions` are not STAR rewrites at all** — the original bullet survives verbatim as a prefix with a clause appended (`Helped migrate some services to the cloud` → `... using Docker for containerization`), so the keyword lands but the weak verb and missing result are untouched; and **34-67% end in a filler benefit** that `guidelines.md` bans by name. The banned-ending list in `guidelines.md` enumerates six specific phrases, which the model routes around by inventing new ones (`enhancing teamwork`, `increasing app functionality`) — a rule-shaped constraint would hold better than a longer list.

Treat every number here except the deterministic ones as noisy. Between two runs of the same code on the same fixture, tack-on rate read 36% then 10% and banned-tail read 34% then 45%, on samples of 22-30 keyword suggestions. Only `_is_resume_line`-style deterministic guards move a metric to a stable 0%.

## Critical Implementation Invariant

`current_text` in every suggestion must be copied **character-for-character** from the resume (no typo fixes, no rewording). This constraint is enforced in `guidelines.md` and must be preserved in any agent prompt changes.

The matcher in `download-modified-docx` is somewhat more forgiving than that rule implies — it normalizes whitespace, strips bullet glyphs, and falls back to a 40-char prefix anchor, then walks forward deleting line-wrap continuation paragraphs. Anything beyond those tolerances still fails, and the response is a valid `.docx` either way, so misses are invisible in the file itself: the endpoint reports them in the `X-Replacements-Applied` / `X-Replacements-Failed` headers (listed in the CORS `expose_headers`, or the browser cannot read them), and `ResumePreview.tsx` warns the user when any failed.

## Development Commands

### Run both servers concurrently
```
make dev
```

### Frontend only (Vite dev server on :5173, proxies API to :8000)
```
cd frontend && npm run dev
```

### Backend only (Uvicorn on :8000)
```
cd backend && uvicorn src.agent.app:app --reload --port 8000
```

### Backend linting and formatting (from `backend/`)
```
make lint          # ruff check + ruff format --diff + mypy --strict
make format        # auto-fix formatting
make spell_check   # codespell check
make spell_fix     # codespell auto-fix
```

### Backend tests (from `backend/`)
```
make test                         # all unit tests
make test TEST_FILE=tests/unit_tests/test_foo.py  # single file
make test_watch                   # watch mode
make extended_tests               # extended test suite
```

Package manager: `uv` (backend), `npm` (frontend).

## Environment Variables

- `OPENAI_API_KEY` — required for AI model calls
- `REASONING_MODEL` — OpenAI model identifier used by both ChatOpenAI and LiteLLM (default: `gpt-4o-mini`)
- `JSEARCH_API_KEY` — RapidAPI key for `/job-search`; endpoint 500s without it
- `GREENHOUSE_API_KEY` — optional, used as basic-auth for `/auto-apply` against boards that require it

## Deployment

### Azure App Service (live)

`deploy/azure-deploy-code.sh` deploys the **source**, not the container: `az acr build` needs ACR Tasks, which Azure blocks on free and student subscriptions (`TasksOperationsNotAllowed`), and building locally would need Docker plus a cross-architecture build. `deploy/azure-deploy.sh` keeps the container path for a subscription that allows it. Dropping the registry also removes its cost.

Four things about the platform that each cost an hour to rediscover:

- **Oryx compresses the build output.** `wwwroot` ends up holding `output.tar.zst`, not the app; it is extracted to a fresh `/tmp/<hash>` on every start. An absolute `PYTHONPATH=/home/site/wwwroot/backend/src` therefore points at nothing. `$(pwd)` at start-up is the only reliable handle on the app root.
- **The startup command must `export` inside `sh -c`.** An inline `PYTHONPATH=... python -m uvicorn ...` prefix does not survive the wrapper script Oryx generates, and the app dies with `ModuleNotFoundError: No module named 'agent'`.
- **Deploy before setting the startup command.** Pointing a live app at code that is not there yet puts it in a crash loop, and a crash-looping site takes Kudu down with it — the deployment then fails with an opaque `502`. `az webapp config set --startup-file ""` also does not clear a bad command; only a REST `PATCH` of `properties.appCommandLine` does.
- **A brand-new subscription has no resource providers registered**, and restricted subscriptions carry an "Allowed resource deployment regions" policy that usually excludes `eastus`. The script registers the providers and defaults to `westus3`.

Secrets come from `backend/.env`, parsed the way python-dotenv does — a naive `cut -d= -f2-` also swallows a trailing `# comment`, which then travels to Azure as part of the key and fails much later as an `'ascii' codec can't encode` error on the `Authorization` header. `_llm_for` is `@lru_cache`d, so a bad key is baked into the cached client and every request fails in milliseconds without ever reaching OpenAI. The script validates that the key is ASCII before deploying.

Azure App Service via `deploy/azure-deploy-code.sh` — **this is what is actually deployed**. A `Dockerfile` (multi-stage, Node 20 → Python 3.11) is kept for container hosts; `deploy/azure-deploy.sh` is its Azure counterpart, unused because ACR Tasks is blocked on this subscription. The Vercel and Render descriptors have been removed. Production entry: `uvicorn src.agent.app:app` with `PYTHONPATH=backend/src`. Frontend dist is served as static files from the FastAPI app. The container's `CMD` is in shell form so it can expand `${PORT}`, which App Service injects.

**`.dockerignore` is load-bearing, not housekeeping.** The Dockerfile does `ADD backend/ /deps/backend`, which without it copies `backend/.env` — a live `OPENAI_API_KEY` — into a published image layer, along with a `.venv` full of host paths. It deliberately does *not* exclude markdown: `guidelines.md` is read at runtime and `backend/README.md` is referenced by `pyproject.toml`, so a blanket `*.md` rule breaks the build. `deploy/azure-deploy.sh` refuses to run if the file is missing.

**Uploaded-document storage decides whether the app works at all in the cloud.** `/upload-resume` writes a `.docx` and returns a `doc_id`; `/download-modified-docx` reads it back on a *later request*. Container-local disk cannot serve that: it is wiped on restart, and with more than one instance the second request can hit a replica that never saw the file, surfacing as "Document not found". `RESUME_STORE_DIR` must therefore point at storage shared across instances and outliving the container — on Azure App Service a path under `/home`, which additionally requires the app setting `WEBSITES_ENABLE_APP_SERVICE_STORAGE=true` (off by default for custom containers, and silently so). Files older than `RESUME_STORE_TTL_HOURS` (default 24) are purged at startup; before that they accumulated forever.

---
> Source: [zxu73/resume-parser](https://github.com/zxu73/resume-parser) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-26 -->
