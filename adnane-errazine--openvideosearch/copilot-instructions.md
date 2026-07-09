## openvideosearch

> Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

# CLAUDE.md

Behavioral guidelines to reduce common LLM coding mistakes. Merge with project-specific instructions as needed.

**Tradeoff:** These guidelines bias toward caution over speed. For trivial tasks, use judgment.

## 1. Think Before Coding

**Don't assume. Don't hide confusion. Surface tradeoffs.**

Before implementing:
- State your assumptions explicitly. If uncertain, ask.
- If multiple interpretations exist, present them - don't pick silently.
- If a simpler approach exists, say so. Push back when warranted.
- If something is unclear, stop. Name what's confusing. Ask.

## 2. Simplicity First

**Minimum code that solves the problem. Nothing speculative.**

- No features beyond what was asked.
- No abstractions for single-use code.
- No "flexibility" or "configurability" that wasn't requested.
- No error handling for impossible scenarios.
- If you write 200 lines and it could be 50, rewrite it.

Ask yourself: "Would a senior engineer say this is overcomplicated?" If yes, simplify.

## 3. Surgical Changes

**Touch only what you must. Clean up only your own mess.**

When editing existing code:
- Don't "improve" adjacent code, comments, or formatting.
- Don't refactor things that aren't broken.
- Match existing style, even if you'd do it differently.
- If you notice unrelated dead code, mention it - don't delete it.

When your changes create orphans:
- Remove imports/variables/functions that YOUR changes made unused.
- Don't remove pre-existing dead code unless asked.

The test: Every changed line should trace directly to the user's request.

## 4. Goal-Driven Execution

**Define success criteria. Loop until verified.**

Transform tasks into verifiable goals:
- "Add validation" → "Write tests for invalid inputs, then make them pass"
- "Fix the bug" → "Write a test that reproduces it, then make it pass"
- "Refactor X" → "Ensure tests pass before and after"

For multi-step tasks, state a brief plan:
```
1. [Step] → verify: [check]
2. [Step] → verify: [check]
3. [Step] → verify: [check]
```

Strong success criteria let you loop independently. Weak criteria ("make it work") require constant clarification.

---

**These guidelines are working if:** fewer unnecessary changes in diffs, fewer rewrites due to overcomplication, and clarifying questions come before implementation rather than after mistakes.

---

## Project-specific: OpenVideoSearch

### Quality gates (run before every PR)

```bash
# Python
uv run ruff check .          # lint — must be zero errors
uv run ruff format --check . # formatting — must be clean
uv run ty check              # type-check — fix errors you introduce; pre-existing ones are tracked
uv run --isolated pytest -q  # 50+ tests — all must pass

# Frontend
cd apps/web
npm run lint                 # ESLint — must be zero errors
npm run build                # tsc + vite — must produce no errors
```

### Key project concepts

**Media kinds** — every uploaded item is `kind ∈ {video, audio, image}`. The ingestion DAG dispatches a per-kind stage list (see `services/pipeline/dag.py`). Video gets all stages; audio skips frames/VLM; image is one VLM call.

**IngestionStatus flow** — `pending → uploading → processing → transcribing → extracting_frames → segmenting → embedding → video_summary → ready | failed`. Enum lives in `packages/core/openvideosearch_core/models/ingestion_status.py`.

**Agent corpus scope** — at run creation `resolve_corpus()` builds a `scope_snapshot` with the media IDs the user chose. Every retrieval tool validates its `media_id` argument against this snapshot via `_check_scope()`. This is a *product* feature (keeps the model in bounds for the conversation), not a security posture.

**No Alembic** — schema is `SQLModel.metadata.create_all` at startup. New tables: add the model, restart. Pre-existing data: `docker compose down -v` to rebuild clean.

**arq worker is a separate process** — ingestion runs in `arq api.worker.WorkerSettings`, not in the API process. `text_delta` events are Redis-only (never written to DB) — do not "fix" this.

**Imports** — package is installed as `api`, not `apps.api`. Use `from api.services.query import ...`.

**`asyncio_mode = "auto"`** in pytest — no `@pytest.mark.asyncio` needed anywhere.

**`uv add`** is the only way to add Python deps — never edit `pyproject.toml` by hand. Frontend: `npm install --save-dev <pkg>` inside `apps/web/`.

### Test layout

```
conftest.py                          # root — db_engine, db_session, session_factory fixtures
apps/api/tests/
    unit/                            # pure logic, no I/O (milvus_expr, http_errors, segment_ops)
    integration/                     # sqlite in-memory DB, no Docker (media ownership, corpus scope)
packages/core/tests/
    test_config.py                   # Settings validators
```

Integration tests use `sqlite+aiosqlite:///:memory:` — no Docker required.

---
> Source: [adnane-errazine/OpenVideoSearch](https://github.com/adnane-errazine/OpenVideoSearch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-09 -->
