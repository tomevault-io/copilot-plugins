## zotero-skimming

> Zotero 9 add-on that finds salient prose sentences in academic PDFs and saves them as native Zotero highlight annotations. It is a dependency-free, manifest-v2 extension loaded from `bootstrap.js`.

# AGENTS.md

## Project

Zotero 9 add-on that finds salient prose sentences in academic PDFs and saves them as native Zotero highlight annotations. It is a dependency-free, manifest-v2 extension loaded from `bootstrap.js`.

## Layout

- `manifest.json` — add-on identity, version, and Zotero 9 compatibility.
- `bootstrap.js` — lifecycle hooks; loads scripts in dependency order.
- `content/annotator.js` — Zotero UI, persisted settings, all-page positioned PDF extraction, sentence-to-rectangle mapping, and annotation creation.
- `content/nlp.js` — model-free NLP baseline: filtering, TF-IDF features, TextRank, role heuristics, and MMR selection.
- `content/model-manager.js` — optional local Transformers.js runtime, model downloads/cache, embeddings, classification, and reranking.
- `content/model-host.mjs` — standalone remote-runtime host experiment; it is not loaded by `bootstrap.js`.
- `README.md` / `CHANGELOG.md` — user-facing description and release history.

## Runtime constraints

- Target Zotero `9.x`; retain manifest v2 and the declared add-on ID unless an intentional compatibility/migration change is requested.
- Scripts share Zotero's chrome scope. Globals are deliberate: `FastKeySentenceNLP`, `FastKeySentenceModels`, and `FastOfflineKeySentenceAnnotator`.
- Preserve load order in `bootstrap.js`: `nlp.js` → `model-manager.js` → `annotator.js`.
- Use Zotero/Gecko APIs (`Zotero`, `Services`, `IOUtils`, `PathUtils`) rather than Node or browser-only assumptions.
- Keep PDF processing local. Baseline ranking must work without downloading models; optional inference must fail back to it cleanly.
- Do not alter the source PDF. Annotations use `Zotero.Annotations.saveFromJSON()` and must retain valid positioned rectangles and sort indexes.
- `Update models` explicitly downloads runtime/model assets into the Zotero profile cache. Preserve q8 selection, local-cache resolution, single-threaded WASM, disabled ONNX proxy workers, and clear progress/failure reporting.

## Code conventions

- JavaScript only; match the existing two-space indentation, semicolons, `const`/`let`, async/await, and object-module style.
- Keep changes narrowly scoped and dependency-free. Avoid bundlers, transpilers, npm metadata, or Node imports unless the project architecture is intentionally changed.
- Use `Zotero.debug()` through existing logging helpers; surface user-actionable failures with the existing dialog/progress UI.
- Treat PDF worker internals and Zotero APIs defensively: feature-detect APIs, handle missing local files, and preserve error messages/fallback behavior.
- Maintain sentence geometry, reading order, reference/table/front-matter filtering, and duplicate-auto-annotation safeguards when changing extraction or ranking code.

## Validation

No automated test suite or build script is currently tracked. For JavaScript syntax checks, run:

```sh
node --check bootstrap.js
node --check content/annotator.js
node --check content/nlp.js
node --check content/model-manager.js
node --check content/model-host.mjs
```

Then manually test in Zotero 9 with representative PDFs: single- and two-column layouts, multi-page documents, tables, references, and scanned/OCR-needed PDFs. Verify menu visibility, settings persistence, native highlight placement, duplicate prevention, baseline ranking, model download progress, and baseline fallback after model/runtime failure.

## Release hygiene

- Keep `manifest.json` version, `README.md` current-version text, and `CHANGELOG.md` aligned for releases.
- Do not commit generated `.xpi`, `.zip`, build output, model caches, or `node_modules/`.

---
> Source: [00sapo/zotero-skimming](https://github.com/00sapo/zotero-skimming) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-24 -->
