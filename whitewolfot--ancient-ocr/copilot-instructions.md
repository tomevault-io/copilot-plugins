## ancient-ocr

> This file tells Claude Code everything it needs to build and extend this project correctly.

# CLAUDE.md — Ancient-OCR (Arabic OCR + Lexicon-Augmented Intelligence)

This file tells Claude Code everything it needs to build and extend this project correctly.
Read it fully before writing any code. Follow the rules below without exception.

---

## What this system does

Ingests scanned Arabic PDFs/images → runs an OCR ensemble → normalizes noisy output → resolves uncertain words using classical Arabic lexicons + morphology → returns corrected Arabic text with full per-token provenance.

Three output modes: `clean` | `annotated` | `debug`.
Two interfaces: FastAPI API + CLI. No separate frontend required.

---

## Repo structure

```
/
├── CLAUDE.md                  ← you are here
├── main.py                    ← pipeline orchestrator (wire-up only)
├── config.yaml                ← all settings; no magic numbers in code
├── requirements.txt
├── pyproject.toml             ← optional extras: [layout], [trocr], [lm]
├── README.md
├── io/                        ← document_loader.py (PDF+image → pages)
├── preprocessing/             ← image_pipeline, deskew, thresholding, denoise, layout_detection
├── ocr_engine/                ← schema, base, paddle, tesseract, trocr, ensemble
├── alignment/                 ← token_matcher, bbox_alignment, string_similarity
├── normalization/             ← arabic_normalizer, noise_filter
├── morphology/                ← root_extractor, pattern_analyzer, camel_adapter
├── lexicon_ingestion/         ← sources, parser, storage, index_builder
├── lexicon_engine/            ← query_engine, candidate_generator, context_scorer, scorer, ranker
├── confidence_engine/         ← scoring, decision, state
├── output/                    ← formatter, json_export, markdown_export, debug_export
├── api/                       ← server.py, routes.py, schemas.py
├── cli/                       ← main.py, commands.py
├── utils/                     ← config.py, logging.py, cache.py
├── eval/                      ← metrics.py, benchmark.py
├── tests/
├── data/                      ← lexicon raw + built indexes (gitignored)
└── models/                    ← downloaded/finetuned weights (gitignored)
```

---

## Build order — mandatory, do not skip phases

```
1.  Skeleton: all __init__.py, config.yaml, utils/config.py, utils/logging.py
2.  main.py orchestrator (safe imports — fail with clear message if module missing)
3.  Shared contracts: ocr_engine/schema.py + confidence_engine/state.py  ← build first, never change shape
4.  io/document_loader.py
5.  preprocessing/: denoise → thresholding → deskew → layout_detection → image_pipeline
6.  ocr_engine/: base → paddle_backend → tesseract_backend → trocr_backend
7.  alignment/: string_similarity → bbox_alignment → token_matcher
8.  ocr_engine/ensemble.py (uses alignment)
9.  normalization/: noise_filter → arabic_normalizer
10. lexicon_ingestion/: sources → parser → storage → index_builder
11. morphology/: root_extractor → pattern_analyzer → camel_adapter
12. lexicon_engine/: query_engine → candidate_generator → context_scorer → scorer → ranker
13. confidence_engine/: scoring → decision  (state already done in step 3)
14. output/: formatter → json_export → markdown_export → debug_export
15. api/: server → routes → schemas
16. cli/: main → commands
17. tests/ + eval/
18. README.md
```

---

## Shared data contracts — define once, obey everywhere

Use **Pydantic v2**. All models must be JSON-serializable. Build these in step 3 and never change the field names downstream.

```python
class WordToken(BaseModel):
    text: str
    confidence: float           # 0..1
    bbox: tuple[int,int,int,int]  # x, y, w, h
    page_index: int
    source: str                 # "paddle"|"tesseract"|"trocr"|"ensemble"

class OCRResult(BaseModel):
    text: str
    words: list[WordToken]
    confidence: float
    page_index: int
    source: str
    raw: dict = {}              # engine-specific payload for debug

class LexiconEntry(BaseModel):
    lemma: str
    root: str | None
    pattern: str | None
    gloss: str
    source: str
    era: str                    # "classical" | "modern"
    domain: str | None
    examples: list[str] = []
    priority: int

class Candidate(BaseModel):
    text: str
    reason: str                 # "spelling_variant"|"normalization"|"root_alt"|"morph_alt"|"identity"
    lexicon_entries: list[LexiconEntry] = []
    features: dict = {}         # lexicon_score, morph_score, ocr_score, context_score
    score: float | None = None

class TokenState(BaseModel):
    original: str
    normalized: str
    normalization_log: list[dict]        # per-step changes (traceability)
    candidates: list[Candidate]
    selected: str
    confidence: float
    sources: list[str]
    decision: str               # "accept"|"accept_with_note"|"uncertain"|"review_required"
    reason_code: str
```

