## finsight-ipo-x-ray-for-indian-retail-investors

> FinSight = IPO X-Ray for Indian retail investors. Phase 1: RHP → cited fact sheet + bilingual chat with ✅/⚠️/❌ number verification. **Big Phase 2 (now):** upload any IPO offer document → red flags, plain-English risk report, risk level with reasons, comparisons. Open-weight models only. Feature freeze Sun 25 Oct, deadline Sun 1 Nov 2026.

# FinSight — Claude Code instructions (always loaded)

FinSight = IPO X-Ray for Indian retail investors. Phase 1: RHP → cited fact sheet + bilingual chat with ✅/⚠️/❌ number verification. **Big Phase 2 (now):** upload any IPO offer document → red flags, plain-English risk report, risk level with reasons, comparisons. Open-weight models only. Feature freeze Sun 25 Oct, deadline Sun 1 Nov 2026.

## Environment rule
**Local only (B-ADR-16, 4 Oct 2026).** The Google Cloud credit and Claude Code cloud sessions are gone: nothing is deployed or hosted, and every part runs in a local session on the Windows laptop (see Environment below). Start with `git pull`. Do not add Cloud Run, GCS, GPU-job, Docker-deploy or paid-hosting code; Supabase sign-in/Postgres, Vercel and the HF Space are optional config only. Parts once marked ☁️ (fixtures first) and 💻 (real data) are now done together in one local session.

## Session protocol (Phase 2)
1. Read the **"Resume here"** note at the top of `PROGRESS.md`, then `docs/phase2/B07_ROADMAP.md` (current part) and its entry in `docs/phase2/B_EXECUTION_PLAN.md`, then only the B-doc sections that part cites. Index: `docs/phase2/B00_README.md` (Phase 1: `docs/00_README.md`).
2. Run the tests (`uv run poe test`) before changing anything; report in one line.
3. **Model check, both directions, before every part:** Model column in B07 / B_EXECUTION_PLAN (O = Opus, S = Sonnet; ★ and bugs that failed twice on Sonnet = Opus). If it differs from the running model, stop before any work and print exactly: "🔁 MODEL SWITCH: next is <ID> (<title>) — recommended <Opus/Sonnet>. Type /model <opus/sonnet>, then say continue." Otherwise print "✓ Model OK: <ID> on <model>".
4. **One sub-phase or part per session.** End with `✅ <ID> done — run /clear and paste the Phase 2 resume prompt.`; mid-part checkpoint per B07 §0.
5. **Loop:** `git pull` → part (fixtures, then real data, Kaggle, eval) → PR merged → next part.
6. Keep the **"Resume here"** block at the top of `PROGRESS.md` current (exact next step, open files, failing tests) after every merge.

