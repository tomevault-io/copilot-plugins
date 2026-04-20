## marker

> This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Project Does

**Marker** is a document conversion pipeline that converts PDFs, images, PPTX, DOCX, XLSX, HTML, and EPUB files into Markdown, JSON, HTML, or chunked output. It uses the [Surya](https://github.com/datalab-to/surya) deep learning library for layout detection, OCR, and table recognition, with optional LLM-based post-processing for higher accuracy.

## Commands

```bash
# Install dependencies
poetry install
poetry install --extras "full"  # adds DOCX, XLSX, PPTX, EPUB support

# Run tests
poetry run pytest
poetry run pytest tests/test_pdf.py  # single file
poetry run pytest -k "test_name"     # single test

# Lint and format
ruff check
ruff format

# CLI entry points
marker_single --fname path/to/file.pdf --output_dir output/
marker --input_folder /path/to/pdfs --output_folder output/
marker_server  # FastAPI server
marker_gui     # Streamlit app
```

## Architecture

The pipeline has five stages that data flows through sequentially:

```
File → Provider → Builders → Processors → Renderer → Output
```

### 1. Providers (`marker/providers/`)
Abstract raw content from input files. The `registry.py` auto-selects the correct provider by file extension. Each provider exposes a unified interface for page images and text.

### 2. Builders (`marker/builders/`)
Construct a `Document` object (defined in `marker/schema/document.py`) from provider output:
- `LayoutBuilder` — runs Surya layout model to detect block regions per page
- `LineBuilder` — extracts text lines, handles OCR error detection
- `OcrBuilder` — runs Surya OCR on pages/regions that need it
- `StructureBuilder` — assembles blocks into a hierarchical document structure

### 3. Processors (`marker/processors/`)
Transform `Block` objects in a defined sequence (~33 processors in `PdfConverter`). Each processor handles one concern: ordering, merging lines, detecting code blocks, formatting tables, extracting equations, etc. LLM-based processors live in `marker/processors/llm/` and are optional (activated with `use_llm=True`).

### 4. Renderers (`marker/renderers/`)
Convert the processed `Document` to an output format: `markdown.py`, `json.py`, `html.py`, `chunk.py`, `ocr_json.py`, `extraction.py`.

### 5. Converters (`marker/converters/`)
End-to-end pipelines that wire together specific builders, processors, and renderers. `PdfConverter` is the main one. `ExtractionConverter` adds schema-based structured extraction.

### Key Files

| File | Role |
|------|------|
| `marker/converters/pdf.py` | `PdfConverter` — main pipeline, defines the processor sequence |
| `marker/schema/__init__.py` | `BlockTypes` enum — all 30+ block types |
| `marker/schema/document.py` | `Document` model and page/block navigation helpers |
| `marker/schema/blocks/` | Individual block classes (Table, Figure, Equation, etc.) |
| `marker/settings.py` | Global settings — device, paths, API keys |
| `marker/models.py` | Load Surya model dict passed into the pipeline |
| `marker/config/parser.py` | CLI config generation; crawls builders/processors for their config |

### LLM Services (`marker/services/`)
Pluggable LLM backends: `gemini.py` (default), `claude.py`, `openai.py`, `azure_openai.py`, `ollama.py`, `vertex.py`. Selected via `GOOGLE_API_KEY` or similar env vars and the `llm_service` config option.

### Configuration System
Builder, processor, and converter classes declare their own config via `__init__` parameters. `marker/config/parser.py` crawls all classes in a pipeline and generates a unified CLI/config dict dynamically.

## Testing Notes

- Tests load PDF fixtures from the `datalab-to/pdfs` Hugging Face dataset (requires network/HF token on first run).
- Fixtures and dataset loading are in `tests/conftest.py`.
- GPU tests run in CI with Python 3.11; CPU fallback works locally.
- `pytest.ini` configures markers and test paths.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saitoi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:copilot_instructions:2026-04-09 -->