---

## Non-negotiable design rules

1. **Never overwrite a token without storing the original + alternatives.**
2. **Every correction must carry its source(s) and a reason code.**
3. **Lexicon evidence > morphology > model guessing.** Morphology supports; never overrules.
4. **Raw per-engine OCR stays accessible throughout** (needed for debug mode).
5. **Config drives everything.** No magic numbers in code — read from `config.yaml`.
6. **No side effects at import.** No model downloads, no DB connections at import time.
7. **Graceful degradation.** Missing optional dep/model/lexicon → log it, disable that signal, continue.
8. **Every module independently unit-testable** with no heavy dependencies.
9. Classical and modern lexicon evidence stay tagged and separable.

---

## Pipeline order (canonical — main.py must follow this exactly)

```
input file
 → io/document_loader          (PDF/PNG/JPG/TIFF → page images)
 → preprocessing/image_pipeline
 → ocr_engine/ensemble         (Paddle primary; Tesseract + TrOCR optional)
 → alignment (inside ensemble)
 → normalization/arabic_normalizer + noise_filter
 → morphology/camel_adapter (+ fallbacks)
 → lexicon_engine/query_engine + candidate_generator + context_scorer + scorer + ranker
 → confidence_engine/scoring + decision
 → output/formatter + exporter
```

---

## Confidence model

```python
final = 0.30*ocr_confidence + 0.30*lexicon_score + 0.20*morphology_score + 0.20*context_score
```

Decision thresholds (from `config.decision`):

| Score | Label |
|---|---|
| >= 0.90 | accept |
| 0.70–0.89 | accept_with_note |
| 0.50–0.69 | uncertain |
| < 0.50 | review_required |

---

## Key technical constraints

**OCR engines:**
- PaddleOCR is primary (`lang="ar"`) but quality on classical/Ottoman/Quranic type is limited — treat it as one signal.
- Tesseract requires system packages `tesseract-ocr` + `tesseract-ocr-ara`. Detect absence and fail clearly.
- TrOCR has **no official Arabic model**. Use a community fine-tuned checkpoint configured in `config.yaml`. Its role is **line-level re-recognition** of low-confidence crops, not bbox token extraction. Make it fully optional.

**Normalization:**
- `normalize_text()` returns `(text, change_log)`. The change log per token IS the traceability — there is no lossless global inverse.
- `reverse_normalization_map()` documents the folding rules only.

**Layout detection:**
- Default is pure-OpenCV projection-profile. `layoutparser` (detectron2) is an **optional extra** `[layout]`. Never make it a hard dependency.

**Lexicon sources:**
- OpenITI/Shamela public-domain dumps for Lisān al-ʿArab, Tāj al-ʿArūs, al-Qāmūs, Lane's Lexicon.
- **DO NOT scrape Almaany** (ToS). Keep a disabled stub entry in `sources.py`.
- Ship a tiny synthetic fixture lexicon at `data/lexicons/_fixture/` so the full pipeline runs with zero external data.

**CAMeL Tools:**
- Lazy-load the morphology DB (`camel_data -i morphology-db-msa-r13`).
- If absent → rule-based fallback in `root_extractor.py` + `pattern_analyzer.py`.

**context_scorer.py:**
- This module MUST exist and produce a `float` in [0,1]. Default: deterministic n-gram/co-occurrence.
- Optional masked-LM (AraBERT) behind `[lm]` extra + config flag.
- Without it the confidence formula silently zeroes out a 20% weight — that is not acceptable.

---

## Running the system

```bash
# CLI
python -m cli.main process input.pdf --mode annotated
python -m cli.main process input.png --mode debug
python -m cli.main batch ./folder/ --mode clean

# API
uvicorn api.server:app --reload
# POST /process  (multipart file + mode param)

# Tests
pytest tests/

# Benchmark
python -m eval.benchmark --data ./data/benchmark/
```

---

## Adding a new lexicon source

1. Add an entry to `lexicon_ingestion/sources.py`.
2. Write a parser adapter in `lexicon_ingestion/parser.py`.
3. Run the ingestion script to populate `data/lexicons/<source>/` and rebuild indexes.
4. Enable in `config.yaml` under `lexicon.sources`.

No other files need changing.

---

## Full design reference

See `docs/MASTER_PLAN.md` for the complete annotated architecture, the rationale behind every design decision, the data-acquisition and licensing table, and a full risk register.

---
> Source: [whitewolfOT/Ancient-OCR](https://github.com/whitewolfOT/Ancient-OCR) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:copilot_instructions:2026-07-30 -->