## Hard rules
- **Plan first:** for any task > ~50 lines, a plan (≤ 15 lines) in the PR description (or wait for Akshat's approval when he asks).
- **One primary package per part;** touch other packages only for wiring, and import them only via their `__init__.py`.
- **Never read:** `data/raw/`, `data/processed/`, PDFs, audio, `models/`, weights, `node_modules/`, `.next/`, `.venv/`, notebook outputs. You may read `data/samples/`, `data/gold/` and `tests/fixtures/real/`. For schema questions, run a summary script or ask Akshat for 5 rows. To see a document, use `uv run python -m finsight.pipeline inspect` (prints ≤ 40 lines; may write ≤ 30 truncated snippets to `data/samples/`).
- **Fixture pack exception (B-ADR-15):** `tests/fixtures/real/` may hold real section text, word boxes and tables from public offer documents and short corpus excerpts, gzip JSON ≤ 5 MB per file, ≤ 20 MB total, written only by `scripts/export_fixtures.py`. Never PDFs or weights.
- **Never train models on the laptop.** Write notebooks. On **Kaggle only** (local sessions), you may upload private datasets, push and run notebooks on GPU, poll status and download outputs (`python -m finsight.weaklabel.kaggle`, ADR-042). Official `kaggle` CLI only. Colab runs are started by Akshat.
- **Never commit or print credentials** (Kaggle, Supabase service key, DB URL, HF/Vercel tokens, `.env`). `.env.example` lists names only.
- **Never deploy, create paid resources or spend credits** without Akshat's explicit "go" in chat. No hosting exists (B-ADR-16).
- **Never hand-edit model outputs or eval results.** Fix the pipeline or show ⚠️.
- **Never add investment advice or predictions;** no buy/sell/apply/avoid instructions (forbidden phrases: `configs/forbidden_phrases.yaml`, B-ADR-13). The risk level always carries its disclaimer.
- **Honesty:** disclose AI-assisted labels (`label_source`), single-seed runs and test-informed decisions.
- **Tests are the memory:** every module ships with pytest tests; property tests for `normalize`. Mark tests that need full documents, the corpus or model weights `@pytest.mark.local` (`poe test` and CI skip them).
- **Stuck rule:** if a part balloons past one session, write `BLOCKED.md` (what failed, what was tried) and stop.
- **Verify, don't assume:** check current library and hosting docs before using an API you're unsure of; record non-obvious choices as a proposed ADR (Phase 2: `B-ADR-NN` in `docs/09_DECISIONS.md` + `docs/phase2/B08_DECISIONS.md`).

## Documentation as code (B09)
Docs change in the same PR as the behaviour they describe. Numbers come from `eval_results/` via scripts, API reference from `openapi.json`, config reference from the settings — never typed by hand. Google-style docstrings on public functions; module `__init__` docstrings state the package's job. Every trained model gets a model card, every dataset a datasheet.

## Git (full rules: `docs/08_GIT_WORKFLOW.md`)
Issue → branch `<type>/b<x.y>-<slug>` → small green commits (Conventional Commits, `Refs #n`) → PR (closes issue) → CI green → rebase-merge, delete branch → tag at phase end. `gh pr create`, `gh pr merge --rebase --delete-branch`. Labels `local` / `akshat` mark where an issue runs. Never commit data (except the fixture pack), weights, PDFs, `.env`, outputs, files > 5 MB. No filler commits. Never force-push `main`.

## Environment (Windows laptop, local sessions)
Your shell is Git Bash. Use `pathlib`, LF endings, no Make. Python via `uv`; tasks via poe.
```
uv sync                      # install
uv run poe test              # fast tests (not slow, not local) | uv run poe test-all   # everything
uv run poe lint | fmt | typecheck
uv run poe api               # FastAPI on :8000
uv run python -m finsight.pipeline build --ipo <id>
cd frontend && pnpm dev      # :3000   (NEXT_PUBLIC_USE_MOCKS=1 for fixtures)
```
Laptop: 16 GB RAM (~8 GB used by other apps), RTX 2050 4 GB. Use `FINSIGHT_PROFILE=dev_light` while coding. Stop Ollama before GPU-heavy offline jobs.

**Profiles:** `dev_light`, `full` (laptop) · `deploy_cpu` (ADR-022 paid HF fallback, dormant). The `cloud` / `cloud_gpu` profiles were removed (B-ADR-16); no hosting.

## Stack
Python 3.11 · uv · ruff · mypy · pytest/hypothesis · pydantic v2 · FastAPI + sse-starlette · PyMuPDF · pdfplumber/Docling · transformers · optimum/onnxruntime · bm25s · faiss-cpu · Ollama / llama-cpp-python · faster-whisper · SQLite / Postgres (SQLAlchemy Core + Alembic) · Supabase (Auth, Postgres, Storage)
Frontend: Next.js App Router · TS strict · Tailwind · shadcn/ui · Motion · TanStack Query · Zustand · Recharts · MSW · openapi-typescript · Vitest · Playwright · supabase-js

## Module map (details: `docs/02_ARCHITECTURE.md` §5, Phase 2: `docs/phase2/B02_ARCHITECTURE.md` §4)
Code lives in `src/finsight/<package>/`, tests in `tests/<package>/`, config in `configs/`, helper scripts in `scripts/`, notebooks in `notebooks/`, the app in `frontend/`.
Phase 1: core · ingest · parse · normalize · extract · weaklabel · retrieve · generate · verify · guard · voice · chat · evaluate · api · pipeline
Phase 2 (new): storage · db · jobs · auth · summary · redflags · risks · risklevel · compare · reports

## End of every part
Tick boxes in `docs/phase2/B07_ROADMAP.md`, update `PROGRESS.md` ("Resume here" + ≤ 10 lines), append a Part C section to `docs/10_FINSIGHT_EXPLAINED.md` for any new module, write local follow-ups to `docs/AKSHAT_TODO.md`, include these in the PR.

---
> Source: [AkshatTm/FinSight-IPO-X-Ray-for-Indian-retail-investors](https://github.com/AkshatTm/FinSight-IPO-X-Ray-for-Indian-retail-investors) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-10-04 -->
