## deepl-to-llm

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project is

A small FastAPI service that exposes a **DeepL-compatible `/translate` endpoint** backed by an LLM (OpenAI-style chat completions API). It exists so tools that speak the DeepL HTTP API (e.g. Collabora Online) can use an LLM for translation without code changes on the client side. The app is split across `main.py` (FastAPI endpoint + LLM call) and `bridge.py` (HTML structure-preservation logic).

## Run

Local dev (requires Python 3.11 per the Dockerfile, though any 3.10+ works):
```bash
pip install -r requirements.txt
uvicorn main:app --host 0.0.0.0 --port 1188 --reload
```

Docker (matches production):
```bash
docker build -t deepl-to-llm .
docker run -p 1188:1188 -e LLM_API_KEY=... -e BRIDGE_TOKEN=... deepl-to-llm
```

## Test

Unit + endpoint tests (no network — LLM is stubbed):
```bash
pip install pytest
python3 -m pytest test_bridge.py test_endpoint.py
```

Live smoke test against the real LLM (reads `LLM_API_URL`/`LLM_API_KEY`/`LLM_MODEL` from the environment — no keys are baked into the file). Exercises Collabora-shaped payloads derived from `test.docx` headers/footers:
```bash
LLM_API_URL=... LLM_API_KEY=... LLM_MODEL=... python3 smoke_live.py
```
There is no linter or formatter configured. The OpenAPI docs at `/docs` (Swagger UI) are the easiest way to exercise the endpoint by hand.

## Configuration

All config is via env vars read at import time in `main.py` (module-level constants — changing them requires a restart). **All must be set in docker-compose; the in-code defaults are placeholders only and will not work against a real LLM.**

- `LLM_API_URL` — upstream chat-completions URL (default is a placeholder `https://example.com/...`).
- `LLM_API_KEY` — bearer token sent to the LLM. Default empty; if empty, no `Authorization` header is sent.
- `LLM_MODEL` — model name passed in the payload. Default empty.
- `BRIDGE_TOKEN` — token clients must present to use this bridge. **If left empty, auth is disabled** (the `verify_token` dependency returns early, so anyone can hit `/translate`). Always set it in production.
- `LLM_TIMEOUT` — seconds for the LLM HTTP call (default `8.5`). Collabora's libcurl client uses a hardcoded 10s total timeout (`CURLOPT_TIMEOUT=10L`); staying under it lets the bridge respond—or fall back to untranslated—before Collabora drops the connection.
- `DEEPL_API_KEY` — optional. If set, the bridge forwards translation requests to the real DeepL API first and falls back to the LLM only on DeepL failure (429 rate-limit, 456 quota, connection error). Lets you A/B test DeepL vs the LLM on the same Collabora document, and keep translating when DeepL quota runs out. A free key (ending `:fx`) auto-selects `api-free.deepl.com`; a Pro key selects `api.deepl.com`.
- `DEEPL_API_URL` — override the DeepL endpoint (auto-detected from the key suffix otherwise).
- `DEEPL_TIMEOUT` — seconds for the DeepL call (default `9.0`).
- `LOG_LEVEL` — `debug` logs every request/LLM-call/response to stderr (`docker logs`); `info` logs only backend choice + fallbacks; any other value silences.

## Architecture

Two modules:

### `bridge.py` — structure-preserving HTML translation (the core)
This is what real DeepL's `tag_handling=html` does, replicated so the LLM **never touches markup**. The previous design handed raw HTML to the LLM with a "please preserve tags" instruction; the model dropped attributes, duplicated tags, and merged runs, which in Collabora's whole-document loop cascaded into runaway header/footer duplication and lost pptx run formatting.

