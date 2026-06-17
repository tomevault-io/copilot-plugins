## medsnap-idea

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- `npm run dev` — Start Vite dev server (required for the Groq proxy to work; see Architecture).
- `npm run build` — Production build via Vite.
- `npm run preview` — Preview the production build locally.
- `npm run lint` — Run ESLint over the project.

There is no test runner configured.

## Architecture

MedSnap is a single-page React 19 + Vite app that walks a user through a 4-step pipeline for checking drug-drug interactions from a photo of pill bottles. State for the whole pipeline lives in `src/App.jsx` (`step`, `image`, `ocrDrugs`, `resolvedDrugs`); each step is a presentational component that receives data + callbacks and hands the next stage's data back up.

The four steps and the external services they call:

1. **`CaptureStep`** — Image input via mobile camera (`capture="environment"`), drag-and-drop, or a manual-entry shortcut that skips OCR. Hands a data URL up to `App`.
2. **`OcrStep`** — Sends each image (data URL) to a **Groq vision model** (`meta-llama/llama-4-scout-17b-16e-instruct`) via the OpenAI-compatible chat completions endpoint with multipart `image_url` content and `response_format: json_object`. The model returns `{"drugs": [...]}` per image; results are deduplicated client-side and shown as an editable textarea. The Groq key is read from `localStorage` — if missing, this step shows an inline key-entry form before processing (the key was previously only collected at step 4). Per-image errors are shown in a collapsible details panel; if all images fail, an error banner appears above the textarea so the user can still type names manually.
3. **`ResolveStep`** — For each entered name, calls the public **RxNorm REST API** (`rxnav.nlm.nih.gov`):
   - `approximateTerm.json` for fuzzy matching → up to 3 candidates per input.
   - `rxcui/{rxcui}/properties.json` to enrich each candidate.
   - `rxcui/{rxcui}/related.json?tty=IN` to walk from the user-selected concept down to its base **ingredient** RxCUI. The downstream FDA lookup is done against the ingredient, not the brand/clinical-drug concept.
   - User must select ≥2 drugs to advance.
4. **`InteractionStep`** — Two-phase analysis:
   - **Phase 1:** For each resolved drug, hits **openFDA** (`api.fda.gov/drug/label.json`) trying three searches in order: `openfda.rxcui`, `generic_name`, `brand_name`. Pulls `drug_interactions`, `warnings`, `contraindications`.
   - **Phase 2:** Sends the truncated label text + drug list to **Groq** (`llama-3.3-70b-versatile`) with a strict-JSON prompt asking for pairwise severities (`HIGH`/`MEDIUM`/`LOW`/`NONE`). Response is stripped of markdown fences and `JSON.parse`d. Renders both a sorted card list and an interaction matrix.

### Groq API key handling

The Groq key is **user-supplied at runtime**, kept in `localStorage` under `medsnap_groq_key`, and sent in the `Authorization` header from the browser. There is no backend.

To avoid CORS issues in dev, requests go to `/api/groq/...` and `vite.config.js` proxies that prefix to `https://api.groq.com` (stripping `/api/groq`). **This proxy only exists during `npm run dev` / `vite preview`** — a static production deploy of `dist/` will need its own equivalent rewrite (or the fetch URL needs to be changed) or the calls will 404.

### Conventions

- JSX-only (no TypeScript). React 19, function components + hooks.
- Styling is plain CSS in `src/App.css` + inline styles; no CSS framework. Theme tokens (`--accent`, `--surface2`, `--text2`, etc.) are defined in `App.css`.
- The disclaimer in `App.jsx` ("For informational purposes only…") is load-bearing — this is a hackathon medical-info tool, not clinical software. Preserve disclaimers when editing.

---
> Source: [riteshbachhar/MedSnap_idea](https://github.com/riteshbachhar/MedSnap_idea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-06-17 -->
