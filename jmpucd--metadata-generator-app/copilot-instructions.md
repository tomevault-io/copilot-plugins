## metadata-generator-app

> Generates archival **metadata** + full OCR text + aligned **searchable PDFs** for digitized

# metadata-generator-app

Generates archival **metadata** + full OCR text + aligned **searchable PDFs** for digitized
collections, using the in-house vision model. Repo: `github.com/jmpucd/metadata-generator-app`
(private). Active branch: **`main`**. A second git remote `server` points at the live
checkout `jmpike@digitization:/digitization/Metadata-Generator-App` — reconcile there too.

> This is the **FastAPI + SvelteKit** app, not the stale public Streamlit version.

## Stack
- `api/` — FastAPI, `uvicorn … --reload` on :8000
- `ui/` — SvelteKit, vite dev on :5173 (access the server UI via SSH tunnel)
- Dagster + SQLAlchemy/SQLite; Typer CLI in `app/cli.py` (`generate` command,
  parallelized across items via `GENERATE_WORKERS`)

## Behavior
Auto-detects **document vs photo** per item:
- **Document** → per-page OCR (one page per request) + an aligned searchable PDF (Qwen OCR
  text quality + Tesseract word boxes for text-layer geometry; per-word morph scaling).
- **Photo** → photo-metadata path.

DB record carries `doc_type`, `full_ocr_text`, `generated_pdf_path`. UI shows a doc-type
badge + "Searchable PDF ↗" link. Resilient HTTP retries + nightly maintenance-window pause
are built in.

## Prompt packs
Reusable prompt modules in `prompts/packs/*.md`, loaded by `app/prompt_packs.py`.
Collections opt in via `Collection.prompt_packs` (comma-separated names; Setup-page
checkboxes). The base prompt holds what's true for every collection (JSON contract,
uncertainty conventions, general foreign-language rule); packs carry material-specific
examples — `language-cjk`, `interiors-foodways`, `correspondence`. A pack's `applies_to`
limits it to the photo / verso / document paths, and `tess_lang` merges into the Tesseract
languages for the searchable-PDF word boxes. Unknown pack names log a warning and are
skipped. Inspect with `python -m app.cli packs [-c COLLECTION]`.

## Endpoint
Default: `https://samwise.library.ucdavis.edu/api` (Open WebUI, OpenAI-compatible),
model `qwen3.6-fast:35b`, key via `VLLM_TOKEN` (or `OLLAMA_TOKEN`). Transport lives in
**digtk.vllm_client** (retries, maintenance pause); `_vllm_infer` just bridges config.
Old cyberdyne-style bare-host `VLLM_BASE_URL` values still work (`/v1` is appended).
Run the CLI with explicit `MODEL_BACKEND=vllm` if `.env` isn't picked up.

## Secrets
Server `.env` holds `OLLAMA_TOKEN=sk-…` — git-ignored, **never commit it**.

## Status / notes
- Chicago Cafe collection (`D-822_Chicago_Cafe_Records`) fully processed (804/804); TIFFs
  converted to JPEG first.
- Next task: build a **Fedora digital collection** from the Chicago Cafe output.
- OCR/PDF/raster + vLLM transport now come from the **digtk** toolkit (editable install
  from `~/code/digtk`; `app/models/words.py`/`pdfbuild.py` were deleted). On the server
  checkout, install digtk too (see digtk/docs/CONSOLIDATION-PLAN.md).

---
> Source: [jmpucd/metadata-generator-app](https://github.com/jmpucd/metadata-generator-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-08-29 -->