- `extract_text_nodes(html)` → `list[str]` — parses the fragment with lxml (inside a throwaway `<div>` wrapper, so multi-root fragments are NOT wrapped in an extra `<span>`), walks in document order, and collects translatable text from `.text`/`.tail` slots. Whitespace-only fragments are skipped. `<script>`/`<style>`/`<code>`/`<pre>` content is skipped.
- `refill_text_nodes(html, translations)` → `str` — puts translated strings back into the same slots. Raises `ValueError` on count mismatch (never silently misalign).
- `batch_texts(texts, max_items=500, max_chars=12000)` → `list[list[str]]` — a **safety net only**, not a context-chopper. Real paragraphs are small (translate in 1-2.5s with thinking disabled) and must NEVER be split — cross-sentence context is what makes LLM translation accurate. The high thresholds only catch pathological payloads (e.g. a 150-span table exported as one fragment) that could blow the time budget. A single text larger than max_chars gets its own batch (never split mid-string).
- `translate_html(html, target_lang, translate_fn=None)` — orchestrates extract → batch → translate → refill. On ANY structural problem (count mismatch, parse error, translator exception) it returns the **original HTML untranslated** — a quality regression is strictly better than a corrupted/duplicated document.
- `map_target_lang(code)` — maps DeepL codes (`ZH`, `EN-US`, `ZH-HANS`, …) to human language names for the prompt; unknown codes fall back to the code itself.

### `main.py` — endpoint + LLM call
1. **`verify_token` dependency** — accepts the bridge token from `Authorization: Bearer <t>`, `Authorization: DeepL-Auth-Key <t>` (DeepL's convention), a bare `Authorization` header, or a form field `auth_key` (DeepL allows auth in form data). Reading `auth_key` consumes the form body, so it only happens when the header is absent.
2. **`llm_translate_items(texts, target_lang_name)` → `list[str]`** — sends the text strings to the LLM with `response_format={"type":"json_object"}` and a system prompt that binds the output to `{"items":[...]}`. This makes the 1:1 count contract enforceable; if the model misbehaves, `translate_html` falls back. `temperature: 0`. Sends `generationConfig.thinkingConfig.thinkingBudget=512` — a SMALL but NON-ZERO Gemini thinking budget. With 0 the model drops short/low-context fragments (e.g. a lone `" 页"`) to empty strings, producing empty `<span></span>` that cascades into header/footer duplication in Collabora's whole-doc loop; 512 is the minimum effective value (verified: ≤128 drops fragments, ≥512 does not). Real paragraphs finish in 3-4s — under Collabora's 10s. Note: the system prompt uses `str.format` with a `{target}` placeholder, so literal braces in the JSON template are escaped as `{{`/`}}` — do not un-escape them or you'll get `KeyError`.
3. **`translate_items(texts, target_lang, target_lang_name, tag_handling)` → `list[str]`** — the backend router. If `DEEPL_API_KEY` is set, calls `deepl_translate_items` (forwards to real DeepL, `tag_handling=html` passed through, `split_sentences=nonewlines`); on any DeepL failure (429/456/network) falls back to `llm_translate_items`. With no DeepL key, uses the LLM directly. Logs `[BACKEND] DeepL/LLM served N/M items`. The endpoint builds a `route_translate` closure capturing the request's `tag_handling`+`target_lang` and passes it as `translate_fn` to the bridge, so the bridge's batching calls the router per batch.
4. **`POST /translate`** — accepts JSON or form bodies. `text` may be a string or array (array → one translation per element, same order). Reads `target_lang` (DeepL code, default `ZH`) and `tag_handling` from the **query string** (Collabora passes `?tag_handling=html`). HTML path goes through `bridge.translate_html`; plain text through `bridge.translate_plain`.
5. **Response shaping** — wraps results into DeepL's `{"translations": [{"detected_source_language": "EN", "text": ...}]}`. `detected_source_language` is hardcoded `"EN"` — the bridge does no real language detection.

`GET /health` returns `{"status":"ok"}`.

## CI / release

`.github/workflows/build-deepl-docker.yml` builds and pushes the Docker image to `ghcr.io/<owner>/<repo>:latest` on **push to `main` that touches `Dockerfile`** (other paths are commented out), plus `workflow_dispatch`. It uses GitHub Actions cache (`type=gha`). Changing `main.py` or `requirements.txt` alone will **not** trigger a build — either also touch `Dockerfile` or run the workflow manually.

---
> Source: [xyonium/deepl-to-llm](https://github.com/xyonium/deepl-to-llm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-11 -->
